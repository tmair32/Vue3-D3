<template>
  <div class="flex justify-center">
    <svg :width="width" :height="height">
      <g :transform="chartTransform">
        <g
          class="grouped"
          v-for="groupedItem in changeChartData"
          :key="groupedItem[xKey]"
        >
          <rect
            v-for="(value, key) in groupedItem.d"
            class="bar-individual"
            :key="key"
            :x="keyScale(key) + xScale(groupedItem[xKey])"
            :y="yScale(value)"
            :width="25"
            :height="height - yScale(value)"
            :fill="colorScale(key)"
          />
        </g>
      </g>
    </svg>
  </div>
</template>
<script lang="ts">
import {
  computed,
  defineComponent,
  onMounted,
  PropType,
  reactive,
  toRefs,
} from "vue";
import {
  max,
  min,
  scaleBand,
  scaleLinear,
  scaleOrdinal,
  schemeCategory10,
} from "d3";

interface marginType {
  top: number;
  right: number;
  bottom: number;
  left: number;
}

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
  svgWidth: 640,
  svgHeight: 400,
  width: 0,
  height: 0,
  boxWidth: 100,
});

interface chartDataItem {
  [key: string]: string | number;
  name: string;
  data1: number;
  data2: number;
  data3: number;
  data4: number;
}

type chartDataItems = Array<chartDataItem>;

const computedData = (chartData: chartDataItems, xKey: string) => {
  const chartTransform = computed(() => `translate(${0}, ${state.margin.top})`);
  const dataKeys = Object.keys(chartData[0]).filter((key) => key !== "name");

  const dataMax = computed(() => {
    return max(chartData, (d) => {
      return Math.max(...dataKeys.map((key) => d[key] as number));
    });
  });

  const dataMin = computed(() => {
    return min(chartData, (d) => {
      return Math.min(...dataKeys.map((key) => d[key] as number));
    });
  });

  const xScale = computed(() => {
    return scaleBand()
      .range([0, state.width])
      .domain(chartData.map((d) => d[xKey] as string))
      .padding(0.2);
  });

  const keyScale = computed(() =>
    scaleBand().domain(dataKeys).range([0, xScale.value.bandwidth()])
  );

  const yScale = computed(() => {
    return scaleLinear()
      .range([state.height, 0])
      .domain([
        (dataMin.value as number) > 0 ? 0 : (dataMin.value as number),
        dataMax.value as number,
      ]);
  });

  const colorScale = computed(() => {
    return scaleOrdinal(schemeCategory10).domain(dataKeys);
  });

  const changeChartData = computed(() => {
    return chartData.map((data) => {
      const { name, ...d } = data;
      return {
        name,
        d,
      };
    });
  });

  return {
    chartTransform,
    changeChartData,
    colorScale,
    dataKeys,
    keyScale,
    xScale,
    yScale,
  };
};

export default defineComponent({
  props: {
    xKey: {
      type: String,
      required: true,
    },
    chartData: {
      type: Array as PropType<
        {
          [key: string]: string | number;
          name: string;
          data1: number;
          data2: number;
          data3: number;
          data4: number;
        }[]
      >,
      default: () => [
        {
          name: "Roses",
          amount: 25,
        },
      ],
    },
  },
  setup(props) {
    const {
      chartTransform,
      changeChartData,
      colorScale,
      dataKeys,
      keyScale,
      xScale,
      yScale,
    } = computedData(props.chartData, props.xKey);

    onMounted(() => {
      state.width = state.svgWidth - state.margin.left - state.margin.right;
      state.height = state.svgHeight - state.margin.top - state.margin.bottom;
    });
    return {
      ...toRefs(state),
      chartTransform,
      changeChartData,
      colorScale,
      dataKeys,
      keyScale,
      xScale,
      yScale,
    };
  },
});
</script>
<style lang=""></style>
