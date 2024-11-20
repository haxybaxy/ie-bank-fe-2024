<template>
  <div class="register-page">
    <h2>Create Your Account</h2>
    <p>Join the best bank in the world</p>
    <form @submit.prevent="handleRegister" class="register-form">
      <label for="username">Username</label>
      <input type="text" id="username" v-model="username" required />

      <label for="email">Email</label>
      <input type="email" id="email" v-model="email" required />

      <label for="password">Password</label>
      <input type="password" id="password" v-model="password" required />

      <label for="confirmPassword">Confirm Password</label>
      <input type="password" id="confirmPassword" v-model="confirmPassword" required />

      <label for="country">Country</label>
      <input type="text" id="country" v-model="country" required />
      
      <label for="dob">Date of Birth</label>
      <input type="date" id="dob" v-model="date_of_birth" required />

      <button type="submit" class="submit-button">Register</button>

      <p class="login-link">
        Already have an account?
        <router-link to="/login" class="link">Log in here</router-link>
      </p>
    </form>
  </div>
</template>

<script>
import axios from 'axios';

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
    handleRegister() {
      // Check if passwords match
      if (this.password !== this.confirmPassword) {
        alert("Passwords do not match!");
        return;
      }

      // Call RESTregister to send the data to the backend
      this.RESTregister();
    },
    RESTregister() {
      const path = `${process.env.VUE_APP_ROOT_URL}/register`; // Backend API URL

      // Prepare the payload with form data
      const payload = {
        username: this.username, // User's name
        email: this.email, // User's email
        password: this.password, // User's password
        country: this.country, // User's country
        date_of_birth: this.date_of_birth, // User's date of birth
      };
      console.log(payload)

      axios
        .post(path, payload) // Send POST request with the payload
        .then((response) => {
          console.log("Registration successful:", response.data);
          alert(`Welcome, ${response.data.username}! Registration successful.`);
          this.$router.push("/login"); // Redirect to the login page
        })
        .catch((error) => {
          console.error("Registration failed:", error.response?.data || error.message);
          alert("Registration failed. Please check your details and try again.");
        });
    },
  },
};
</script>

<style scoped>
.register-page {
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
color: transparent; /* Fallback for non-webkit browsers */
}

p {
color: #e0e0e0;
margin-bottom: 20px;
}

.register-form {
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

.login-link {
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

