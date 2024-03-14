<script setup lang="ts">
import { computed, ref } from 'vue'
import { v4 as uuid } from 'uuid'

type Props = {

}

const data = [['name', ['A', 'B', 'C']]]

type Col = {
  key: string;
  text: string;
  type: 'text' | 'number' | 'select' | 'multi-select'
  options?: string[]
}

type Row = {
  key: string;
  cols: { key: string; value: any }[];
}

// const props = withDefaults(defineProps<Props>(), {})

const cols = ref<Col[]>([{
  key: uuid(),
  text: 'Name',
  type: 'text',
}, {
  key: uuid(),
  text: 'Tags',
  type: 'multi-select',
  options: []
}])

const rows = ref<Row[]>([])

const addRow = () => {
  rows.value.push({
    key: uuid(),
    cols: cols.value.map((x) => ({ key: x.key, value: undefined }))
  })
}

const getColType = (key: string) => cols.value.find(x => x.key === key)?.type

const allColLength = computed(() => cols.value.length + 2)

addRow()
</script>

<template>
  <div>
    {{ cols }}
  </div>
  <div>
    {{ rows }}
  </div>
  <table>
    <thead>
      <tr>
        <th>
          <input type="checkbox" />
        </th>
        <th v-for="(col) in cols" :key="col.key" :title="col.key">
          <button>{{ col.text }}</button>
        </th>
        <th>
          <div>
            <button>
              +
            </button>
            <button>
              ...
            </button>
          </div>
        </th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="(row, rowIndex) in rows" :key="row.key">
        <td>
          <input type="checkbox" />
        </td>
        <td v-for="(col, colIndex) in row.cols" :key="col.key" :title="col.key">
          <template v-if="getColType(col.key) === 'text'">
            {{ data[rowIndex][colIndex] }}
          </template>
          <template v-else-if="getColType(col.key) === 'multi-select'">
            {{ data[rowIndex][colIndex] }}
          </template>
        </td>
        <td></td>
      </tr>
      <tr>
        <td :colspan="allColLength">
          <button @click="addRow">+New</button>
        </td>
      </tr>
    </tbody>
    <tfoot>
      <tr>
        <td v-for="(col, index) in cols" :key="col.key" :colspan="index ? 1 : 2">
          <button>Calculate v</button>
        </td>
      </tr>
    </tfoot>
  </table>
</template>

<style scoped>
th,
td {
  border: 1px solid black;
}
</style>
