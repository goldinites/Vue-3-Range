<template>
  <div
      class="range"
      :class="{'range--disabled': disabled}"
      ref="rangeRef"
      @click="rangeClickHandler"
  >
    <div class="range__line"/>
    <div class="range__line range__line--value"
         :style="`
              left: ${rangeValueStyles.left}px;
              width: ${rangeValueStyles.width}px;
            `"
    />
    <div
        class="range__thumb range__thumb--left"
        :style="`left: ${leftThumbPosition}px;`"
        :class="{'range__thumb--elevated': minValue >= maxValue && lastThumb === 'left'}"
        @mousedown="leftRangeThumbMouseDownHandler"
        @touchstart="leftRangeThumbMouseDownHandler"
    />
    <div
        v-if="double"
        class="range__thumb range__thumb--right"
        :style="`left: ${rightThumbPosition}px;`"
        :class="{'range__thumb--elevated': maxValue <= minValue && lastThumb === 'right'}"
        @mousedown="rightRangeThumbMouseDownHandler"
        @touchstart="rightRangeThumbMouseDownHandler"
    />
  </div>
</template>

<script setup>
import {computed, onMounted, ref, watch} from "vue";

const props = defineProps({
  min: {
    type: Number,
    default: 0
  },
  max: {
    type: Number,
    default: 1
  },
  minValue: {
    type: Number,
  },
  maxValue: {
    type: Number,
  },
  step: {
    type: Number,
    default: 1,
  },
  double: {
    type: Boolean,
    default: true
  },
  clickable: {
    type: Boolean,
    default: true
  },
  disabled: {
    type: Boolean,
    default: false
  }
});

const emit = defineEmits(['update']);

if (!props.min && props.min !== 0) {
  throw new Error('Минимум не установлен');
}

if (!props.max) {
  throw new Error('Максимум не установлен');
}

if (props.min > props.max) {
  throw new Error('Минимум не должен быть больше максимума');
}

if (props.max < props.min) {
  throw new Error('Максимум не должен быть меньше минимума');
}

if (props.minValue > props.maxValue) {
  throw new Error('Минимальное значение не должно быть больше максимального');
}

if (props.maxValue < props.minValue) {
  throw new Error('Максимальное значение не должно быть меньше минимального');
}

const minValue = ref(!props.minValue || props.minValue < props.min ? props.min : props.minValue);
const maxValue = ref(!props.double || !props.maxValue || props.maxValue > props.max ? props.max : props.maxValue);
const rangeRef = ref(null);
const rangeRefWidth = ref(0);
const rangeRefLeft = ref(0);
const lastThumb = ref('left');

const maxSteps = (props.max - props.min) / props.step;
const decimal = props.step?.toString()?.split('.')?.[1]?.length ?? 0;

const stepWidth = computed(() => {
  if (rangeRefWidth.value && maxSteps) {
    return rangeRefWidth.value / maxSteps;
  }

  return 0;
});

const leftThumbPosition = computed(() => {
  return stepWidth.value * (minValue.value - props.min) / props.step;
});

const rightThumbPosition = computed(() => {
  return stepWidth.value * (maxValue.value - props.min) / props.step;
});

const rangeValueStyles = computed(() => {
  const single = {
    left: 0,
    width: leftThumbPosition.value
  };

  const double = {
    left: leftThumbPosition.value,
    width: rightThumbPosition.value - leftThumbPosition.value
  }

  if (props.double) {
    return double;
  } else {
    return single;
  }
})

const leftRangeThumbMouseDownHandler = () => {
  document.addEventListener('mousemove', leftRangeThumbMouseMoveHandler);
  document.addEventListener('mouseup', leftRangeThumbMouseUpHandler);
  document.addEventListener('touchmove', leftRangeThumbMouseMoveHandler);
  document.addEventListener('touchend', leftRangeThumbMouseUpHandler);
}

const rightRangeThumbMouseDownHandler = () => {
  document.addEventListener('mousemove', rightRangeThumbMouseMoveHandler);
  document.addEventListener('mouseup', rightRangeThumbMouseUpHandler);
  document.addEventListener('touchmove', rightRangeThumbMouseMoveHandler);
  document.addEventListener('touchend', rightRangeThumbMouseUpHandler);
}

const leftRangeThumbMouseMoveHandler = (e) => {
  const value = calcRangeValue(e);

  setMinValue(value);

  emitValues();
}

const rightRangeThumbMouseMoveHandler = (e) => {
  const value = calcRangeValue(e);

  setMaxValue(value);

  emitValues()
}

const leftRangeThumbMouseUpHandler = () => {
  lastThumb.value = 'left';
  document.removeEventListener('mousemove', leftRangeThumbMouseMoveHandler);
  document.removeEventListener('mouseup', leftRangeThumbMouseUpHandler);
  document.removeEventListener('touchmove', leftRangeThumbMouseMoveHandler);
  document.removeEventListener('touchend', leftRangeThumbMouseUpHandler);
}

const rightRangeThumbMouseUpHandler = () => {
  lastThumb.value = 'right';
  document.removeEventListener('mousemove', rightRangeThumbMouseMoveHandler);
  document.removeEventListener('mouseup', rightRangeThumbMouseUpHandler);
  document.removeEventListener('touchmove', rightRangeThumbMouseMoveHandler);
  document.removeEventListener('touchend', rightRangeThumbMouseUpHandler);
}

const rangeClickHandler = (e) => {
  if (props.clickable) {
    let nearestThumb;
    const value = calcRangeValue(e);

    if (value < minValue.value) {
      nearestThumb = 'left';
    } else if (value > maxValue.value) {
      nearestThumb = 'right';
    } else {
      const minDiff = value - minValue.value;
      const maxDiff = maxValue.value - value;

      if (minDiff < maxDiff) {
        nearestThumb = 'left';
      } else if (maxDiff < minDiff) {
        nearestThumb = 'right';
      } else if (maxDiff === minDiff) {
        nearestThumb = Math.random() > 0.5 ? 'left' : 'right';
      }
    }

    if (nearestThumb === 'left' || !props.double) {
      setMinValue(value);
    } else {
      setMaxValue(value);
    }

    emitValues();
  }
}

const setMinValue = (value) => {
  if (value !== minValue.value && typeof value === "number") {
    if (value > maxValue.value) {
      minValue.value = maxValue.value;
    } else if (value <= props.min) {
      minValue.value = props.min;
    } else {
      minValue.value = value;
    }
  }
}

const setMaxValue = (value) => {
  if (value !== maxValue.value && typeof value === "number") {
    if (value < minValue.value) {
      maxValue.value = minValue.value;
    } else if (value >= props.max) {
      maxValue.value = props.max;
    } else {
      maxValue.value = value;
    }
  }
}

const calcRangeValue = (e) => {
  let clientX = e.clientX;

  if (e.type === 'touchmove') {
    clientX = e.touches[0].clientX;
  }

  const dx = clientX - rangeRefLeft.value;
  let val = (dx / stepWidth.value) * props.step;
  const mod = val % props.step;

  val -= mod;

  return +((val + props.min).toFixed(decimal)) ?? 0;
}

const emitValues = () => {
  const single = {
    value: minValue.value
  }

  const double = {
    minValue: minValue.value,
    maxValue: maxValue.value
  }

  if (props.double) {
    emit('update', double)
  } else {
    emit('update', single)
  }
}

watch(() => props.minValue, (value) => {
  setMinValue(value)
});

watch(() => props.maxValue, (value) => {
  setMaxValue(value)
});

watch(() => props.disabled, (value) => {
  if (value) {
    document.removeEventListener('mousemove', leftRangeThumbMouseMoveHandler);
    document.removeEventListener('mouseup', leftRangeThumbMouseUpHandler);
    document.removeEventListener('touchmove', leftRangeThumbMouseMoveHandler);
    document.removeEventListener('touchend', leftRangeThumbMouseUpHandler);
    document.removeEventListener('mousemove', rightRangeThumbMouseMoveHandler);
    document.removeEventListener('mouseup', rightRangeThumbMouseUpHandler);
    document.removeEventListener('touchmove', rightRangeThumbMouseMoveHandler);
    document.removeEventListener('touchend', rightRangeThumbMouseUpHandler);
  }
});

onMounted(() => {
  if (rangeRef.value) {
    const rangeRefClientRect = rangeRef.value?.getBoundingClientRect();

    if (rangeRefClientRect) {
      rangeRefWidth.value = rangeRefClientRect.width;
      rangeRefLeft.value = rangeRefClientRect.left;
    }
  }
});
</script>

<style lang="scss" scoped>
.range {
  position: relative;
  width: 100%;
  padding: 20px 0;

  &--disabled {
    pointer-events: none;
    opacity: 0.75;
  }

  &__line {
    background-color: #f2f2f2;
    width: 100%;
    height: 2px;
    border-radius: 2px;

    &--value {
      position: absolute;
      left: 0;
      top: 50%;
      transform: translateY(-50%);
      background-color: #06c;
    }
  }

  //&__value {
  //  position: absolute;
  //  left: 0;
  //  right: 0;
  //  top: 50%;
  //  transform: translateY(-50%);
  //  height: 2px;
  //  background-color: #06c;
  //  border-radius: 2px;
  //}

  &__thumb {
    position: absolute;
    z-index: 2;
    top: 50%;
    background-color: #06c;
    width: 4px;
    height: 16px;
    transform: translate(-50%, -50%);
    border-radius: 2px;
    cursor: pointer;
    user-select: none;

    &--left {
      left: 0;
    }

    &--right {
      right: 0;
    }

    &--elevated {
      z-index: 5;
    }
  }
}
</style>
