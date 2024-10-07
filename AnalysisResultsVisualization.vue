<template>
  <div class="analysis-results-visualization">
    <h2 class="main-title">Analysis Results</h2>
    <div v-if="chartData" class="chart-container">
      <h3>{{ chartTitle }}</h3>
      <component
        :is="chartComponent"
        :data="chartData"
        :options="chartOptions"
      />
    </div>
    <div v-else class="no-data">
      No valid data available for visualization.
      <pre>Debug info: {{ debugInfo }}</pre>
    </div>
  </div>
</template>

<script>
import { defineComponent, computed, ref, watchEffect } from "vue";
import { Pie, Bar } from "vue-chartjs";
import {
  Chart as ChartJS,
  Title,
  Tooltip,
  Legend,
  ArcElement,
  CategoryScale,
  LinearScale,
  BarElement,
} from "chart.js";

ChartJS.register(
  Title,
  Tooltip,
  Legend,
  ArcElement,
  CategoryScale,
  LinearScale,
  BarElement
);

export default defineComponent({
  name: "AnalysisResultsVisualization",
  components: { Pie, Bar },
  props: {
    results: {
      type: Object,
      required: true,
    },
    type: {
      type: String,
      required: true,
      validator: (value) => ["binary", "multiclass"].includes(value),
    },
  },
  setup(props) {
    const debugInfo = ref("");
    const chartData = ref(null);

    const chartTitle = computed(() => {
      return props.type === "binary"
        ? "Binary Classification"
        : "Multi-class Classification";
    });

    const chartComponent = computed(() => {
      return props.type === "binary" ? Pie : Bar;
    });

    const chartOptions = computed(() => ({
      responsive: true,
      maintainAspectRatio: false,
      plugins: {
        legend: {
          display: true,
          position: "top",
        },
        title: {
          display: true,
          text: chartTitle.value,
          font: {
            size: 16,
          },
        },
      },
      scales:
        props.type === "multiclass"
          ? {
              y: {
                beginAtZero: true,
                max:
                  Math.max(
                    ...(props.results.data || []).filter((v) => v != null),
                    0
                  ) * 1.1,
                title: {
                  display: true,
                  text: "Count",
                },
              },
              x: {
                title: {
                  display: true,
                  text: "Class",
                },
              },
            }
          : undefined,
    }));

    watchEffect(() => {
      if (!props.results || !props.results.labels || !props.results.data) {
        debugInfo.value = `Invalid data structure: ${JSON.stringify(
          props.results
        )}`;
        chartData.value = null;
        return;
      }

      // Filter out undefined or null values
      const validData = props.results.data
        .map((value, index) => ({
          label: props.results.labels[index],
          value: value,
        }))
        .filter((item) => item.value != null && item.label !== "undefined");

      chartData.value = {
        labels: validData.map((item) => item.label),
        datasets: [
          {
            data: validData.map((item) => item.value),
            backgroundColor: [
              "#36A2EB",
              "#FF6384",
              "#FFCE56",
              "#4BC0C0",
              "#9966FF",
              "#FF9F40",
              "#41B883",
              "#E46651",
              "#00D8FF",
              "#DD1B16",
            ],
          },
        ],
      };
    });

    return {
      chartTitle,
      chartComponent,
      chartData,
      chartOptions,
      debugInfo,
    };
  },
});
</script>

<style scoped>
.analysis-results-visualization {
  width: 100%;
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}
.main-title {
  text-align: center;
  margin-bottom: 20px;
}
.chart-container {
  height: 400px; /* Fixed height */
  position: relative;
  margin-bottom: 40px; /* Add space between charts */
}
.no-data {
  text-align: center;
  color: #666;
  font-style: italic;
}
</style>
