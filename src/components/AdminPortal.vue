<template>
    <div class="admin-portal">
      <h1>Users</h1>
      <!-- Create New User Button -->
      <button class="btn btn-success" @click="openCreateModal">Create New User</button>
  
      <!-- Users Table -->
      <table class="table table-hover users-table">
        <thead>
          <tr>
            <th>Username</th>
            <th>Password</th>
            <th>Email</th>
            <th>Date of Birth</th>
            <th>State</th>
            <th>Role</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="user in users" :key="user.id">
            <td>{{ user.username }}</td>
            <td>{{ user.password }}</td>
            <td>{{ user.email }}</td>
            <td>{{ user.date_of_birth }}</td>
            <td>{{ user.state }}</td>
            <td>{{ user.role }}</td>
            <td>
              <!-- Update Button -->
              <button class="btn btn-primary btn-sm" @click="openUpdateModal(user)">Update</button>
              <!-- Delete Button -->
              <button class="btn btn-danger btn-sm" @click="deleteUser(user.id)">Delete</button>
            </td>
          </tr>
        </tbody>
      </table>
  
      <!-- Create/Edit User Modal -->
      <b-modal ref="userModal" :title="isEditing ? 'Edit User' : 'Create New User'" hide-footer>
        <b-form @submit.prevent="isEditing ? updateUser() : createUser()">
          <b-form-group label="Username" label-for="username-input">
            <b-form-input id="username-input" v-model="userForm.username" required></b-form-input>
          </b-form-group>
          <b-form-group label="Password" label-for="password-input">
            <b-form-input id="password-input" type="password" v-model="userForm.password" required></b-form-input>
          </b-form-group>
          <b-form-group label="Email" label-for="email-input">
            <b-form-input id="email-input" type="email" v-model="userForm.email" required></b-form-input>
          </b-form-group>
          <b-form-group label="Date of Birth" label-for="dob-input">
            <b-form-input id="dob-input" type="date" v-model="userForm.date_of_birth" required></b-form-input>
          </b-form-group>
          <b-form-group label="State" label-for="state-input">
            <b-form-input id="state-input" v-model="userForm.state" required></b-form-input>
          </b-form-group>
          <b-form-group label="Role" label-for="role-input">
            <b-form-input id="role-input" v-model="userForm.role" required></b-form-input>
          </b-form-group>
          <b-button type="submit" variant="success">{{ isEditing ? 'Update User' : 'Create User' }}</b-button>
        </b-form>
      </b-modal>
    </div>
</template>


<script>
import axios from 'axios';
export default {
  name: "AdminPortal",
  data() {
    return {
      users: [],
      userForm: {
        id: null,
        username: '',
        password: '',
        email: '',
        date_of_birth: '',
        state: '',
        role: ''
      },
      isEditing: false
    };
  },
  methods: {
    // Fetch all users from the database
    fetchUsers() {
      axios.get(`${process.env.VUE_APP_ROOT_URL}/users`)
        .then(response => {
          this.users = response.data.users;
        })
        .catch(error => {
          console.error("Error fetching users:", error);
        });
    },
    // Open modal to create a new user
    openCreateModal() {
      this.isEditing = false;
      this.resetUserForm();
      this.$refs.userModal.show();
    },
    // Open modal to update an existing user
    openUpdateModal(user) {
      this.isEditing = true;
      this.userForm = { ...user }; // Load user data into form
      this.$refs.userModal.show();
    },
    // Reset user form fields
    resetUserForm() {
      this.userForm = {
        id: null,
        username: '',
        password: '',
        email: '',
        date_of_birth: '',
        state: '',
        role: ''
      };
    },
    // Create a new user
    createUser() {
      axios.post(`${process.env.VUE_APP_ROOT_URL}/users`, this.userForm)
        .then(() => {
          this.fetchUsers(); // Refresh user list
          this.$refs.userModal.hide();
        })
        .catch(error => {
          console.error("Error creating user:", error);
        });
    },
    // Update an existing user
    updateUser() {
      axios.put(`${process.env.VUE_APP_ROOT_URL}/users/${this.userForm.id}`, this.userForm)
        .then(() => {
          this.fetchUsers(); // Refresh user list
          this.$refs.userModal.hide();
        })
        .catch(error => {
          console.error("Error updating user:", error);
        });
    },
    // Delete a user by ID
    deleteUser(userId) {
      if (confirm("Are you sure you want to delete this user?")) {
        axios.delete(`${process.env.VUE_APP_ROOT_URL}/users/${userId}`)
          .then(() => {
            this.fetchUsers(); // Refresh user list
          })
          .catch(error => {
            console.error("Error deleting user:", error);
          });
      }
    }
  },
  created() {
    this.fetchUsers(); // Load users on component mount
  }
};
</script>

<style scoped>
    .admin-portal {
        padding: 20px;
    }
    
    .users-table {
        width: 100%;
        margin-top: 20px;
    }
    
    button {
        margin: 5px;
    }
</style>