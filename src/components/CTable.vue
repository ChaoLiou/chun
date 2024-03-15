<script setup lang="ts">
import { computed, ref } from "vue";
import { v4 as uuid } from "uuid";
import tinycolor from "tinycolor2";
import CTextCell from "./CTable/CTextCell.vue";
import CNumberCell from "./CTable/CNumberCell.vue";
import CSelectCell from "./CTable/CSelectCell.vue";
import CMultiSelectCell from "./CTable/CMultiSelectCell.vue";

type Option = {
  id: string;
  color: string;
  value: string;
};
type Col = {
  id: string;
  text: string;
  type: "text" | "number" | "select" | "multi-select";
  options?: Option[];
};

type Row = {
  id: string;
  cells: { id: string; colId: string; value: any }[];
};

const cols = ref<Col[]>([
  {
    id: uuid(),
    text: "text",
    type: "text",
  },
  {
    id: uuid(),
    text: "number",
    type: "number",
  },
  {
    id: uuid(),
    text: "select",
    type: "select",
    options: [
      {
        id: uuid(),
        color: tinycolor.random().saturate().toHexString(),
        value: "A",
      },
    ],
  },
  {
    id: uuid(),
    text: "multi-select",
    type: "multi-select",
    options: [
      {
        id: uuid(),
        color: tinycolor.random().saturate().toHexString(),
        value: "A",
      },
      {
        id: uuid(),
        color: tinycolor.random().saturate().toHexString(),
        value: "B",
      },
    ],
  },
]);

const rows = ref<Row[]>([]);
const selectedCellId = ref<string | null>(null);

const addRow = (defaultValues?: any[]) => {
  rows.value.push({
    id: uuid(),
    cells: cols.value.map((x, i) => ({
      id: uuid(),
      colId: x.id,
      value: defaultValues?.[i],
    })),
  });
};

const getColType = (key: string) => cols.value.find((x) => x.id === key)?.type;
const getColOptions = (key: string) =>
  cols.value.find((x) => x.id === key)?.options;

const setColOptions = (newOptions: Option[], colId: string) => {
  cols.value = cols.value.map((col) => {
    if (col.id === colId) {
      col.options = newOptions;
    }
    return col;
  });
};

const allColLength = computed(() => cols.value.length + 2);

addRow(["text", 123, "A", ["A", "B"]]);
addRow();
addRow();

document.body.addEventListener("click", () => {
  selectedCellId.value = null;
});

document.body.addEventListener("keyup", (e) => {
  if (e.key === "Escape") {
    selectedCellId.value = null;
  }
});
</script>

<template>
  <table>
    <thead>
      <tr>
        <th style="justify-self: center; align-self: center">
          <input type="checkbox" />
        </th>
        <th v-for="col in cols" :key="col.id" :title="col.id">
          <button>{{ col.text }}</button>
        </th>
        <th>
          <div style="display: grid; grid-template-columns: 32px auto">
            <button style="text-align: center">+</button>
            <button>...</button>
          </div>
        </th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="row in rows" :key="row.id">
        <td style="justify-self: center; align-self: center">
          <input type="checkbox" />
        </td>
        <td v-for="cell in row.cells" :key="cell.id" :title="cell.id">
          <template v-if="getColType(cell.colId) === 'text'">
            <CTextCell
              v-model:value="cell.value"
              :is-editing="cell.id === selectedCellId"
              @focus="selectedCellId = cell.id"
              @blur="selectedCellId = null"
            ></CTextCell>
          </template>
          <template v-else-if="getColType(cell.colId) === 'number'">
            <CNumberCell
              v-model:value="cell.value"
              :is-editing="cell.id === selectedCellId"
              @focus="selectedCellId = cell.id"
              @blur="selectedCellId = null"
            ></CNumberCell>
          </template>
          <template v-else-if="getColType(cell.colId) === 'select'">
            <CSelectCell
              v-model:value="cell.value"
              :options="getColOptions(cell.colId)"
              :is-editing="cell.id === selectedCellId"
              @focus="selectedCellId = cell.id"
              @update:options="setColOptions($event, cell.colId)"
            ></CSelectCell>
          </template>
          <template v-else-if="getColType(cell.colId) === 'multi-select'">
            <CMultiSelectCell
              v-model:value="cell.value"
              :options="getColOptions(cell.colId)"
              :is-editing="cell.id === selectedCellId"
              @focus="selectedCellId = cell.id"
              @update:options="setColOptions($event, cell.colId)"
            ></CMultiSelectCell>
          </template>
        </td>
        <td></td>
      </tr>
      <tr>
        <td :colspan="allColLength" style="grid-column: 1/-1">
          <button style="display: flex; align-items: center" @click="addRow()">
            <span class="material-symbols-outlined"> add </span>
            <span>New</span>
          </button>
        </td>
      </tr>
    </tbody>
    <tfoot>
      <tr>
        <td
          v-for="(col, index) in cols"
          :key="col.id"
          :style="{ 'grid-column': index ? undefined : '1 / 3' }"
        >
          <button
            style="
              display: flex;
              align-items: center;
              justify-content: flex-end;
            "
          >
            <span>Calculate</span>
            <span class="material-symbols-outlined"> expand_more </span>
          </button>
        </td>
        <td></td>
      </tr>
    </tfoot>
  </table>
  {{ selectedCellId }}
  <div>
    {{ cols }}
  </div>
  <div>
    {{ rows }}
  </div>
</template>

<style scoped>
table {
  width: 100%;
}

tr {
  display: grid;
  grid-template-columns: 20px repeat(v-bind(allColLength - 1), 1fr);
  color: rgba(55, 53, 47, 0.65);
  border-top: 1px solid rgb(233, 233, 231);
}

tr button {
  height: 32px;
}

th,
td {
  padding: 0;
  border-right: 1px solid rgb(233, 233, 231);
}

button {
  width: 100%;
  border: none;
  cursor: pointer;
  background-color: #ffffff;
  text-align: start;
  padding: 0px 8px;
}

tfoot button {
  text-align: end;
}
</style>
