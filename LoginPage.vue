<template>
  <div class="login-container">
    <form @submit.prevent="login" class="login-form">
      <h2>Login to Malware Analysis Tool</h2>
      <div class="form-group">
        <label for="username">Username:</label>
        <input type="text" id="username" v-model="username" required />
      </div>
      <div class="form-group">
        <label for="password">Password:</label>
        <input type="password" id="password" v-model="password" required />
      </div>
      <button type="submit">Login</button>
      <p v-if="error" class="error-message">{{ error }}</p>
    </form>
  </div>
</template>

<script>
import { ref } from "vue";
import { useRouter } from "vue-router";
import api from "@/api/config"; // Import the configured api

export default {
  name: "LoginPage",
  setup() {
    const username = ref("");
    const password = ref("");
    const error = ref("");
    const router = useRouter();

    const login = async () => {
      try {
        console.log("Attempting login with:", {
          username: username.value,
          password: password.value,
        });
        const response = await api.post("/api/login", {
          username: username.value,
          password: password.value,
        });
        console.log("Login response:", response.data);
        if (response.data.success) {
          localStorage.setItem("token", response.data.token);
          localStorage.setItem("isAdmin", response.data.isAdmin);
          if (response.data.isAdmin) {
            router.push("/admin");
          } else {
            router.push("/dashboard");
          }
        } else {
          error.value =
            "Login failed: " + (response.data.msg || "Unknown error");
        }
      } catch (err) {
        console.error("Login error:", err);
        error.value =
          "An error occurred. Please try again. Details: " +
          (err.response?.data?.msg || err.message);
      }
    };

    return {
      username,
      password,
      error,
      login,
    };
  },
};
</script>

<style scoped>
.login-container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  background-color: #f0f0f0;
}

.login-form {
  background-color: white;
  padding: 2rem;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  width: 300px;
}

.form-group {
  margin-bottom: 1rem;
}

label {
  display: block;
  margin-bottom: 0.5rem;
}

input {
  width: 100%;
  padding: 0.5rem;
  border: 1px solid #ccc;
  border-radius: 4px;
}

button {
  width: 100%;
  padding: 0.5rem;
  background-color: #4caf50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.error-message {
  color: red;
  margin-top: 1rem;
}
</style>
