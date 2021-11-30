<template>
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
      <template v-for="(donut, i) in progressValues" :key="i">
        <circle
          :id="`circle_${i}`"
          :stroke-dashoffset="donut.offset"
          :stroke-dasharray="[donut.line, donut.gap]"
          :stroke-width="strokeWidth"
          :stroke="donut.color"
          fill="transparent"
          :r="radius"
          cy="50%"
          cx="50%"
          class="donut-segment"
        />
      </template>
      <rect
        width="100%"
        :height="svgBox / 2"
        y="50%"
        fill="#fff"
        v-show="isHalf"
      />
      <!-- Text -->
      <g class="chart-text">
        <text
          v-for="(donut, i) in progressValues"
          :key="i"
          :id="`text_${i}`"
          :x="donut.text.x"
          :y="donut.text.y"
          class="chart-percentage"
          :fill="parseInt(donut.percent) >= 26 ? '#fff' : '#222'"
          :dx="strokeWidth * -0.42"
          :dy="strokeWidth * -0.01"
        >
          {{ donut.percent }}
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
    maxValue: {
      type: Number,
      default: 300,
    },
    values: {
      type: Array,
      default: [30],
    },
    colors: {
      type: Array,
      default: ['red', 'skyblue', 'orange', 'cyan', 'green'],
    },
  },
  created() {
    // if (this.value > this.maxValue) this.line = this.maxValue;
  },
  data() {
    return {
      lines: this.values,
      svgBox: Math.PI * this.radius, // pi * r  (assumed)
      circumference: 2 * Math.PI * this.radius, // 2 * pi * r (formula)
    };
  },
  computed: {
    radiusBoxWidth() {
      // Need this box to find big box's empty space width of any side
      // left-side width + diameter + right-side width
      return this.strokeWidth + this.radius * 2 + this.strokeWidth;
    },
    getLeftHandPosition() {
      // Subtract big box from small box to get big box' side's width
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
    progressValues() {
      const { isHalf, circumference, svgBox, lines, maxValue, colors, radius } =
        this;

      const FIRST_SEGMENT_OFFSET = svgBox;
      const CIRCLE_CENTER = svgBox / 2;
      const TRANSLATE_ANGLE_IN_RADIANS = 3.14;
      let PRECEDING_SEGMENTS_LENGTH = 0;
      let PRECEDING_SEGMENTS_ANGLE = 0;

      return lines.map((line, index) => {
        // find percent of this line in overall donut's length
        const percentOfMaxValue = (line / maxValue) * 100;
        // scaled-down value to donut length i.e. percentOfDonutLength
        const percentOfDonutLength = (percentOfMaxValue * circumference) / 100;
        const gap = circumference - percentOfDonutLength;
        const currentSegmentOffset =
          circumference - (PRECEDING_SEGMENTS_LENGTH + FIRST_SEGMENT_OFFSET);

        const angle = this.findAngleInRadians(
          isHalf ? percentOfDonutLength / 2 : percentOfDonutLength,
          radius
        );
        const translatedAngle =
          TRANSLATE_ANGLE_IN_RADIANS + PRECEDING_SEGMENTS_ANGLE + angle / 2; // angle/2 -> put label in the center of arc
        const xaxis = radius * Math.cos(translatedAngle) + CIRCLE_CENTER;
        const yaxis = radius * Math.sin(translatedAngle) + CIRCLE_CENTER;

        PRECEDING_SEGMENTS_ANGLE += angle;
        PRECEDING_SEGMENTS_LENGTH += isHalf
          ? percentOfDonutLength / 2
          : percentOfDonutLength;

        return {
          line: isHalf ? percentOfDonutLength / 2 : percentOfDonutLength,
          gap: isHalf ? gap + percentOfDonutLength / 2 : gap,
          percent: percentOfMaxValue.toFixed(0) + '%',
          offset: index == 0 ? FIRST_SEGMENT_OFFSET : currentSegmentOffset,
          color: colors[index],
          text: {
            x: xaxis,
            y: yaxis,
          },
          PRECEDING_SEGMENTS_LENGTH,
        };
      });
    },
  },
  methods: {
    findAngleInRadians(arcLength, radius) {
      // Arc length (L) = (θ/360) * 2πr  ==> θ = (L/2πr) * 360 (result in degree)
      // L = rθ  ==> θ = L/r  (result in radian)
      const angle = arcLength / radius;
      console.log({ angle });
      return angle;
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

text {
  font-size: 0.85rem;
}
</style>
