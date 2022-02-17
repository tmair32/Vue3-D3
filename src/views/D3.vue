<template>
  <div class="m-4">
    <!-- || Pie Chart || -->
    <pie-chart :chartData="objectData" xKey="name" yKey="amount" />

    <!-- || Control Bar || -->
    <div>
      <label for="itemCount">Record Count: </label>
      <input id="itemCount" v-model.number="itemCount" type="number" />
    </div>
    <div>
      <label for="min">Min Value: </label>
      <input id="min" v-model.number="min" type="number" min="0" />
    </div>
    <div>
      <label for="max">Max Value: </label>
      <input id="max" v-model.number="max" type="number" max="100" />
    </div>

    <!-- || Line Area Chart || -->
    <line-area-chart
      class="chart"
      :chart-data="data"
      :width="300"
      :height="500"
    />

    <!-- || Bar Chart || -->
    <bar-chart :chartData="objectData" xKey="name" yKey="amount" />
  </div>
</template>
<script lang="ts">
import { defineComponent, onMounted, reactive, toRefs } from "vue";
import { randomInt, interval, now } from "d3";
import LineAreaChart from "@/components/LineAreaChart.vue";
// import BarChart from "@/components/BarChart.vue";
import BarChart from "@/components/BarChartAxis.vue";
import PieChart from "@/components/PieChart.vue";

const REFRESH_RATE = 2000;

type State = {
  data: number[];
  itemCount: number;
  min: number;
  max: number;
  objectData: Array<{ name: string; amount: number }>;
};

export default defineComponent({
  name: "ChartView",
  components: {
    LineAreaChart,
    BarChart,
    PieChart,
  },
  setup() {
    const state = reactive<State>({
      data: [],
      itemCount: 25,
      min: 10,
      max: 100,
      objectData: [
        {
          name: "Roses",
          amount: 25,
        },
        {
          name: "Tulips",
          amount: 40,
        },
        {
          name: "Daisies",
          amount: 15,
        },
        {
          name: "Narcissuses",
          amount: 9,
        },
      ],
    });
    onMounted(() => {
      interval(
        () => {
          state.data = Array.from(
            { length: state.itemCount },
            randomInt(state.min, state.max)
          );
        },
        REFRESH_RATE,
        now() - REFRESH_RATE
      );
    });

    return { ...toRefs(state) };
  },
});
</script>
<style scoped>
div {
  text-align: center;
}
</style>
