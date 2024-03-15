<script lang="ts" setup>
import tinycolor from "tinycolor2";
import { computed } from "vue";

type Props = {
  color?: string;
  closable?: boolean;
};

const props = withDefaults(defineProps<Props>(), {
  color: undefined,
  closable: false,
});

type Emits = {
  (e: "close"): void;
};

const emit = defineEmits<Emits>();

const textColor = computed(() =>
  tinycolor(props.color).isLight() ? "#000000" : "#ffffff"
);

const padding = computed(() =>
  props.closable ? "0px 0px 0px 6px" : "0px 6px"
);
</script>

<template>
  <span class="ctag">
    <span>
      <slot></slot>
    </span>
    <button v-if="props.closable" class="ctag__close" @click="emit('close')">
      <span class="material-symbols-outlined"> close </span>
    </button>
  </span>
</template>

<style lang="scss" scoped>
.ctag {
  border-radius: 3px;
  height: 20px;
  line-height: 20px;
  color: #402c1b;
  display: flex;
  width: min-content;
  vertical-align: middle;
  font-size: 14px;
  color: v-bind(textColor);
  background-color: v-bind(color);
  padding: v-bind(padding);
}
.ctag__close {
  border: none;
  background-color: inherit;
  width: 20px;
  cursor: pointer;
  color: v-bind(textColor);
  display: grid;
  justify-content: center;
  align-content: center;
  span {
    font-size: 1em;
  }
}
</style>
