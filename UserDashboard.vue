<template>
  <div class="user-dashboard">
    <header>
      <h1>Malware Analysis Tool</h1>
      <nav>
        <button @click="logout">Logout</button>
      </nav>
    </header>

    <main>
      <section class="upload-section">
        <h2>Upload File for Analysis</h2>
        <p>CSV upload only, size less than 5MB</p>
        <input type="file" @change="handleFileUpload" accept=".csv" />
        <button @click="handleDownloadTemplate">Download Template</button>
        <div class="analysis-type">
          <button
            :class="{ active: analysisType === 'classification' }"
            @click="analysisType = 'classification'"
          >
            Classification
          </button>
          <button
            :class="{ active: analysisType === 'prediction' }"
            @click="analysisType = 'prediction'"
          >
            Prediction
          </button>
        </div>
        <button @click="handleAnalyzeFile" :disabled="!selectedFile">
          Analyze
        </button>
      </section>

      <section v-if="results" class="results-section">
        <h2>Analysis Results</h2>
        <div v-if="analysisType === 'classification'">
          <AnalysisResultsVisualization
            v-if="results.binary_classification"
            :results="results.binary_classification"
            type="binary"
          />
          <AnalysisResultsVisualization
            v-if="results.multi_class_classification"
            :results="results.multi_class_classification"
            type="multiclass"
          />
        </div>
        <div v-else-if="analysisType === 'prediction'">
          <p>Prediction functionality is not available at this time.</p>
        </div>
      </section>
    </main>
  </div>
</template>

<script>
import { ref, onMounted, onUnmounted } from "vue";
import { useRouter } from "vue-router";
import { analyzeFile, downloadTemplate } from "@/api/config";
import AnalysisResultsVisualization from "./AnalysisResultsVisualization.vue";

export default {
  name: "UserDashboard",
  components: {
    AnalysisResultsVisualization,
  },
  setup() {
    const router = useRouter();
    const selectedFile = ref(null);
    const analysisType = ref("classification");
    const results = ref(null);
    let analyzeTimeout = null;

    const logout = () => {
      localStorage.removeItem("token");
      router.push("/login");
    };

    const handleFileUpload = (event) => {
      const file = event.target.files[0];
      if (file && file.size <= 5 * 1024 * 1024) {
        // 5MB limit
        selectedFile.value = file;
      } else {
        alert("Please select a CSV file smaller than 5MB.");
        event.target.value = ""; // Clear the file input
      }
    };

    const handleDownloadTemplate = async () => {
      try {
        const response = await downloadTemplate();
        const url = window.URL.createObjectURL(new Blob([response.data]));
        const link = document.createElement("a");
        link.href = url;
        link.setAttribute("download", "analysis_template.csv");
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
      } catch (error) {
        console.error("Error downloading template:", error);
        alert(
          "An error occurred while downloading the template. Please try again."
        );
      }
    };

    const handleAnalyzeFile = async () => {
      if (!selectedFile.value) return;

      const formData = new FormData();
      formData.append("file", selectedFile.value);
      formData.append("analysisType", analysisType.value);

      try {
        console.log("Sending file for analysis:", selectedFile.value.name);
        const response = await analyzeFile(formData);
        console.log("Full server response:", response);
        results.value = response.data;
        console.log("Processed results:", results.value);
      } catch (error) {
        console.error("Error analyzing file:", error);
        alert("An error occurred while analyzing the file. Please try again.");
      }
    };

    onMounted(() => {
      // Any setup code if needed
    });

    onUnmounted(() => {
      if (analyzeTimeout) {
        clearTimeout(analyzeTimeout);
      }
    });

    return {
      logout,
      selectedFile,
      analysisType,
      results,
      handleFileUpload,
      handleDownloadTemplate,
      handleAnalyzeFile,
    };
  },
};
</script>

<style scoped>
.upload-section {
  background-color: #f0f0f0;
  padding: 20px;
  border-radius: 8px;
  margin-bottom: 20px;
}

.analysis-type {
  margin: 10px 0;
}

.analysis-type button {
  margin-right: 10px;
}

.analysis-type button.active {
  background-color: #45a049;
}

.results-section {
  background-color: #f0f0f0;
  padding: 20px;
  border-radius: 8px;
}
</style>
