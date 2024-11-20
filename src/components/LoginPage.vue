<template>
  <div class="login-page">
    <h2>Welcome Back to IE Bank</h2>
    <p>Please log in to continue</p>
    <form @submit.prevent="login" class="login-form">
      <label for="username">Username</label>
      <input type="text" id="username" v-model="username" required />

      <label for="password">Password</label>
      <input type="password" id="password" v-model="password" required />

      <button type="submit" class="submit-button">Login</button>

      <p class="register-link">
        Don't have an account? 
        <router-link to="/register" class="link">Register here</router-link>
      </p>
    </form>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      id: '',
      username: '', // User's username
      email: '', // User's email
      country: '', // User's password
      state: '',
      date_of_birth: '', // User's password confirmation
      role: '', // User's country
      status: '', // User's date of birth
    };
  },
  methods: {
    // Login API method
    RESTlogin(username, password) {
      const path = `${process.env.VUE_APP_ROOT_URL}/login`; // Backend API URL
      const payload = { username, password };

      axios
        console.log(path)
        .post(path, payload)
        .then((response) => {
          // Handle successful login
          console.log("Login successful:", response.data);
          alert(`Welcome back, ${response.data.name || username}!`);

          // Store user info or token if required
          localStorage.setItem("authToken", response.data.token || ""); // Example: Store token
          console.log(response)

          // Redirect based on role (optional)
          if (response.data.role === "admin") {
            this.$router.push("/admin");
          } else {
            this.$router.push("/user");
          }
        })
        .catch((error) => {
          // Handle login error
          console.error("Login failed:", error.response?.data || error.message);
          alert("Login failed. Please check your credentials.");
        });
    },

    // Login handler
    login() {
      if (!this.username || !this.password) {
        alert("Please fill in both fields.");
        return;
      }
      this.RESTlogin(this.username, this.password);
    },
  },
};
</script>

<style scoped>
.login-page {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 100vh;
  background: linear-gradient(to bottom right, #0648d7, #2ea901);
  color: #fff;
}

h2 {
  font-size: 48px;
  background-color: #ffffff;
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  color: transparent; 
}

p {
  color: #ffffff;
  margin-bottom: 20px;
}

.login-form {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 300px;
}

label {
  align-self: flex-start;
  margin-bottom: 5px;
  color: #f0f0f0;
}

input {
  width: 100%;
  padding: 10px;
  margin-bottom: 15px;
  border-radius: 5px;
  border: none;
}

.submit-button {
  width: 100%;
  padding: 10px;
  background-color: #2ea901;
  color: white;
  font-size: 16px;
  font-weight: bold;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.submit-button:hover {
  background-color: #0648d7;
}

.register-link {
  margin-top: 20px;
  color: #f0f0f0;
}

.link {
  color: #2ea901;
  text-decoration: underline;
  font-weight: bold;
}

.link:hover {
  color: #0648d7;
}
</style>
