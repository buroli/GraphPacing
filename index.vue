<template>
  <svg
    :style="{ width: height + 'px', height: height + 'px'}"
    viewBox="0 0 36 36"
    class="circular-chart">
    <path
      class="circle-bg"
      d="M18 2.0845
          a 15.9155 15.9155 0 0 1 0 31.831
          a 15.9155 15.9155 0 0 1 0 -31.831"
    />
    <path
      :stroke-dasharray="`${ percentFilled }, 100`"
      :stroke="whichColor"
      class="circle"
      d="M18 2.0845
          a 15.9155 15.9155 0 0 1 0 31.831
          a 15.9155 15.9155 0 0 1 0 -31.831"
    />
    <g
      v-if="percentPacing > 0"
      :transform="`translate(${ coord.x }, ${ coord.y })`"
      class="pacing">
      <circle
        cx="2"
        cy="2"
        r="2" />
    </g>
    <text
      v-if="showPercentage"
      x="18.5"
      y="21"
      class="percentage">{{ ~~graphView }}%</text>
  </svg>
</template>
<script>
/*
    radian = -angleInDegrees * Math.PI / 180.0;

    //get the cartesian x coordinate (centerX = x coordinate of the center of the circle == 36 in our case)
    x = (diameter - radius) * Math.sin(angleInRadians) * radius;

    //get the cartesian y coordinate (centerY = y coordinate of the center of the circle == 36 in our case)
    y = radius * Math.cos(angleInRadians) * radius;
  */

import { ColorCodes, ColorMapping } from './constants/CampaignColors'

export default {
  name: 'GraphPacing',
  props: {
    percentFilled: {
      type: [Number, String],
      required: true
    },
    percentPacing: {
      type: [Number, String],
      required: true
    },
    width: {
      type: Number,
      default: 60
    },
    height: {
      type: Number,
      default: 60
    },
    // This property overrides the color mapping
    // useful when a campaign is paused or anything but running
    // so we don't bother computing the color
    colorOverride: {
      type: String,
      default: ''
    },
    showPercentage: {
      type: Boolean,
      default: true
    },
    parentView: {
      type: String
    }
  },
  data () {
    return {
      centerX: 15.9155, // center X of viewBox
      centerY: 15.9155, // center Y of viewBox
      radius: 15.9155, // radius of circle without border
      diameter: 31.831,
      currentView: ''
    }
  },
  computed: {
    computeDegre () {
      return (this.percentPacing * 360) / 100
    },
    computeRadian () {
      return parseFloat(((-this.computeDegre * Math.PI) / 180).toFixed(4))
    },
    computeX () {
      return (this.diameter - this.radius) - Math.sin(this.computeRadian) * this.radius
    },
    computeY () {
      return this.radius - Math.cos(this.computeRadian) * this.radius
    },
    coord () {
      return {x: this.computeX, y: this.computeY}
    },
    whichColor () {
      // Color override (Campaign might be paused)
      if (!!this.colorOverride) {
        return this.colorOverride
      }

      // Compute color according on threshold mapping
      const pacing = parseFloat(this.percentPacing);
      const delivery = parseFloat(this.percentFilled) * 100;
      let color = ColorCodes.ERROR;

      for (const threshold in ColorMapping) {
        if (pacing < threshold) {
          break
        }

        color = ColorMapping[threshold];
      }

      return color
    },
    graphView () {
      if(!!this.parentView){
        return this.parentView === 'pacing' ? this.percentPacing : this.percentFilled;
      }
      return this.percentFilled;
    }
  }
}
</script>
<style lang="scss" scoped>
  @import 'global_var';
  .circular-chart {
    display: block;
  }

  .circle-bg {
    fill: none;
    stroke: #eee;
    stroke-width: 2;
  }

  .circle {
    fill: none;
    stroke-width: 2.8;
    stroke-linecap: round;
    animation: progress 1s ease-out forwards;
  }
  .pacing{
    fill: #696B70;
    stroke: #fff;
    stroke-width: 1;
    // transition: transform .1s linear;
  }
  @keyframes progress {
    0% {
      stroke-dasharray: 0 100;
    }
  }

  .circular-chart.orange .circle {
    stroke: #ff9f00;
  }

  .circular-chart.green .circle {
    stroke: #4CC790;
  }

  .circular-chart.blue .circle {
    stroke: #3c9ee5;
  }

  .percentage {
    fill: #666F77;
    font-weight: 600;
    font-family: sans-serif;
    font-size: toRem(10);
    text-anchor: middle;
  }
</style>
