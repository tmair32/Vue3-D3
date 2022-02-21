<template>
  <div class="flex justify-center">
    <svg :width="svgWidth" :height="svgHeight">
      <g :transform="chartTransform">
        <g
          class="grouped"
          v-for="groupedItem in chartData"
          :key="groupedItem[xKey]"
        >
          <rect
            v-for="item in groupedItem.map((data) => {
              data.data1, data.data2, data.data3, data.data4;
            })"
            class="bar-positive fill-violet-500 hover:fill-rose-400"
            :key="item[xKey]"
            :x="keyScale(item[xKey])"
            :y="yScale(item[xKey])"
            :width="keyScale.bandwidth()"
            :height="yScale(item[xKey])"
          />
        </g>
      </g>
    </svg>
  </div>
</template>
<script lang="ts">
import { computed, defineComponent, PropType, reactive, toRefs } from "vue";
import { max, min, scaleBand, scaleLinear } from "d3";

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
  svgWidth: 540,
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
  const chartTransform = computed(
    () => `translate(${state.margin.left}, ${state.margin.top})`
  );
  const dataKeys = Object.keys(chartData[0]).filter((key) => key !== "batch");

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
      .domain(chartData.map((d) => d.batch as string))
      .paddingInner(1)
      .paddingOuter(0.5);
  });

  const keyScale = computed(() => scaleBand().domain(dataKeys));

  const yScale = computed(() => {
    return scaleLinear()
      .range([state.height, 0])
      .domain([
        (dataMin.value as number) > 0 ? 0 : (dataMin.value as number),
        dataMax.value as number,
      ]);
  });

  const changeChartData = computed(() => {
    return chartData.map((data) => {
      return {
        [xKey]: data[xKey],
        data: {
          ...
        },
      };
    });
  });

  return {
    chartTransform,
    keyScale,
    xScale,
    yScale,
  };
};

export default defineComponent({
  props: {
    backgroundWidth: {
      type: Number,
      // required: true,
    },
    backgroundHeight: {
      type: Number,
      // required: true,
    },
    width: {
      type: Number,
      // required: true,
    },
    height: {
      type: Number,
      // required: true,
    },
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
    const { chartTransform, keyScale, xScale, yScale } = computedData(
      props.chartData
    );
    const noBatchData = props.chartData.map((data) => {
      data.data1, data.data2, data.data3, data.data4;
    });
    return {
      ...toRefs(state),
      chartTransform,
      keyScale,
      noBatchData,
      xScale,
      yScale,
    };
  },
});
</script>
<style lang=""></style>
