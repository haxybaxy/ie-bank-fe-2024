<template>
  <div class="user-portal">
    <!-- Header Section -->
    <div class="header">
      <h1 class="user-title">User Portal</h1>
      <button class="logout-button" @click="logout">Logout</button>
    </div>

    <!-- Accounts Section -->
    <h1 class="section-title">My Accounts</h1>
    <button class="btn-create" v-b-modal.account-modal>Create Account</button>

    <b-alert v-if="showMessage" :variant="alertVariant" show>
      {{ message }}
    </b-alert>

    <table class="table table-hover account-table">
      <thead>
        <tr>
          <th>Account Name</th>
          <th>Account Number</th>
          <th>Balance</th>
          <th>Currency</th>
          <th>Country</th>
          <th>Status</th>
          <th>Actions</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="account in accounts" :key="account.id">
          <td>{{ account.name }}</td>
          <td>{{ account.account_number }}</td>
          <td>{{ account.balance }}</td>
          <td>{{ account.currency }}</td>
          <td>{{ account.country }}</td>
          <td>
            <span v-if="account.status === 'Active'" class="badge badge-success">{{ account.status }}</span>
            <span v-else class="badge badge-danger">{{ account.status }}</span>
          </td>
          <td>
            <div class="btn-group">
              <button class="btn btn-info btn-sm" @click="openUpdateAccountModal(account)">Edit</button>
              <button class="btn btn-danger btn-sm" @click="deleteAccount(account.id)">Delete</button>
            </div>
          </td>
        </tr>
      </tbody>
    </table>

    <!-- Transactions Section -->
    <h1 class="section-title">Recent Transactions</h1>
    <button class="btn-create" v-b-modal.transaction-modal>Create Transaction</button>

    <table class="table table-hover transactions-table">
      <thead>
        <tr>
          <th>Date</th>
          <th>Amount</th>
          <th>Currency</th>
          <th>From Account</th>
          <th>To Account</th>
          <th>Status</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="transaction in transactions" :key="transaction.id">
          <td>{{ transaction.date }}</td>
          <td>{{ transaction.amount }}</td>
          <td>{{ transaction.currency }}</td>
          <td>{{ transaction.from_account }}</td>
          <td>{{ transaction.to_account }}</td>
          <td>{{ transaction.transaction_status }}</td>
        </tr>
      </tbody>
    </table>

    <!-- Create Account Modal -->
    <b-modal ref="accountModal" id="account-modal" title="Create Account" hide-footer>
      <b-form @submit.prevent="onSubmitAccount" class="modal-form">
        <b-form-group label="Account Name">
          <b-form-input v-model="createAccountForm.name" placeholder="Enter account name" required></b-form-input>
        </b-form-group>
        <b-form-group label="Balance">
          <b-form-input type="number" v-model="createAccountForm.balance" placeholder="Enter positive balance" required></b-form-input>
        </b-form-group>
        <b-form-group label="Currency">
          <b-form-input v-model="createAccountForm.currency" placeholder="Enter currency (€ or $)" required></b-form-input>
        </b-form-group>
        <b-form-group label="Country">
          <b-form-input v-model="createAccountForm.country" placeholder="Enter country" required></b-form-input>
        </b-form-group>
        <b-button type="submit" class="modal-submit-button" variant="success">Create Account</b-button>
      </b-form>
    </b-modal>

    <!-- Create Transaction Modal -->
    <b-modal ref="transactionModal" id="transaction-modal" title="Create Transaction" hide-footer>
      <b-form @submit.prevent="onSubmitTransaction" class="modal-form">
        <b-form-group label="Amount">
          <b-form-input type="number" v-model="createTransactionForm.amount" placeholder="Enter transaction amount" required></b-form-input>
        </b-form-group>
        <b-form-group label="Currency">
          <b-form-input v-model="createTransactionForm.currency" placeholder="Enter currency (e.g., USD)" required></b-form-input>
        </b-form-group>
        <b-form-group label="From Account">
          <b-form-select v-model="createTransactionForm.from_account" :options="accountOptions" placeholder="Select source account" required></b-form-select>
        </b-form-group>
        <b-form-group label="To Account">
          <b-form-input v-model="createTransactionForm.to_account" placeholder="Enter recipient account" required></b-form-input>
        </b-form-group>
        <b-button type="submit" class="modal-submit-button" variant="success">Create Transaction</b-button>
      </b-form>
    </b-modal>

    <!-- Edit Account Modal -->
    <b-modal ref="editAccountModal" id="edit-account-modal" title="Edit Account" hide-footer>
      <b-form @submit.prevent="onSubmitUpdate" class="modal-form">
        <b-form-group label="Account Name">
          <b-form-input v-model="editAccountForm.name" placeholder="Enter new account name" required></b-form-input>
        </b-form-group>
        <b-button type="submit" class="modal-submit-button" variant="info">Update Account</b-button>
      </b-form>
    </b-modal>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
  return {
    accounts: [],
    transactions: [],
    createTransactionForm: {
        amount: '',
        currency: "",
        from_account: "",
        to_account: "",
    },
    createAccountForm: {
        name: '',
        balance: '',
        currency: '',
        country: '',
    },
    editAccountForm: {
      id: "",
      name: "",
    },
    showMessage: false,
    message: "",
  };
  },
  computed: {
  accountOptions() {
    return this.accounts.map(account => ({
      value: account.account_number,
      text: `${account.name} (${account.account_number})`,
      }));
    },
  },

  methods: {
    logout() {
      localStorage.removeItem("authToken");
      this.$router.push("/login");
    },

    // Ensure accounts are loaded before opening the transaction modal
    openTransactionModal() {
      if (!this.accounts || this.accounts.length === 0) {
        this.message = "Please wait, accounts are still loading.";
        this.showMessage = true;
        setTimeout(() => (this.showMessage = false), 3000);
        return;
      }
      this.$refs.transactionModal.show();
    },
    
    //GET functions
    RESTgetTransactions() {
      const path = `${process.env.VUE_APP_ROOT_URL}/transactions`;
      axios
        .get(path)
        .then((response) => {
          this.transactions = response.data.transactions;
        })
        .catch((error) => {
          console.error(error);
        });
    },

    RESTgetAccounts() {
      const path = `${process.env.VUE_APP_ROOT_URL}/accounts`;
      axios
        .get(path)
        .then((response) => {
          this.accounts = response.data.accounts;
        })
        .catch((error) => {
          console.error(error);
        });
    },

    // POST functions
    RESTcreateAccount(payload) {
      const path = `${process.env.VUE_APP_ROOT_URL}/accounts`;
      axios
        .post(path, payload)
        .then((response) => {
          this.RESTgetAccounts();
          this.message = "Account Created successfully!";
          this.showMessage = true;
          setTimeout(() => {
            this.showMessage = false;
          }, 3000);
        })
        .catch((error) => {
          console.error(error);
          this.RESTgetAccounts();
        });
    },  

    RESTcreateTransaction(payload) {
      const path = `${process.env.VUE_APP_ROOT_URL}/transactions`;
      axios
        .post(path, payload)
        .then((response) => {
          this.RESTgetTransactions();
          this.message = "Transaction created successfully!";
          this.showMessage = true;
          setTimeout(() => {
            this.showMessage = false;
          }, 3000);
        })
        .catch((error) => {
          console.error(error);
          this.RESTgetTransactions();
        });
    },

    // PUT functions
    RESTupdateAccount(payload, accountId) {
      const path = `${process.env.VUE_APP_ROOT_URL}/accounts/${accountId}`;
      axios
        .put(path, payload)
        .then(() => {
          this.RESTgetAccounts();
          this.message = "Account Updated successfully!";
          this.showMessage = true;
          setTimeout(() => {
            this.showMessage = false;
          }, 3000);
        })
        .catch((error) => {
          console.error(error);
          this.RESTgetAccounts();
        });
    },

    // DELETE function
    RESTdeleteAccount(accountId) {
      const path = `${process.env.VUE_APP_ROOT_URL}/accounts/${accountId}`;
      axios
        .delete(path)
        .then((response) => {
          this.RESTgetAccounts();
          this.message = "Account Deleted successfully!";
          this.showMessage = true;
          setTimeout(() => {
            this.showMessage = false;
          }, 3000);
        })
        .catch((error) => {
          console.error(error);
          this.RESTgetAccounts();
        });
      }
    },    


  /***************************************************
    * FORM MANAGEMENT
   **************************************************/

    // Initialize forms empty
    initCreateAccountForm() {
      this.createAccountForm ={
        name: "",
        balance: 0,
        currency: "",
        country: "",
      };
    },

    // Initialize transaction form
    initTransactionForm() {
      this.createTransactionForm = {
        amount: 0,
        currency: "",
        from_account: "",
        to_account: "",
      };
    },

    initEditForm(){
      this.editAccountForm = {
        name: "",
        id: "",
      };
    },

  // Handle submit event for creating a transaction
  onSubmitTransaction(e) {
    e.preventDefault();
    if (this.createTransactionForm.amount <= 0) {
      alert("Transaction amount must be positive.");
      return;
    }
    if (this.createTransactionForm.from_account === this.createTransactionForm.to_account) {
      alert("The source and destination accounts must be different.");
      return;
    }
    const currencyRegex = /^[€$¥£₹₩₽]+$/; // Matches currency symbols
    if (!currencyRegex.test(this.createTransactionForm.currency)) {
      alert("Please enter a valid currency symbol (e.g., $, €, ¥, £).");
      return;
    }

    this.$refs.TransactionModal.hide();
    const payload = {
      amount: this.createTransactionForm.amount,
      currency: this.createTransactionForm.currency,
      from_account_id: this.createTransactionForm.from_account,
      to_account_id: this.createTransactionForm.to_account,
    };
    this.RESTcreateTransaction(payload);
    this.initTransactionForm();
  },

  // Handle submit event for creating an account
  onSubmitAccount(e) {
    e.preventDefault();
    if (this.createAccountForm.balance < 0) {
      alert("Balance must be a positive value.");
      return;
    }
    const currencyRegex = /^[€$¥£₹₩₽]+$/; // Matches currency symbols
    if (!currencyRegex.test(this.createAccountForm.currency)) {
      alert("Please enter a valid currency symbol (e.g., $, €, ¥, £).");
      return;
    }
    if (this.editAccountForm.name.length > 20) {
      alert("Account name must not exceed 20 characters.");
      return;
    }


    this.$refs.accountModal.hide();
    const payload = {
      name: this.createAccountForm.name,
      balance: this.createAccountForm.balance,
      currency: this.createAccountForm.currency,
      country: this.createAccountForm.country,
    };
    this.RESTcreateAccount(payload);
    this.initCreateAccountForm();
  },

  // Handle submit event for edit account
  onSubmitUpdate(e) {
      e.preventDefault();
      this.$refs.editAccountModal.hide();
      const payload = {
        name: this.editAccountForm.name,
      };
      this.RESTupdateAccount(payload, this.editAccountForm.id);
      this.initEditForm();
    },

    // Handle edit button
    editAccount(account) {
      this.editAccountForm = account;
    },

    // Handle Delete button
    deleteAccount(account) {
      this.RESTdeleteAccount(account.id);
    },

  created() {
    this.RESTgetAccounts();
    this.RESTgetTransactions();
  },
};
</script>


