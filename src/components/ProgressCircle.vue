<template>
  <div class="wrapper" :style="wrapperStyle">
    <svg
      :viewBox="`0 0 ${diameter} ${diameter}`"
      :width="diameter"
      :height="diameter"
      class="progressCircle"
    >
      <!-- Default size START -->
      <g
        v-if="size === SIZE_DEFAULT"
        :transform="`rotate(${rotate},${halfDiameter},${halfDiameter})`"
      >
        <circle
          :cx="halfDiameter"
          :cy="halfDiameter"
          :r="getRadius()"
          :stroke="inactiveStrokeColor"
          :stroke-width="defaultStrokeWidth"
          fill="none"
          :stroke-dasharray="getLengthOfDashAndSpaceBetween()"
        />
        <path
          :d="
            describeArc(
              halfDiameter,
              halfDiameter,
              getRadius(),
              0,
              getActiveEnd()
            )
          "
          fill="none"
          :stroke="strokeColor"
          :stroke-width="defaultStrokeWidth"
          :stroke-dasharray="getLengthOfDashAndSpaceBetween()"
        />
      </g>
      <!-- Default size END -->
      <!-- Compact size START -->
      <defs v-if="size === SIZE_COMPACT">
        <radialGradient
          :id="'radial-gradient' + _uid"
          :fx="gradient.fx"
          :fy="gradient.fy"
          :cx="gradient.cx"
          :cy="gradient.cy"
          :r="gradient.r"
        >
          <stop offset="30%" :stop-color="strokeColor" />
          <stop offset="100%" :stop-color="strokeColor" />
        </radialGradient>
      </defs>
      <circle
        v-if="size === SIZE_COMPACT"
        :r="innerCircleRadius"
        :cx="radius"
        :cy="radius"
        fill="transparent"
        :stroke="inactiveStrokeColor"
        :stroke-dasharray="circumference"
        stroke-dashoffset="0"
        :stroke-linecap="compactStrokeLinecap"
        :style="strokeStyle"
      ></circle>
      <circle
        v-if="size === SIZE_COMPACT"
        :transform="'rotate(270, ' + radius + ',' + radius + ')'"
        :r="innerCircleRadius"
        :cx="radius"
        :cy="radius"
        fill="transparent"
        :stroke="'url(#radial-gradient' + _uid + ')'"
        :stroke-dasharray="circumference"
        :stroke-dashoffset="circumference"
        :stroke-linecap="compactStrokeLinecap"
        :style="progressStyle"
      ></circle>
      <!-- Compact size END -->
    </svg>
    <!-- Values and label -->
    <div class="content">
      <span class="progressValue" :style="{ fontSize: valueFontSize }">
        <span v-if="size === SIZE_COMPACT">
          {{ currentValue }}/{{ maxValue }}
        </span>
        <span v-else> {{ currentValue }} / {{ maxValue }} </span>
      </span>
      <span v-if="label !== ''" class="label">{{ label }}</span>
    </div>
  </div>
</template>

<script>
const [SIZE_DEFAULT, SIZE_COMPACT] = ["default", "compact"];

