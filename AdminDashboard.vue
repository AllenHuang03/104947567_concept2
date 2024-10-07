<template>
  <div class="admin-dashboard">
    <header>
      <h1>Admin Dashboard</h1>
      <nav>
        <button @click="activeView = 'overview'">Overview</button>
        <button @click="activeView = 'scanResults'">Scan Results</button>
        <button @click="activeView = 'modelMetrics'">Model Metrics</button>
        <button @click="logout">Logout</button>
      </nav>
    </header>

    <main>
      <section v-if="activeView === 'overview'" class="overview">
        <h2>Overview</h2>
        <div class="stats-grid">
          <div class="stat-card">
            <h3>Total Scans</h3>
            <p>{{ dashboardData.totalScans }}</p>
          </div>
          <div class="stat-card">
            <h3>Malware Detected</h3>
            <p>{{ dashboardData.totalMalwareDetected }}</p>
          </div>
          <div class="stat-card">
            <h3>Overall Model Accuracy</h3>
            <p>{{ dashboardData.overallModelAccuracy.toFixed(2) }}%</p>
          </div>
        </div>
      </section>

      <section v-if="activeView === 'scanResults'" class="scan-results">
        <h2>Recent Scan Results</h2>
        <table>
          <thead>
            <tr>
              <th>File Name</th>
              <th>Analysis Time</th>
              <th>Result</th>
            </tr>
          </thead>
          <tbody>
            <tr
              v-for="result in dashboardData.recentScanResults"
              :key="result.fileName"
            >
              <td>{{ result.fileName }}</td>
              <td>{{ new Date(result.analysisTime).toLocaleString() }}</td>
              <td>{{ result.result }}</td>
            </tr>
          </tbody>
        </table>
      </section>

      <section v-if="activeView === 'modelMetrics'" class="model-metrics">
        <h2>Model Metrics</h2>
        <div class="metrics-grid">
          <div class="metric-card">
            <h3>Binary Classification Accuracy</h3>
            <p>{{ dashboardData.binaryClassificationAccuracy.toFixed(2) }}%</p>
          </div>
          <div class="metric-card">
            <h3>Multi-Class Classification Accuracy</h3>
            <p>
              {{ dashboardData.multiClassClassificationAccuracy.toFixed(2) }}%
            </p>
          </div>
        </div>
      </section>
    </main>
  </div>
</template>

<script>
import { ref, onMounted } from "vue";
import { useRouter } from "vue-router";
import { getAdminDashboard } from "@/api/config";

export default {
  name: "AdminDashboard",
  setup() {
    const activeView = ref("overview");
    const router = useRouter();
    const dashboardData = ref({
      totalScans: 0,
      totalMalwareDetected: 0,
      overallModelAccuracy: 0,
      recentScanResults: [],
      binaryClassificationAccuracy: 0,
      multiClassClassificationAccuracy: 0,
    });

    const logout = () => {
      localStorage.removeItem("token");
      localStorage.removeItem("isAdmin");
      router.push("/login");
    };

    const fetchDashboardData = async () => {
      try {
        const response = await getAdminDashboard();
        dashboardData.value = response.data;
      } catch (error) {
        console.error("Error fetching dashboard data:", error);
        // Handle error (e.g., show error message to user)
      }
    };

    onMounted(fetchDashboardData);

    return {
      activeView,
      logout,
      dashboardData,
    };
  },
};
</script>

<style scoped>
/* ... (keep existing styles) ... */

.stats-grid,
.metrics-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 20px;
  margin-bottom: 20px;
}

.stat-card,
.metric-card {
  background-color: #f0f0f0;
  padding: 15px;
  border-radius: 8px;
  text-align: center;
}

table {
  width: 100%;
  border-collapse: collapse;
  margin-bottom: 20px;
}

th,
td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: left;
}

th {
  background-color: #f2f2f2;
}
</style>
