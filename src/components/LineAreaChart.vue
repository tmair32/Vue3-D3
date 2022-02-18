<template>
  <div class="text-pink-400">
    <!-- <div>
      Scale X: {{ scaleX(0) }}
      <br />
      Scale Y: {{ scaleY(0) }}
    </div> -->
    <div class="flex justify-center">
      <svg ref="chartRef" :width="width" :height="height">
        <g>
          <path class="area" :d="areaPath" fill="#9d92b2" />
          <path
            class="line"
            :e="linePath"
            stroke="#6c1d5f"
            stroke-width="3px"
          />
        </g>
        <g>
          <g ref="axisBottomRef" :transform="axisBottomTransform" />
          <g ref="axisLeftRef" :transform="axisLeftTransform" />
        </g>
      </svg>
    </div>
  </div>
</template>
<script lang="ts">
import {
  defineComponent,
  computed,
  PropType,
  ref,
  onMounted,
  watchEffect,
  watch,
} from "vue";
import {
  scaleLinear,
  max,
  area,
  line,
  axisBottom,
  axisLeft,
  select,
  drag,
} from "d3";

const MARGIN = 25;

interface Ref<T> {
  value: T;
}
export default defineComponent({
  props: {
    chartData: {
      type: Array as PropType<number[]>,
      default: () => [],
    },
    width: {
      type: Number,
      required: true,
    },
    height: {
      type: Number,
      required: true,
    },
  },
  setup(props) {
    const scaleX = computed(() => {
      return scaleLinear()
        .domain([0, props.chartData.length - 1])
        .range([MARGIN, props.width - MARGIN]);
    });
    const scaleY = computed(() => {
      return scaleLinear()
        .domain([0, max(props.chartData) ?? 0])
        .range([props.height - MARGIN, MARGIN]);
    });

    const areaPathGenerator = computed(() => {
      return area<number>()
        .x((_, i) => scaleX.value(i))
        .y0(scaleY.value(0))
        .y1((d) => scaleY.value(d));
    });

    const linePathGenerator = computed(() => {
      return line<number>()
        .x((_, i) => scaleX.value(i))
        .y((d) => scaleY.value(d));
    });

    const areaPath = computed(
      () => areaPathGenerator.value(props.chartData) ?? ""
    );
    const linePath = computed(
      () => linePathGenerator.value(props.chartData) ?? ""
    );

    const axisBottomRef: Ref<SVGGElement | undefined> = ref();
    const axisLeftRef: Ref<SVGGElement | undefined> = ref();

    const axisBottomTransform = computed(
      () => `translate(0, ${scaleY.value(0)})`
    );
    const axisLeftTransform = computed(
      () => `translate(${scaleX.value(0)}, 0)`
    );

    const svgRef: Ref<SVGSVGElement | undefined> = ref();

    onMounted(() => {
      if (!svgRef.value) return;
      select(svgRef.value).call(
        drag<SVGSVGElement, unknown>()
          .on("start", function dragStart() {
            console.log("a");
            select(this).raise().classed("active", true);
          })
          .on("drag", function dragging(e) {
            const xCoor = e.x;
            const yCoor = e.y;

            select(this).attr("x", xCoor).attr("y", yCoor);
          })
          .on("end", function dragEnd() {
            select(this).classed("active", false);
          })
      );

      watch(
        scaleX,
        () => {
          if (!axisBottomRef.value) return;
          select(axisBottomRef.value).call(axisBottom(scaleX.value));
        },
        { immediate: true, deep: true }
      );
      watchEffect(() => {
        if (!axisLeftRef.value) return;
        select(axisLeftRef.value).call(axisLeft(scaleY.value));
      });
    });

    return {
      scaleX,
      scaleY,
      areaPath,
      linePath,
      axisBottomRef,
      axisLeftRef,
      axisBottomTransform,
      axisLeftTransform,
    };
  },
});
</script>