export default {
  name: "ProgressCircle",
  props: {
    maxValue: { type: Number, required: true },
    currentValue: { type: Number, required: true },
    label: { type: String, required: false, default: "" },
    size: {
      type: String,
      validator: val => [SIZE_DEFAULT, SIZE_COMPACT].includes(val),
      default: SIZE_DEFAULT
    }
  },
  beforeCreate() {
    this.SIZE_DEFAULT = SIZE_DEFAULT;
    this.SIZE_COMPACT = SIZE_COMPACT;
  },
  data() {
    return {
      gradient: {
        fx: 0.99,
        fy: 0.5,
        cx: 0.5,
        cy: 0.5,
        r: 0.65
      },
      strokeDashoffset: 0,
      diameter: this.size === SIZE_DEFAULT ? 200 : 120,
      valueFontSize: this.size === SIZE_DEFAULT ? "1.2rem" : "1.5rem",
      strokeColor: "#ff0071",
      inactiveStrokeColor: "#d6d6d8",
      compactStrokeWidth: 7,
      compactStrokeLinecap: "butt",
      compactAnimateSpeed: 1000,
      animationIncrements: 1000 / 60,
      dashCount: 80,
      defaultStrokeWidth: 20,
      dashSpacing: 1 / 4,
      rotate: -90
    };
  },
  computed: {
    defaultProgressValue() {
      return Math.floor((this.currentValue * this.dashCount) / this.maxValue);
    },
    halfDiameter() {
      return this.diameter / 2;
    },
    radius() {
      return this.halfDiameter;
    },
    circumference() {
      return Math.PI * this.innerCircleDiameter;
    },
    stepSize() {
      if (this.maxValue === 0) {
        return 0;
      }
      return 100 / this.maxValue;
    },
    finishedPercentage() {
      return this.stepSize * this.currentValue;
    },
    circleSlice() {
      return (2 * Math.PI) / this.maxValue;
    },
    animateSlice() {
      return this.circleSlice / this.totalPoints;
    },
    innerCircleDiameter() {
      return this.diameter - this.compactStrokeWidth * 2;
    },
    innerCircleRadius() {
      return this.innerCircleDiameter / 2;
    },
    totalPoints() {
      return this.compactAnimateSpeed / this.animationIncrements;
    },
    wrapperStyle() {
      return {
        height: `${this.diameter}px`,
        width: `${this.diameter}px`
      };
    },
    progressStyle() {
      return {
        height: `${this.diameter}px`,
        width: `${this.diameter}px`,
        strokeWidth: `${this.compactStrokeWidth}px`,
        strokeDashoffset: this.strokeDashoffset,
        transition: `stroke-dashoffset ${this.compactAnimateSpeed}ms linear`
      };
    },
    strokeStyle() {
      return {
        height: `${this.diameter}px`,
        width: `${this.diameter}px`,
        strokeWidth: `${this.compactStrokeWidth}px`
      };
    }
  },
  methods: {
    getActiveEnd() {
      if (this.defaultProgressValue == 0) {
        return 0;
      }
      return (
        360 *
        (this.defaultProgressValue * this.getDashPerc() +
          (this.defaultProgressValue - 1) * this.getSpacePerc())
      );
    },
    getLengthOfDashAndSpaceBetween() {
      return [
        2 * Math.PI * this.getRadius() * this.getDashPerc(),
        2 * Math.PI * this.getRadius() * this.getSpacePerc()
      ];
    },
    getSpacePerc() {
      return this.dashSpacing / this.dashCount;
    },
    getDashPerc() {
      return (1 - this.dashSpacing) / this.dashCount;
    },
    getRadius() {
      return (this.diameter - this.defaultStrokeWidth) / 2;
    },
    polarToCartesian(cx, cy, radius, degrees) {
      const radians = (degrees * Math.PI) / 180.0;
      return {
        x: cx + radius * Math.cos(radians),
        y: cy + radius * Math.sin(radians)
      };
    },
    describeArc(cx, cy, radius, startDegrees, endDegrees) {
      const start = this.polarToCartesian(cx, cy, radius, startDegrees);
      const end = this.polarToCartesian(cx, cy, radius, endDegrees);
      const largeArc = Math.abs(endDegrees - startDegrees) < 180 ? 0 : 1;
      const sweep = endDegrees < startDegrees ? 0 : 1;
      return `M${start.x} ${start.y} A${radius} ${radius} 0 ${largeArc} ${sweep} ${end.x} ${end.y}`;
    },
    changeCompactProgress() {
      this.strokeDashoffset =
        ((100 - this.finishedPercentage) / 100) * this.circumference;
    }
  },
  watch: {
    maxValue() {
      if (this.size === SIZE_COMPACT) {
        this.changeCompactProgress();
      }
    },
    currentValue() {
      if (this.size === SIZE_COMPACT) {
        this.changeCompactProgress();
      }
    }
  },
  created() {
    if (this.size === SIZE_COMPACT) {
      this.changeCompactProgress();
    }
  }
};
</script>

<style scoped>
.wrapper {
  position: relative;
  margin: 20px;
}
.progressCircle {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translateX(-50%) translateY(-50%);
}
.content {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translateX(-50%) translateY(-50%);
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
.progressValue {
  font-weight: bold;
  color: #26262a;
}
.label {
  font-weight: 100;
  color: #26262a;
}
</style>
