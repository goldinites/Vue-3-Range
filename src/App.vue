<template>
  <div class="app">
    <Range
      :min="+min"
      :max="+max"
      :min-value="+minValue"
      :max-value="+maxValue"
      :step="step"
      @update="updateValues"
    />
    <div class="values">
      <div>{{ minValue }}</div>
      <div>{{ maxValue }}</div>
    </div>
  </div>
</template>

<script setup>
import Range from "@/components/Range.vue";
import {ref} from "vue";

const min = 1;
const max = 100;
const minValue = ref(min);
const maxValue = ref(max);
const step = 1;

const updateValues = (values) => {
  // minValue.value = values.value;
  minValue.value = values.minValue;
  maxValue.value = values.maxValue;
}
</script>

<style lang="scss" scoped>
.app {
  width: 340px;
  margin: 0 auto;
  height: 100vh;
  display: flex;
  justify-content: center;
  flex-direction: column;
  align-items: center;
}

.values {
  display: flex;
  justify-content: space-between;
  margin-top: 25px;
  width: 100%;
}
</style>

