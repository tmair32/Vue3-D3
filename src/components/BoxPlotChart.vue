<template>
  <div class="flex justify-center">
    <svg :width="svgWidth" :height="svgHeight">
      <g :transform="plotTransform">
        <g class="box" v-for="item in chartData" :key="item[xKey]">
          <line
            class="stroke-black"
            :x1="xScale(item[xKey])"
            :x2="xScale(item[xKey])"
            :y1="yScale(item.min)"
            :y2="yScale(item.max)"
          />
          <rect
            class="fill-gray-400"
            :x="xScale(item[xKey]) - boxWidth / 2"
            :y="yScale(item.q3)"
            :width="boxWidth"
            :height="yScale(item.q1) - yScale(item.q3)"
          />
          <line
            class="stroke-black"
            v-for="(mmm, idx) of ['min', 'mid', 'max']"
            :key="idx"
            :x1="xScale(item[xKey]) - boxWidth / 2"
            :x2="xScale(item[xKey]) + boxWidth / 2"
            :y1="yScale(item[mmm])"
            :y2="yScale(item[mmm])"
          />
        </g>
      </g>
    </svg>
  </div>
</template>
<script lang="ts">
import {
  computed,
  // ComputedRef,
  defineComponent,
  onMounted,
  PropType,
  reactive,
  toRefs,
} from "vue";

import { max, min, scaleBand, scaleLinear } from "d3";

interface marginType {
  top: number;
  right: number;
  bottom: number;
  left: number;
}

interface chartDataItem {
  [key: string]: string | number;
  name: string;
  q1: number;
  q3: number;
  mid: number;
  min: number;
  max: number;
}

type chartDataItems = Array<chartDataItem>;

type State = {
  margin: marginType;
  svgWidth: number;
  svgHeight: number;
  width: number;
  height: number;
  boxWidth: number;
};

const state = reactive<State>({
  margin: {
    top: 10,
    right: 30,
    bottom: 30,
    left: 40,
  },
  svgWidth: 540,
  svgHeight: 400,
  width: 0,
  height: 0,
  boxWidth: 100,
});

const computedData = (chartData: chartDataItems, xKey: string) => {
  const plotTransform = computed(
    () => `translate(${state.margin.left}, ${state.margin.top})`
  );

  const dataMax = computed(() => {
    return max(chartData, (d) => d.max as number);
  });

  const dataMin = computed(() => {
    return min(chartData, (d) => d.min as number);
  });

  const xScale = computed(() => {
    return scaleBand()
      .range([0, state.width])
      .domain(chartData.map((d) => d[xKey] as string))
      .paddingInner(1)
      .paddingOuter(0.5);
  });

  const yScale = computed(() => {
    return scaleLinear()
      .range([state.height, 0])
      .domain([
        (dataMin.value as number) > 0 ? 0 : (dataMin.value as number),
        dataMax.value as number,
      ]);
  });

  return {
    plotTransform,
    xScale,
    yScale,
  };
};

export default defineComponent({
  props: {
    xKey: {
      type: String,
      default: "",
    },
    yKey: {
      type: String,
      default: "",
    },
    chartData: {
      type: Array as PropType<
        {
          [key: string]: string | number;
          name: string;
          q1: number;
          q3: number;
          mid: number;
          min: number;
          max: number;
        }[]
      >,
      default: () => [
        {
          name: "Roses",
          q1: 100,
          q3: 150,
          mid: 120,
        },
      ],
    },
  },
  setup(props) {
    const { plotTransform, xScale, yScale } = computedData(
      props.chartData,
      props.xKey
    );

    onMounted(() => {
      state.width = state.svgWidth - state.margin.left - state.margin.right;
      state.height = state.svgHeight - state.margin.top - state.margin.bottom;
    });
    return {
      ...toRefs(state),
      plotTransform,
      xScale,
      yScale,
    };
  },
});
</script>
<style lang=""></style>
