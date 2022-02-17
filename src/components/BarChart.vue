<template>
  <div ref="containerRef" class="flex justify-center">
    <svg ref="chartRef" :width="svgWidth" :height="svgHeight">
      <g>
        <rect
          v-for="item in chartData"
          class="bar-positive fill-violet-500 hover:fill-rose-400"
          :key="item[xKey]"
          :x="xScale(item[xKey])"
          :y="yScale(0)"
          :width="xScale.bandwidth()"
          :height="0"
          @mouseover="showTooltip"
          @mouseout="hideTooltip"
        />
      </g>
    </svg>
    <div v-show="tooltip" class="bg-gray-600">
      <p>abcdefg</p>
    </div>
  </div>
</template>
<script lang="ts">
import {
  defineComponent,
  PropType,
  reactive,
  ref,
  computed,
  onMounted,
  toRefs,
} from "vue";

import { scaleLinear, scaleBand, max, min, selectAll } from "d3";

type State = {
  svgWidth: number;
  redrawToggle: boolean;
  tooltip: boolean;
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
    const state = reactive<State>({
      svgWidth: 0,
      redrawToggle: true,
      tooltip: false,
    });

    const containerRef = ref();

    const chartLoad = () => {
      selectAll(".bar-positive")
        .data(props.chartData)
        .transition()
        .delay((_, i: number) => {
          return i * 150;
        })
        .duration(1000)
        .attr("y", (d) => {
          return yScale.value(d[props.yKey] as number);
        })
        .attr("height", (d) => {
          return svgHeight.value - yScale.value(d[props.yKey] as number);
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

    const dataMax = computed(() => {
      return max(props.chartData, (d) => {
        return d[props.yKey] as number;
      });
    });

    const dataMin = computed(() => {
      return min(props.chartData, (d) => {
        return d[props.yKey] as number;
      });
    });

    const xScale = computed(() => {
      return scaleBand()
        .rangeRound([0, state.svgWidth])
        .padding(0.1)
        .domain(
          props.chartData.map((d) => {
            return d[props.xKey] as string;
          })
        );
    });

    const yScale = computed(() => {
      return scaleLinear()
        .rangeRound([svgHeight.value, 0])
        .domain([
          (dataMin.value as number) > 0 ? 0 : (dataMin.value as number),
          dataMax.value as number,
        ]);
    });

    const svgHeight = computed(() => {
      return state.svgWidth / 1.61803398875;
    });

    const showTooltip = (e: MouseEvent) => {
      console.log(e);
      state.tooltip = true;
    };

    const hideTooltip = (e: MouseEvent) => {
      console.log(e);
      state.tooltip = false;
    };

    onMounted(() => {
      state.svgWidth = containerRef.value.offsetWidth * 0.75;
      chartResize();
      chartLoad();
    });
    return {
      ...toRefs(state),
      xScale,
      yScale,
      containerRef,
      svgHeight,
      showTooltip,
      hideTooltip,
    };
  },
});
</script>
<style lang=""></style>
