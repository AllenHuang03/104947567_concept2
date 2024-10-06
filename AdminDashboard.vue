<template>
  <div class="admin-dashboard">
    <header>
      <h1>Admin Dashboard</h1>
      <nav>
        <button @click="activeView = 'stats'">Statistics</button>
        <button @click="activeView = 'upload'">Upload and Analyze</button>
        <button @click="activeView = 'users'">User Management</button>
        <button @click="logout">Logout</button>
      </nav>
    </header>

    <main>
      <section v-if="activeView === 'stats'">
        <!-- Existing statistics content -->
      </section>

      <section v-if="activeView === 'upload'">
        <FileUploadAnalysis />
      </section>

      <section v-if="activeView === 'users'">
        <!-- User management content -->
      </section>
    </main>
  </div>
</template>

<script>
import { ref } from "vue";
import { useRouter } from "vue-router";
import FileUploadAnalysis from "./FileUploadAnalysis.vue";

export default {
  name: "AdminDashboard",
  components: {
    FileUploadAnalysis,
  },
  setup() {
    const activeView = ref("stats");
    const router = useRouter();

    const logout = () => {
      localStorage.removeItem("token");
      localStorage.removeItem("isAdmin");
      router.push("/login");
    };

    return {
      activeView,
      logout,
    };
  },
};
</script>

<style scoped>
.admin-dashboard {
  font-family: Arial, sans-serif;
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 20px;
  margin-bottom: 20px;
}

.stat-card {
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

button {
  padding: 5px 10px;
  margin-right: 5px;
  background-color: #4caf50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:hover {
  background-color: #45a049;
}
</style>
