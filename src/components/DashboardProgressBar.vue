<template>
  <div class="progress">
    <div class="progress__unit">{{ keyResult.unit }}</div>
    <div class="progress__bar-container">
      <div ref="bar" class="progress__bar" :class="{ completed }" style="width: 0%">
        <div class="progress__current-value" :data-progress="keyResult.currentValue || keyResult.startValue">
          {{ keyResult.currentValue || keyResult.startValue }}
        </div>
      </div>
      <div class="progress__change-container">
        <div ref="change" class="progress__change"></div>
      </div>
    </div>
    <div class="progress__startValue">{{ keyResult.startValue }}</div>
    <div class="progress__targetValue">{{ keyResult.targetValue }}</div>
  </div>
</template>

<script>
import { scaleLinear } from 'd3-scale';
import { format } from 'd3-format';
import { select } from 'd3-selection';
import 'd3-transition';

export default {
  name: 'DashboardProgressBar',

  props: {
    keyResult: {
      type: Object,
      required: true,
    },
  },

  data: () => ({
    completed: false,
  }),

  computed: {
    bar() {
      return select(this.$refs.bar);
    },
    change() {
      return select(this.$refs.change);
    },
  },

  watch: {
    keyResult() {
      this.bar.transition().duration(1000).style('width', this.getWidth());
    },
  },

  mounted() {
    this.bar.style('width', this.getWidth());
    this.getChangeStyle(this.change);
  },

  methods: {
    format(val) {
      return format('%')(val);
    },

    getWidth() {
      const { progression } = this.keyResult;

      this.completed = progression >= 1;

      return format('.0%')(progression);
    },

    getChangeStyle(el) {
      const { fromValue, startValue, targetValue, toValue } = this.keyResult;

      let fromPos = 0;
      let width = 0;
      let direction;

      const scale = scaleLinear().domain([startValue, targetValue]).clamp(true);

      if (toValue || fromValue <= 0) {
        fromPos = scale(fromValue);
        width = scale(toValue || 0) - fromPos;
        direction = 'positive';
      } else {
        fromPos = scale(toValue || 0);
        width = scale(fromValue) - fromPos;
        direction = 'negative';
      }

      el.style('width', this.format(width));
      el.style('left', this.format(fromPos));
      el.classed(direction, true);
    },
  },
};
</script>

<style lang="scss" scoped>
.progress {
  display: grid;
  grid-gap: 0.15em;
  grid-template-areas:
    'unit unit unit'
    'bar bar bar'
    'startValue . targetValue';

  grid-template-rows: auto auto auto;
  grid-template-columns: auto 1.5em auto;
  font-size: 0.9em;
}

.progress__bar-container {
  position: relative;
  grid-area: bar;
  height: 1.7em;
  padding: 2px;
  background: rgba(white, 0.1);
  border: 1px solid var(--color-bg);
  border-radius: 4px;
}

.progress__bar {
  display: flex;
  align-items: center;
  justify-content: flex-end;
  width: 50%;
  height: 100%;
  padding-right: 2px;
  overflow: hidden;
  background: rgba(white, 0.6);
  border-radius: 3px;

  &.completed {
    background: var(--color-primary);

    &::before {
      display: inline-block;
      width: 100%;
      color: var(--color-black);
      font-weight: 900;
      font-family: 'Font Awesome 5 Free';
      text-align: center;
      content: '\f00c';
    }

    > * {
      display: none;
    }
  }
}

.progress__unit {
  grid-area: unit;
}

.progress__current-value {
  display: flex;
  color: var(--color-black);
  font-weight: 600;
  font-size: 0.9em;
}

.progress__change-container {
  position: absolute;
  top: 2px;
  left: 2px;
  width: calc(100% - 4px);
  height: calc(100% - 4px);
}

.progress__change {
  position: absolute;
  top: 0;
  left: 0;
  height: 100%;
  border-radius: 3px;

  $positive: rgba(#dddddd, 0.55);
  $negative: rgba(red, 0.08);

  &.positive {
    background: repeating-linear-gradient(-45deg, transparent, transparent 2.5px, $positive 2.5px, $positive 5px);
    border-left: 1px solid #e0e0e0;
    mix-blend-mode: multiply;
  }

  &.negative {
    background: $negative;
    mix-blend-mode: multiply;
  }
}

.progress__startValue {
  grid-area: startValue;
}

.progress__targetValue {
  grid-area: targetValue;
  text-align: right;
}
</style>
