<template>
  <div ref="containerRef" class="flex justify-center">
    <svg ref="chartRef" :width="svgWidth" :height="svgHeight">
      <g v-for="item in chartData" class="slice" :key="item[xKey]" />
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
  ref,
  toRefs,
} from "vue";
import {
  arc,
  pie,
  PieArcDatum,
  scaleOrdinal,
  schemeCategory10,
  selectAll,
} from "d3";

interface Ref<T> {
  value: T;
}

interface chartDataItem {
  [key: string]: string | number;
  name: string;
  amount: number;
}

type chartDataItems = Array<chartDataItem>;

type State = {
  svgWidth: number;
  svgHeight: number;
};

const state = reactive<State>({
  svgWidth: 0,
  svgHeight: 0,
});

const drawChart = (chartData: chartDataItems, yKey: string) => {
  const containerRef = ref();
  const color = scaleOrdinal(schemeCategory10);

  const pieChartGenerator = computed(() => {
    return pie<chartDataItem>()
      .sort(null)
      .value((d) => d[yKey] as number);
  });
  const piePathData = computed(() => pieChartGenerator.value(chartData));
  const radius = Math.min(state.svgHeight, state.svgWidth) / 2;
  const arcPath = arc<PieArcDatum<chartDataItem>>()
    .innerRadius(radius * 0)
    .outerRadius(radius * 1);

  const chartLoad = () => {
    selectAll(".slice")
      .data(piePathData.value)
      .enter()
      .append("g")
      .attr("class", "arc")
      .append("path")
      .attr("d", arcPath)
      .attr("fill", (d, i) => {
        console.log(d, i);
        return color(i);
      });
  };

  onMounted(() => {
    state.svgWidth = containerRef.value.offsetWidth * 0.75;
    state.svgHeight = containerRef.value.offsetWidth * 0.75;
    chartLoad();
  });

  return { containerRef, piePathData };
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
        { [key: string]: string | number; name: string; amount: number }[]
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
    const { containerRef, piePathData } = drawChart(
      props.chartData,
      props.yKey
    );

    return {
      ...toRefs(state),
      containerRef,
      piePathData,
    };
  },
});
</script>
<style lang=""></style>
