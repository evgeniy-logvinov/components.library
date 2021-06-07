<template>
  <div>
    <svg :height="height" :width="width">
      <g>
        <path
          v-for="(value, index) in sortedValues" :key="value+index"
          :d="describeArc(cx, cy, 50, 0, 90)"
          :stroke="colors[index]"
          :stroke-width="strokeWidth"
          :stroke-dasharray="adjustedCircumference"
          :stroke-dashoffset="calculateStrokeDashOffset(value, circumference)"
          :transform="returnCircleTransformValue(index)"
          fill="transparent"
          @click="onClick(index)"
        >
          <!-- <rect> -->
            <title>{{}}</title>
          <!-- </rect> -->
        </path>
      </g>
      <g>
        <text x="50%" y="90%" text-anchor="middle" :stroke="colors[selected]" font-size="16px">{{sortedValues[selected]}}%</text>
      </g>
    </svg>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue, Watch } from 'vue-property-decorator';

interface ChartData {
  degrees: number;
}

/**
 * https://css-tricks.com/building-a-donut-chart-with-vue-and-svg/
 * 
 * http://jsbin.com/quhujowota/1/edit?html,js
 * https://codepen.io/vineethtrv/pen/xGjQOX
 * https://webdesign.tutsplus.com/tutorials/how-to-build-a-css-only-semi-circle-donut-chart--cms-26997
 * https://codepen.io/janverstr/pen/PvPoaV
 */
@Component({components: {}})
export default class VcSemiDonutChart extends Vue {

  @Prop({ required: true })
  private initialValues!: number[];

  @Prop({ required: true })
  private colors!: string[];

  @Prop({ default: 140 })
  private width!: number;

  @Prop({ default: 55 })
  private radius!: number;

  private selected: number = 0;
  private piePart: number = 2;
  private angleOffset: number = 0;
  private strokeWidth: number = 25;
  private chartData: ChartData[] = [];
  private height: number = 70;
  private cx: number = 70;
  private cy: number = 70;
  private sortedValues: number[] = [];

  private created(): void {
    const summ = this.initialValues.reduce((acc, val) => acc + val);
    this.sortedValues = this.initialValues;
    this.calculateChartData();
  }

  private onClick(index): void {
    this.selected = index;
  }

  private calculateChartData(): void {
    this.sortedValues.forEach((dataVal: number) => {
      this.chartData.push({ degrees: this.angleOffset });
      this.angleOffset = this.dataPercentage(dataVal) * 360 + this.angleOffset;
    });
  }

  private calculateStrokeDashOffset(dataVal: number, circumference: number): number {
    return this.dataPercentage(dataVal) * circumference;
  }

  private dataPercentage(dataVal: number): number {
    return dataVal / this.dataTotal / this.piePart;
  }

  private returnCircleTransformValue(index: number): string {
    return `rotate(${this.chartData[index].degrees}, ${this.cx}, ${this.cy})`;
  }

  private polarToCartesian(centerX, centerY, radius, angleInDegrees) {
    const angleInRadians = (angleInDegrees + 180) * Math.PI / 180.0;

    return {
      x: centerX + (radius * Math.cos(angleInRadians)),
      y: centerY + (radius * Math.sin(angleInRadians)),
    };
  }

  private describeArc(x: number, y: number, radius: number, startAngle: number, endAngle: number): string {
    const start = this.polarToCartesian(x, y, radius, endAngle);
    const end = this.polarToCartesian(x, y, radius, startAngle);

    const largeArcFlag = endAngle - startAngle <= 180 ? '0' : '1';

    const d = [
        'M', start.x, start.y,
        'A', radius, radius, 0, largeArcFlag, 0, end.x, end.y,
    ].join(' ');

    return d;
  }

  // adjust the circumference to add small white gaps
  get adjustedCircumference(): number {
    return this.circumference - 2;
  }

  get circumference(): number {
    return 2 * Math.PI * this.radius;
  }

  get dataTotal(): number {
    return this.initialValues.reduce((acc, val) => acc + val);
  }

  @Watch('width', {immediate: true})
  private onWidth(val: number): void {
    this.cx = this.width / 2;
    this.cy = this.width / 2;
    this.height = this.width / 2;
  }
}
</script>
<style lang="scss" scoped>
rect {
  width: 100%;
  height: 100%;
  fill: #69c;
  stroke: #069;
  stroke-width: 5px;
  opacity: 0.5
}
</style>