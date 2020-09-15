<template>
  <div class="wrapper" :style="wrapperStyle">
    <!-- Default progress -->
    <svg
      :viewBox="`0 0 ${diameter} ${diameter}`"
      :width="diameter"
      :height="diameter"
      class="progressCircle"
      v-if="size === 'default'"
    >
      <g
        :transform="
          `rotate(${rotate},${diameter / 2},${diameter / 2}) ${
            reverse ? 'scale(1,-1) translate(0, -' + diameter + ')' : ''
          }`
        "
      >
        <circle
          :cx="diameter / 2"
          :cy="diameter / 2"
          :r="getRadius()"
          :stroke="inactiveStrokeColor"
          :stroke-width="defaultStrokeWidth"
          fill="none"
          :stroke-dasharray="getLengths()"
        />
        <path
          :d="
            describeArc(diameter / 2, diameter / 2, getRadius(), 0, activeEnd())
          "
          fill="none"
          :stroke="strokeColor"
          :stroke-width="defaultStrokeWidth"
          :stroke-dasharray="getLengths()"
        />
      </g>
    </svg>
    <!-- Compact progress -->
    <svg
      class="progressCircle"
      v-if="size === 'compact'"
      :width="diameter"
      :height="diameter"
      version="1.1"
      xmlns="http://www.w3.org/2000/svg"
    >
      <defs>
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
    </svg>
    <!-- Progress values -->
    <div class="content">
      <span class="progressValue" :style="{ fontSize: valueFontSize }">
        <span v-if="size === 'compact'">
          {{ currentValue }}/{{ maxValue }}
        </span>
        <span v-else> {{ currentValue }} / {{ maxValue }} </span>
      </span>
      <span v-if="label !== ''" class="label">{{ label }}</span>
    </div>
  </div>
</template>

<script>
export default {
  name: "ProgressCircle",
  props: {
    maxValue: { type: Number, required: true },
    currentValue: { type: Number, required: true },
    label: { type: String, required: false, default: "" },
    size: {
      type: String,
      validator: val => ["default", "compact"].includes(val),
      default: "default"
    }
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
      diameter: this.size === "default" ? 200 : 120,
      valueFontSize: this.size === "default" ? "1rem" : "1.5rem",
      strokeColor: "#ff0071",
      inactiveStrokeColor: "#d6d6d8",
      compactStrokeWidth: 7,
      compactStrokeLinecap: "butt",
      compactAnimateSpeed: 1000,
      animationIncrements: 1000 / 60,
      dashCount: 80,
      defaultStrokeWidth: 20,
      dashSpacing: 1 / 4,
      rotate: -90,
      reverse: false
    };
  },
  computed: {
    // Compact progress
    radius() {
      return this.diameter / 2;
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
    },
    // Default progress
    defaultProgressValue() {
      return Math.floor((this.currentValue * this.dashCount) / this.maxValue);
    }
  },
  methods: {
    // Compact progress
    changeProgress() {
      this.strokeDashoffset =
        ((100 - this.finishedPercentage) / 100) * this.circumference;
      const angleOffset = (this.currentValue - 1) * this.circleSlice;
      let i = angleOffset / this.animateSlice;
      const incrementer = Math.abs(i - this.totalPoints) / this.totalPoints;
      const isMoveForward = i < this.totalPoints;
    },
    // Default progress
    // Determine the 'end' angle of the path for the active dashes in degrees.
    activeEnd() {
      if (this.defaultProgressValue == 0) {
        return 0;
      }
      return (
        360 *
        (this.defaultProgressValue * this.dashPerc() +
          (this.defaultProgressValue - 1) * this.spacePerc())
      );
    },
    // An array of the length of the dash & the length of the space between dashes
    getLengths() {
      return [
        2 * Math.PI * this.getRadius() * this.dashPerc(),
        2 * Math.PI * this.getRadius() * this.spacePerc()
      ];
    },
    // The space beween dashes as a percentage of the total length
    spacePerc() {
      return this.dashSpacing / this.dashCount;
    },
    // The length of a dash as a percentage of the total length
    dashPerc() {
      return (1 - this.dashSpacing) / this.dashCount;
    },
    // Radius of the circle arc
    getRadius() {
      return (this.diameter - this.defaultStrokeWidth) / 2;
    },
    // SVG path definition requires points in cartesian space
    polarToCartesian(cx, cy, radius, degrees) {
      const radians = (degrees * Math.PI) / 180.0;
      return {
        x: cx + radius * Math.cos(radians),
        y: cy + radius * Math.sin(radians)
      };
    },
    // Path definition for circular arc
    describeArc(cx, cy, radius, startDegrees, endDegrees) {
      const start = this.polarToCartesian(cx, cy, radius, startDegrees);
      const end = this.polarToCartesian(cx, cy, radius, endDegrees);
      let largeArc = Math.abs(endDegrees - startDegrees) < 180 ? 0 : 1;
      let sweep = endDegrees < startDegrees ? 0 : 1;
      return `M${start.x} ${start.y} A${radius} ${radius} 0 ${largeArc} ${sweep} ${end.x} ${end.y}`;
    }
  },
  watch: {
    maxValue() {
      this.changeProgress();
    },
    currentValue() {
      this.changeProgress();
    }
  },
  created() {
    this.changeProgress();
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
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
