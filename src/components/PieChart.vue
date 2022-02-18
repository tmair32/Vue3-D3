<template>
  <div ref="containerRef" class="flex justify-center">
    <svg :width="svgWidth" :height="svgHeight">
      <g
        class="stroke-white stroke-2 stroke-join-auto"
        :transform="pieTransform"
      >
        <path class="arc" v-for="item in chartData" :key="item[xKey]" />
      </g>
      <g class="font-sans" :transform="pieTransform">
        <text class="text" v-for="item in chartData" :key="item[xKey]" />
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

const getLegendPosition = (
  type: string,
  mode: string,
  i: number,
  radius: number
) => {
  if (type === "x") {
    return mode === "dot" ? -30 : -15;
  } else if (type === "y") {
    return mode === "dot" ? radius + 15 : radius + 20;
  }
};

const drawChart = (chartData: chartDataItems, xKey: string, yKey: string) => {
  const containerRef = ref();

  const getPieData = pie<chartDataItem>()
    .sort(null)
    .value((d) => {
      return d[yKey] as number;
    });

  const pieTransform = computed(
    () => `translate(${state.svgWidth / 2}, ${state.svgHeight / 2})`
  );

  const color = scaleOrdinal(schemeCategory10);
  const pieData = getPieData(chartData);
  const radius = computed(() => Math.min(state.svgWidth, state.svgHeight) / 2);

  const chartLoad = () => {
    const arcPath = arc<PieArcDatum<chartDataItem>>()
      .innerRadius(0)
      .outerRadius(radius.value);

    selectAll(".arc")
      .data(pieData)
      .join("path")
      .transition()
      .delay((_, i: number) => {
        return i * 150;
      })
      .duration(1000)
      .attr("d", arcPath)
      .attr("fill", (d) => color(d.data[xKey] as string));
  };

  const textLoad = () => {
    const arcLabel = arc<PieArcDatum<chartDataItem>>()
      .innerRadius(0)
      .outerRadius(radius.value);
    selectAll(".text")
      .data(pieData)
      .join("text")
      .attr("transform", (d) => `translate(${arcLabel.centroid(d)})`)
      .text((d) => d.data[xKey]);
  };

  onMounted(() => {
    state.svgWidth = containerRef.value.offsetWidth * 0.25;
    state.svgHeight = containerRef.value.offsetWidth * 0.25;

    chartLoad();
    textLoad();
  });

  return {
    chartLoad,
    containerRef,
    pieTransform,
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
    const { containerRef, pieTransform } = drawChart(
      props.chartData,
      props.xKey,
      props.yKey
    );
    return {
      ...toRefs(state),
      containerRef,
      pieTransform,
    };
  },
});
</script>
<style scoped>
.arc text {
  font: 12px;
  text-anchor: middle;
}

.title {
  fill: green;
  font-weight: italic;
}
</style>