<style scoped>
/* Add your styles here */

.btn-block {
  width: 100%;
  margin-bottom: 15px;
}

.user-portal {
  background: linear-gradient(to bottom right, #8fd3e0, #8ed890);
  padding: 20px;
  color: black; /* Ensure text is black */
}
.header {
  display: flex;
  align-items: center;
  justify-content: center; /* Center the title and button horizontally */
  margin-bottom: 20px;
  position: relative;
}

.user-title {
  font-size: 48px;
  margin: 0 auto;
}

.logout-button {
  position: absolute;
  top: 0;
  right: 0;
  background-color: #d6533c;
  color: white;
  border: none;
  padding: 10px 20px;
  cursor: pointer;
  border-radius: 5px;
  font-size: 16px;
}
.logout-button:hover {
  background-color: #a53b28;
}

.section-title {
  font-size: 36px;
  margin-top: 20px;

}

.account-table,
.transactions-table {
  border: 2px solid #117b72;
  width: 100%;
  margin-top: 20px;
}

.table th {
  background-color: #f1f1f1;
  color: black;
  text-align: left;
}

.table td {
  border: 1px solid #ddd;
  padding: 10px;
  color: black; /* Ensure text in tables is black */
}

.btn-primary {
  background-color: #0648d7;
  border: none;
}

.btn-primary:hover {
  background-color: #2ea901;
}

.btn-create:hover {
  background: linear-gradient(to bottom right, #379b92, #117b72);
  color: white; /* Ensures the text stays visible */
  border-color: #3cba54; /* Matches the button color */
}

.btn-create {
  background-color: #117b72; /* Original green color */
  color: white;
  border: none; /* Removes any default border */
  font-size: 16px;
  padding: 10px 20px;
  cursor: pointer;
  width: 100%; /* Full width for the button */
  margin-bottom: 15px; /* Space below the button */
  border-radius: 5px; /* Rounded corners */
  transition: background-color 0.3s ease; /* Smooth transition for the hover effect */
}

.modal-form {
  padding: 20px;
  background-color: #f8f9fa; /* Light gray background */
  border-radius: 10px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.modal-form b-form-group {
  margin-bottom: 15px;
}

.modal-form b-form-input,
.modal-form b-form-select {
  border-radius: 5px;
  border: 1px solid #ccc; /* Light border */
  padding: 10px;
}

.modal-form b-form-input:focus,
.modal-form b-form-select:focus {
  border-color: #117b72; /* Matches button hover */
  box-shadow: 0 0 5px rgba(17, 123, 114, 0.5); /* Subtle glow */
}

.modal-submit-button {
  width: 100%;
  padding: 10px;
  font-size: 16px;
  border-radius: 5px;
  background: linear-gradient(to bottom right, #379b92, #117b72);
  color: white;
  border: none;
  transition: background-color 0.3s ease;
}

.modal-submit-button:hover {
  background: linear-gradient(to bottom right, #2ea901, #0648d7);
  color: white;
}
</style>
