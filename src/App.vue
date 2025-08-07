<script setup lang="ts">
import { Bar } from "vue-chartjs";
import {
  Chart as ChartJS,
  Title,
  Tooltip,
  Legend,
  BarElement,
  CategoryScale,
  LinearScale,
  type ChartOptions,
  type ChartData,
} from "chart.js";
import ChartDataLabels from "chartjs-plugin-datalabels";

const chartData: ChartData<"bar"> = {
  labels: ["8月5日", "8月6日", "8月7日", "8月8日"],
  datasets: [
    {
      label: "mike",
      data: [5, 0, 20, 0],
      backgroundColor: "#42A5F5",
    },
    {
      label: "anne",
      data: [3, 10, 0, 4],
      backgroundColor: "#66BB6A",
    },
    {
      label: "bill",
      data: [0, 15, 3, 0],
      backgroundColor: "#FFA726",
    },
  ],
};

ChartJS.register(
  Title,
  Tooltip,
  Legend,
  BarElement,
  CategoryScale,
  LinearScale,
  ChartDataLabels
);

const chartOptions: ChartOptions<"bar"> = {
  responsive: true,
  scales: {
    x: { stacked: true },
    y: { stacked: true, grace: "5%" },
  },
  onClick: (e, el, chart) => {
    // console.log("event :>> ", e);
    console.log("el :>> ", el);
    console.log("chart :>> ", chart);

    if (el.length > 0) {
      const datasetIndex = el[0].datasetIndex;
      const dataIndex = el[0].index;
      const value = chart.data.datasets[datasetIndex].data[dataIndex];
      console.log(
        `Clicked on dataset ${datasetIndex}, data index ${dataIndex}, value: ${value}`
      );
      console.log("label :>> ", chart.data.labels?.[dataIndex]);
      console.log("value :>> ", value);
    }
  },
  plugins: {
    legend: {
      position: "right",
    },
    tooltip: {
      enabled: false,
    },
    datalabels: {
      labels: {
        value: {
          color: "#000",
          font: {
            weight: "bold",
            size: 16,
          },
          formatter: (value: number) => {
            return !!value ? `${value}` : "";
          },
        },
        sum: {
          anchor: "end",
          align: "top",
          color: "red",
          font: {
            weight: "bold",
            size: 20,
          },
          clip: true,
          formatter: (v: number, ctx) => {
            const datasets = ctx.chart.data.datasets;
            const dataIndex = ctx.dataIndex;
            if (ctx.datasetIndex === datasets.length - 1) {
              const sum = datasets
                .map((ds) => ds.data[dataIndex] as number)
                .reduce((a, b) => a + b, 0);
              return sum;
            }
            return "";
          },
        },
      },
    },
  },
};
</script>

<template>
  <div class="chart-container">
    <Bar id="my-chart-id" :options="chartOptions" :data="chartData" />
  </div>
</template>

<style scoped>
.chart-container {
  margin-top: 20px;
  position: relative;
  width: 100%;
  height: 400px;
}
</style>
