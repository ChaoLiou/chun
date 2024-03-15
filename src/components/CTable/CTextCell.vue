<script lang="ts" setup>
import { nextTick, onUpdated, ref, watch } from "vue";

type Props = {
  value: string;
  isEditing?: boolean;
};

const props = withDefaults(defineProps<Props>(), {
  value: "",
  isEditing: false,
});

type Emits = {
  (e: "update:value", payload: string): void;
  (e: "focus"): void;
  (e: "blur"): void;
};

const emit = defineEmits<Emits>();
const $input = ref<HTMLElement>();
const _value = ref<string>();

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
        if ($input.value) {
          $input.value.focus();
        }
      });
    }
  }
);

const onBlur = () => {
  emit("update:value", _value.value ?? "");
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
      @blur="onBlur()"
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
