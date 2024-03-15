<script lang="ts" setup>
import { nextTick, onUpdated, ref, watch } from "vue";

type Props = {
  value?: number;
  isEditing?: boolean;
};

const props = withDefaults(defineProps<Props>(), {
  value: undefined,
  isEditing: false,
});

type Emits = {
  (e: "update:value", payload: number): void;
  (e: "focus"): void;
  (e: "blur"): void;
};

const emit = defineEmits<Emits>();
const $input = ref();
const _value = ref();

watch(
  () => props.value,
  (value) => {
    _value.value = value;
  },
  {
    immediate: true,
  }
);

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

const onBlur = () => {
  let updatedValue = _value.value ?? "";
  updatedValue = updatedValue.replace(/[^0-9.]/g, "");
  emit("update:value", parseFloat(updatedValue));
  emit("blur");
};

onUpdated(() => {
  if (props.value !== _value.value) {
    onBlur();
  }
});
</script>

<template>
  <div v-show="props.isEditing" @click.stop>
    <textarea
      ref="$input"
      :value="props.value"
      @input="(event) => _value = (event?.target as any)?.value"
      @keydown.enter="onBlur()"
      @blur="emit('blur')"
    />
  </div>
  <button v-show="!props.isEditing" @click.stop="emit('focus')">
    {{ props.value }}
  </button>
</template>

<style scoped>
textarea {
  position: absolute;
  box-sizing: border-box;
  width: 240px;
  border: none;
  resize: none;
  padding: 6px 9px;
  height: 34px;
  box-shadow: rgba(15, 15, 15, 0.05) 0px 0px 0px 1px,
    rgba(15, 15, 15, 0.1) 0px 3px 6px, rgba(15, 15, 15, 0.2) 0px 9px 24px;
}
textarea:focus-visible {
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
</style>
