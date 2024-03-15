<script lang="ts" setup>
import { computed, nextTick, ref, watch } from "vue";
import CTag from "./CSelectCell/CTag.vue";
import { v4 as uuid } from "uuid";
import tinycolor from "tinycolor2";

type Props = {
  value: string[];
  options?: { id: string; color: string; value: string }[];
  isEditing?: boolean;
};

const props = withDefaults(defineProps<Props>(), {
  value: () => [],
  options: () => [],
  isEditing: false,
});

type Emits = {
  (e: "update:value", payload: string[]): void;
  (
    e: "update:options",
    payload: { id: string; color: string; value: string }[]
  ): void;
  (e: "focus"): void;
  (e: "blur"): void;
};

const emit = defineEmits<Emits>();

const inputValue = ref<string>();
const inputValueToLowerCase = computed(() => inputValue.value?.toLowerCase());
const $input = ref();

const dropdownItems = computed(() =>
  props.options.filter(
    (x) =>
      !inputValueToLowerCase.value ||
      x.value.toLowerCase().includes(inputValueToLowerCase.value)
  )
);

const isCreatable = computed(() => {
  if (inputValue.value) {
    if (dropdownItems.value.length === 0) {
      return true;
    } else if (dropdownItems.value.length === 1) {
      if (dropdownItems.value[0].value !== inputValue.value) {
        return true;
      }
    }
  }
  return false;
});

watch(
  () => props.isEditing,
  (value) => {
    if (value) {
      nextTick(() => {
        $input.value.focus();
      });
    }
  }
);

const getOptionColor = (value: string) => {
  const target = props.options.find((x) => x.value === value);
  return target?.color;
};

const newColor = ref<string>();

watch(inputValue, (value, oldValue) => {
  if (value && !oldValue) {
    newColor.value = tinycolor.random().saturate().toHexString();
  }
});

const updateValue = (value: string) => {
  emit("update:value", Array.from(new Set([...props.value, value])));
};

const createAndSelect = () => {
  if (inputValue.value && newColor.value) {
    updateValue(inputValue.value);
    emit("update:options", [
      ...props.options,
      { id: uuid(), color: newColor.value, value: inputValue.value },
    ]);
    inputValue.value = "";
  }
};

const clickEnterOnInput = () => {
  if (dropdownItems.value.length > 0) {
    updateValue(dropdownItems.value[0].value);
  } else {
    createAndSelect();
  }
};

const closeTag = (value: string) => {
  const updated = props.value.filter((x) => x !== value);
  emit("update:value", updated);
};
</script>

<template>
  <div v-show="props.isEditing" class="dialog" @click.stop>
    <div
      style="
        padding: 6px 8px;
        background-color: rgba(242, 241, 238, 0.6);
        display: flex;
      "
    >
      <CTag
        v-for="item in props.value"
        :key="item"
        :color="getOptionColor(item)"
        :closable="true"
        style="margin-right: 6px"
        @close="closeTag(item)"
      >
        {{ item }}
      </CTag>
      <input
        ref="$input"
        v-model="inputValue"
        placeholder="Search for an option..."
        @keydown.enter="clickEnterOnInput"
      />
    </div>
    <div>
      <span
        style="
          display: flex;
          padding-left: 14px;
          padding-right: 14px;
          margin-top: 6px;
          margin-bottom: 8px;
          color: rgba(55, 53, 47, 0.65);
          fill: rgba(55, 53, 47, 0.45);
          font-size: 12px;
          font-weight: 500;
          line-height: 120%;
        "
      >
        Select an option or create one
      </span>
      <div v-for="item in dropdownItems" :key="item.value" class="options">
        <button @click="updateValue(item.value)">
          <span style="padding: 0px 6px">::</span>
          <span>
            <CTag :color="getOptionColor(item.value)">
              {{ item.value }}
            </CTag>
          </span>
          <span>
            <button
              style="
                height: 20px;
                min-height: 20px;
                padding: 0px;
                text-align: center;
                background-color: inherit;
              "
            >
              ...
            </button>
          </span>
        </button>
      </div>
      <div style="footer">
        <button v-if="isCreatable" @click="createAndSelect">
          Create
          <CTag :color="newColor">
            {{ inputValue }}
          </CTag>
        </button>
      </div>
    </div>
  </div>
  <button
    v-show="!props.isEditing"
    style="display: flex"
    @click.stop="emit('focus')"
  >
    <CTag
      v-for="item in props.value"
      :key="item"
      :color="getOptionColor(item)"
      :style="{ marginRight: '6px' }"
    >
      {{ item }}
    </CTag>
  </button>
</template>

<style scoped>
.dialog {
  position: absolute;
  background-color: #ffffff;
  min-width: 300px;
  box-shadow: rgba(15, 15, 15, 0.05) 0px 0px 0px 1px,
    rgba(15, 15, 15, 0.1) 0px 3px 6px, rgba(15, 15, 15, 0.2) 0px 9px 24px;
}
input {
  border: none;
  background-color: inherit;
}
input:focus-visible {
  outline: none;
}
button {
  width: 100%;
  min-height: 32px;
  border: none;
  cursor: pointer;
  background-color: #ffffff;
  text-align: start;
  padding: 6px 8px;
}
.options > button {
  display: grid;
  grid-template-columns: 20px auto 20px;
}
.footer > button:hover,
.options > button:hover {
  background-color: rgba(55, 53, 47, 0.08);
}
</style>
