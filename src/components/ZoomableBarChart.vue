<template>
  <div ref="containerRef" class="flex justify-center">
    <svg
      ref="chartRef"
      :width="svgWidth + 55"
      :height="svgHeight + 55"
      class="m-50px"
      @wheel.prevent="zoomzoom"
    >
      <g>
        <rect
          v-for="item in chartData"
          class="bar-positive fill-violet-500 hover:fill-rose-400"
          :key="item[xKey]"
          :x="xScale(item[xKey])"
          :y="yScale(0)"
          :width="xScale.bandwidth()"
          :height="0"
        />
      </g>
      <g>
        <g ref="axisBottomRef" :transform="axisBottomTransform" />
        <g ref="axisLeftRef" :transform="axisLeftTransform" />
      </g>
    </svg>
  </div>
</template>
<script lang="ts">
import {
  computed,
  ComputedRef,
  defineComponent,
  onMounted,
  PropType,
  reactive,
  ref,
  toRefs,
  watch,
} from "vue";

import {
  axisBottom,
  axisLeft,
  max,
  min,
  scaleBand,
  scaleLinear,
  select,
  selectAll,
  zoom,
  zoomTransform,
} from "d3";

type State = {
  svgWidth: number;
  redrawToggle: boolean;
};

interface Ref<T> {
  value: T;
}

interface chartDataItem {
  [key: string]: string | number;
  name: string;
  amount: number;
}

type chartDataItems = Array<chartDataItem>;

const state = reactive<State>({
  svgWidth: 0,
  redrawToggle: true,
});

const getComputedData = (
  chartData: chartDataItems,
  xKey: string,
  yKey: string
) => {
  // || getComputedData ||
  const dataMax = computed(() => {
    return max(chartData, (d) => {
      return d[yKey] as number;
    });
  });

  const dataMin = computed(() => {
    return min(chartData, (d) => {
      return d[yKey] as number;
    });
  });

  const xScale = computed(() => {
    return scaleBand()
      .rangeRound([20, state.svgWidth])
      .padding(0.1)
      .domain(
        chartData.map((d) => {
          return d[xKey] as string;
        })
      );
  });

  const yScale = computed(() => {
    return scaleLinear()
      .rangeRound([svgHeight.value, 20])
      .domain([
        (dataMin.value as number) > 0 ? 0 : (dataMin.value as number),
        dataMax.value as number,
      ]);
  });

  const svgHeight = computed(() => {
    return state.svgWidth / 1.61803398875;
  });

  return {
    dataMax,
    dataMin,
    xScale,
    yScale,
    svgHeight,
  };
};

const drawChart = (
  chartData: chartDataItems,
  yKey: string,
  yScale: ComputedRef,
  svgHeight: ComputedRef<number>
) => {
  const containerRef = ref();

  const chartLoad = () => {
    selectAll(".bar-positive")
      .data(chartData)
      .transition()
      .delay((_, i: number) => {
        return i * 150;
      })
      .duration(1000)
      .attr("y", (d) => {
        return yScale.value(d[yKey] as number);
      })
      .attr("height", (d) => {
        return svgHeight.value - yScale.value(d[yKey] as number);
      });
  };

  const chartResize = () => {
    window.addEventListener("resize", () => {
      state.redrawToggle = false;
      setTimeout(() => {
        state.redrawToggle = true;
        state.svgWidth = containerRef.value.offsetWidth * 0.75;
        chartLoad();
      }, 300);
    });
  };

  onMounted(() => {
    state.svgWidth = containerRef.value.offsetWidth * 0.75;
    chartResize();
    chartLoad();
  });

  return {
    containerRef,
    ...toRefs(state),
  };
};

const drawAxis = (
  chartData: chartDataItems,
  xScale: ComputedRef,
  yScale: ComputedRef
) => {
  const axisBottomRef: Ref<SVGGElement | undefined> = ref();
  const axisLeftRef: Ref<SVGGElement | undefined> = ref();

  const axisBottomTransform = computed(
    () => `translate(0, ${yScale.value(0) + 15})`
  );
  const axisLeftTransform = computed(() => `translate(20, 0)`);

  onMounted(() => {
    watch(
      xScale,
      () => {
        if (!axisBottomRef.value) return;
        select(axisBottomRef.value).call(axisBottom(xScale.value));
      },
      {
        immediate: true,
        deep: true,
      }
    );
    watch(
      yScale,
      () => {
        if (!axisLeftRef.value) return;
        select(axisLeftRef.value).call(axisLeft(yScale.value));
      },
      {
        immediate: true,
        deep: true,
      }
    );
  });

  return {
    axisBottomRef,
    axisLeftRef,
    axisBottomTransform,
    axisLeftTransform,
  };
};

const zoomChart = (svgHeight: ComputedRef) => {
  const chartRef: Ref<SVGSVGElement | undefined> = ref();

  onMounted(() => {
    if (!chartRef.value) return;
    const svg = select(chartRef.value);
    const zoomBehavior = zoom<SVGSVGElement>()
      .scaleExtent([0.5, 5])
      .translateExtent([
        [0, 0],
        [state.svgWidth, svgHeight.value],
      ])
      .on("zoom", () => {
        const zoomTranslate = zoomTransform(svg.node() as Element);
        console.log(zoomTranslate);
      });
    svg.call(zoomBehavior);
  });

  return { chartRef };
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
    yKey: {
      type: String,
      required: true,
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
    const { xScale, yScale, svgHeight } = getComputedData(
      props.chartData,
      props.xKey,
      props.yKey
    );

    const { containerRef } = drawChart(
      props.chartData,
      props.yKey,
      yScale,
      svgHeight
    );

    const {
      axisBottomRef,
      axisLeftRef,
      axisBottomTransform,
      axisLeftTransform,
    } = drawAxis(props.chartData, xScale, yScale);

    const { chartRef } = zoomChart(svgHeight);

    return {
      ...toRefs(state),
      axisBottomRef,
      axisLeftRef,
      axisBottomTransform,
      axisLeftTransform,
      containerRef,
      chartRef,
      xScale,
      yScale,
      svgHeight,
    };
  },
});
</script>
<style lang=""></style>
