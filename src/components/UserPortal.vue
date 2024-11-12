<template>
    <div class="jumbotron vertical-center">
      <div class="container main-content">
        <div class="row">
          <div class="col-sm-12">
            <h1>My Accounts</h1>
            
            <!-- Alert Message -->
            <b-alert v-if="showMessage" :variant="alertVariant" show>
              {{ message }}
            </b-alert>
    
            <!-- Account Table -->
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
                    <span
                      v-if="account.account_status === 'Active'"
                      class="badge badge-success"
                    >{{ account.status }}</span>
                    <span v-else class="badge badge-danger">{{ account.account_status }}</span>
                  </td>
                  <td>
                    <!-- Make Transfer Button for Each Account -->
                    <button
                      type="button"
                      class="btn btn-primary btn-sm"
                      @click="openTransferModal(account)"
                    >
                      Make Transfer
                    </button>
                  </td>
                </tr>
              </tbody>
            </table>
    
            <!-- Transactions Table -->
            <h1>Recent Transactions</h1>
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
    
            <footer class="text-center">
              Copyright &copy; All Rights Reserved.
            </footer>
          </div>
        </div>
    
        <!-- Transfer Modal -->
        <b-modal
          ref="transferModal"
          id="transfer-modal"
          title="Transfer Money"
          hide-backdrop
          hide-footer
        >
          <b-form @submit="onSubmitTransfer" class="w-100">
            <b-form-group label="From Account:" label-for="from-account-input">
              <b-form-input
                id="from-account-input"
                type="text"
                :value="transferForm.fromAccountNumber"
                readonly
              />
            </b-form-group>
            <b-form-group
              label="To Account Number:"
              label-for="to-account-input"
            >
              <b-form-input
                id="to-account-input"
                type="text"
                v-model="transferForm.toAccountNumber"
                placeholder="Enter recipient account number"
                required
              />
            </b-form-group>
            <b-form-group label="Amount:" label-for="amount-input">
              <b-form-input
                id="amount-input"
                type="number"
                v-model="transferForm.amount"
                :max="transferForm.maxAmount"
                placeholder="Enter transfer amount"
                required
              />
            </b-form-group>
            <b-button type="submit" variant="outline-info">Submit Transfer</b-button>
          </b-form>
        </b-modal>
      </div>
    </div>
  </template>
  
  
  <script>
    import axios from "axios";
    export default {
      name: "AppAccounts",
      data() {
        return {
          accounts: [],
          transactions: [],
          transferForm: {
            fromAccountId: "",
            toAccountNumber: "",
            amount: 0
          },
          showMessage: false,
          message: "",
          alertVariant: "success"
        };
      },
      methods: {
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
        RESTtransferMoney(payload) {
          const path = `${process.env.VUE_APP_ROOT_URL}/transfer`;
          axios
            .post(path, payload)
            .then((response) => {
              this.RESTgetAccounts();
              this.RESTgetTransactions();
              this.message = "Transfer Successful!";
              this.alertVariant = "success";
              this.showMessage = true;
              setTimeout(() => {
                this.showMessage = false;
              }, 3000);
            })
            .catch((error) => {
              console.error(error);
              this.message = "Transfer Failed!";
              this.alertVariant = "danger";
              this.showMessage = true;
              setTimeout(() => {
                this.showMessage = false;
              }, 3000);
            });
        },
        prepareTransfer(account) {
          this.transferForm.fromAccountId = account.id;
          this.$refs.transferModal.show();
        },
        onSubmitTransfer(e) {
          e.preventDefault();
          this.$refs.transferModal.hide();
  
          const fromAccount = this.accounts.find(
            (acc) => acc.id === this.transferForm.fromAccountId
          );
          if (this.transferForm.amount > fromAccount.balance) {
            this.message = "Insufficient funds for transfer!";
            this.alertVariant = "danger";
            this.showMessage = true;
            setTimeout(() => {
              this.showMessage = false;
            }, 3000);
            return;
          }
  
          const payload = {
            fromAccountId: this.transferForm.fromAccountId,
            toAccountNumber: this.transferForm.toAccountNumber,
            amount: this.transferForm.amount,
          };
          this.RESTtransferMoney(payload);
        }
      },
      created() {
        this.RESTgetAccounts();
        this.RESTgetTransactions();
      },
    };
  </script>
  
  <style scoped>
    .main-content {
    margin-left: 2%;
    width: 100%;
  }
  h1 {
    font-size: 2.5rem; /* Adjust font size as needed */
    margin-bottom: 10px; /* Reduce the space below the title */
}

.account-table {
    margin-top: 10; /* Remove extra space at the top of the table */
    /* Ensure the table fills the container width if needed */
    width: 100%;
    padding-top: 0; 
}

  .table {
    width: 100%;
  }
  .account-table, .transactions-table {
    font-size: 0.9rem;
  }
  .account-table th, .account-table td,
  .transactions-table th, .transactions-table td {
    padding: 8px 12px;
  }
  </style>
  