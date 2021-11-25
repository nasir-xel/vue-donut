<template>
  <!-- https://heyoka.medium.com/scratch-made-svg-donut-pie-charts-in-html5-2c587e935d72 -->
  <div class="donut">
    <svg xmlns="http://www.w3.org/2000/svg" :width="svgBox" :height="svgBox">
      <title>Layer 1</title>
      <circle
        id="svg_1"
        fill="#fff"
        :r="radius"
        cy="50%"
        cx="50%"
        class="donut-hole"
      />
      <circle
        id="svg_2"
        :stroke-width="strokeWidth"
        stroke="#d2d3d4"
        fill="transparent"
        :r="radius"
        cy="50%"
        cx="50%"
        :class="{ 'donut-ring': true, 'is-half': isHalf }"
      />
      <circle
        id="svg_3"
        :stroke-dashoffset="svgBox"
        :stroke-dasharray="[progress.line, progress.gap]"
        :stroke-width="strokeWidth"
        :stroke="color"
        fill="transparent"
        :r="radius"
        cy="50%"
        cx="50%"
        class="donut-segment"
      />
      <circle
        id="svg_4"
        :stroke-dashoffset="svgBox"
        :stroke-dasharray="[progress.line, progress.gap]"
        :stroke-width="strokeWidth"
        :stroke="color"
        fill="transparent"
        :r="radius"
        cy="50%"
        cx="50%"
        class="donut-segment"
      />
      <rect
        width="100%"
        :height="svgBox / 2"
        y="50%"
        fill="#fff"
        v-if="isHalf"
      />
      <!-- Text -->
      <g class="chart-text">
        <text
          x="50%"
          y="19%"
          class="chart-percentage"
          :fill="parseInt(progress.percent) >= 26 ? '#fff' : '#222'"
        >
          {{ progress.percent }}
        </text>
        <text x="50%" y="50%" class="chart-number">
          <slot />
        </text>
        <template v-if="isHalf && showValues">
          <text :x="getLeftHandPosition" y="50%" class="chart-label">
            {{ getLeftHandPosition }}
          </text>
          <text :x="getRightHandPosition" y="50%" class="chart-label">
            {{ getRightHandPosition }}
          </text>
        </template>
      </g>
    </svg>
  </div>
</template>

<script>
export default {
  props: {
    showValues: {
      type: Boolean,
      default: false,
    },
    isHalf: {
      type: Boolean,
      default: false,
    },
    radius: {
      type: Number,
      default: 50,
    },
    strokeWidth: {
      type: Number,
      default: 30,
    },
    minValue: {
      type: Number,
      default: 0.0,
    },
    value: {
      type: Number,
      default: 30,
    },
    maxValue: {
      type: Number,
      default: 300,
    },
    color: {
      type: String,
      default: 'darkblue',
    },
  },
  created() {
    if (this.value > this.maxValue) this.line = this.maxValue;
  },
  data() {
    return {
      line: this.value,
      svgBox: Math.PI * this.radius, // pi * r  (assumed)
      circumference: 2 * Math.PI * this.radius, // 2 * pi * r (formula)
    };
  },
  computed: {
    radiusBoxWidth() {
      // left-side width + diameter + right-side width
      return this.strokeWidth + this.radius * 2 + this.strokeWidth;
    },
    getLeftHandPosition() {
      // svgBox width - radiusBox width divide by 2 to get left only position
      // add strokeWidth to stay inside
      return Number(
        ((this.svgBox - this.radiusBoxWidth) / 2 + this.strokeWidth).toFixed(2)
      );
    },
    getRightHandPosition() {
      // subtract width till left hand from svgBox to get last hand position
      return Number((this.svgBox - this.getLeftHandPosition).toFixed(2));
    },
    // Redundant
    strokDashArray() {
      const line = this.line;
      const gap = this.circumference - this.line;

      return [line, gap];
    },
    progress() {
      const percentOfMaxValue = (this.line / this.maxValue) * 100;
      const percentOfDonut = (percentOfMaxValue * this.circumference) / 100;
      const gap = this.circumference - percentOfDonut;

      return {
        line: this.isHalf ? percentOfDonut / 2 : percentOfDonut,
        gap: this.isHalf ? gap / 2 : gap,
        percent: percentOfMaxValue.toFixed(0) + '%',
      };
    },
  },
};
</script>

<style scoped>
@import url(https://fonts.googleapis.com/css?family=Montserrat:400);

.donut {
  height: 100%;
}

.chart-text {
  font: 16px/1.4em 'Montserrat', Arial, sans-serif;
  fill: #000;
  -moz-transform: translateY(0.25em);
  -ms-transform: translateY(0.25em);
  -webkit-transform: translateY(0.25em);
  transform: translateY(0.25em);
}

.chart-number {
  font-size: 1.25em;
  line-height: 1;
  text-anchor: middle;
  -moz-transform: translateY(-0.25em);
  -ms-transform: translateY(-0.25em);
  -webkit-transform: translateY(-0.25em);
  transform: translateY(-0.25em);
}

.chart-label {
  font-size: 1em;
  text-transform: uppercase;
  text-anchor: middle;
  /* font-weight: bold; */
  -moz-transform: translateY(0.7em);
  -ms-transform: translateY(0.7em);
  -webkit-transform: translateY(0.7em);
  transform: translateY(0.7em);
}

/* .is-half {
  stroke: #fff;
} */
</style>
