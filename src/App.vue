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
  type Plugin,
  type ChartType,
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

/**
 * 自訂 Chart.js Plugin - 在堆疊長條圖的最上方顯示加總數值
 *
 * 功能：
 * 1. 計算每個分類（X軸）的所有堆疊層級的加總
 * 2. 在每個堆疊長條的最上方顯示加總數值
 * 3. 處理隱藏的 dataset（不計入加總）
 */
const totalSumPlugin: Plugin<ChartType> = {
  id: "totalSum",
  afterDatasetDraw: (chart, _args, _options) => {
    const { ctx, data } = chart;

    // 檢查必要的資料是否存在
    if (!data.datasets || data.datasets.length === 0 || !data.labels) {
      return;
    }

    // 計算每個分類（X軸位置）的加總數值和最高點座標
    data.labels.forEach((_, categoryIndex) => {
      let sum = 0; // 該分類的加總
      let topY = Infinity; // 最高點的 Y 座標（像素）

      // 遍歷所有 datasets，計算加總並找出最高點
      data.datasets.forEach((dataset, datasetIndex) => {
        const meta = chart.getDatasetMeta(datasetIndex);

        // 檢查 dataset 是否被隱藏
        if (meta.hidden) {
          return; // 隱藏的 dataset 不參與計算
        }

        // 檢查該分類索引的資料是否存在
        if (!meta.data[categoryIndex]) {
          return;
        }

        const dataValue = dataset.data[categoryIndex] as number;
        const barElement = meta.data[categoryIndex];

        // 只計算非零數值
        if (dataValue && dataValue > 0) {
          sum += dataValue;

          // 找出最高點的 Y 座標（數值越小代表越靠近頂部）
          if (barElement.y < topY) {
            topY = barElement.y;
          }
        }
      });

      // 只有當加總大於 0 且找到有效的最高點時才顯示
      if (sum > 0 && topY !== Infinity) {
        // 取得該分類第一個 dataset 的 X 座標（所有 dataset 在同一分類的 X 座標相同）
        const firstVisibleDatasetIndex = data.datasets.findIndex(
          (_, index) => !chart.getDatasetMeta(index).hidden
        );

        if (firstVisibleDatasetIndex === -1) {
          return; // 沒有可見的 dataset
        }

        const xPos = chart.getDatasetMeta(firstVisibleDatasetIndex).data[
          categoryIndex
        ]?.x;

        if (xPos === undefined) {
          return; // 找不到有效的 X 座標
        }

        // 繪製加總數值
        ctx.save();
        ctx.font = "bold 14px sans-serif";
        ctx.fillStyle = "#333";
        ctx.textAlign = "center";
        ctx.textBaseline = "bottom";

        // 在最高點上方 8px 處顯示加總
        ctx.fillText(sum.toString(), xPos, topY - 8);

        ctx.restore();
      }
    });
  },
};

ChartJS.register(
  Title,
  Tooltip,
  Legend,
  BarElement,
  CategoryScale,
  LinearScale,
  ChartDataLabels,
  totalSumPlugin
);

type ExtendedPlugins = ChartOptions["plugins"] & {
  totalSum?: object;
};

interface ExtendedChartOptions extends ChartOptions<"bar"> {
  plugins?: ExtendedPlugins;
}

const chartOptions: ExtendedChartOptions = {
  responsive: true,
  scales: {
    x: { stacked: true },
    y: { stacked: true, grace: "5%" },
  },
  onClick: (_e, el, chart) => {
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
    totalSum: {
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
        // sum: {
        //   anchor: "end",
        //   align: "top",
        //   color: "red",
        //   font: {
        //     weight: "bold",
        //     size: 20,
        //   },
        //   clip: true,
        //   formatter: (v: number, ctx) => {
        //     const datasets = ctx.chart.data.datasets;
        //     const dataIndex = ctx.dataIndex;
        //     if (ctx.datasetIndex === datasets.length - 1) {
        //       const sum = datasets
        //         .map((ds) => ds.data[dataIndex] as number)
        //         .reduce((a, b) => a + b, 0);
        //       return sum;
        //     }
        //     return "";
        //   },
        // },
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
