<template>
  <div id="app" class="section">
    <h1 class="title">Expense Tracker Form</h1>

    <!-- Amount Input -->
    <div class="field">
      <label class="label">Amount:</label>
      <div class="control">
        <input v-model.number="amount" type="number" id="amount" class="input" placeholder="Enter the amount">
      </div>
    </div>

    <!-- Type Radio Buttons -->
    <div class="field">
      <label class="label">Type:</label>
      <div class="control is-flex is-justify-content-space-around">
        <!-- Expense Radio Button -->
        <label class="radio button is-large is-light has-background-white"
          :class="{ 'has-background-primary': type === 'Expense' }">
          <input type="radio" id="expense" name="type" value="Expense" v-model="type">
          <span class="has-text-centered">Expense</span>
        </label>

        <!-- Income Radio Button -->
        <label class="radio button is-large is-light has-background-white"
          :class="{ 'has-background-primary': type === 'Income' }">
          <input type="radio" id="income" name="type" value="Income" v-model="type">
          <span class="has-text-centered">Income</span>
        </label>
      </div>

    </div>

    <!-- Description Input -->
    <div class="field">
      <label class="label">Description <span class="has-text-danger">*</span>:</label>
      <div class="control">
        <input placeholder="Eg:Tea,Petrol,Movie,Lunch etc." v-model="description" type="text" id="desc" class="input" :class="{ 'is-danger': descriptionError }">
      </div>
      <p v-if="descriptionError" class="help is-danger">{{ descriptionError }}</p>
    </div>

    <!-- Category Select (Searchable Dropdown using Vue-multiselect) -->
    <div class="field">
      <label class="label">Category <span class="has-text-danger">*</span>:</label>
      <div class="control">
        <div class="select is-fullwidth">
          <multiselect v-model="selectedCategory" :options="type === 'Income' ? incomeCategories : expenseCategories" label="name" track-by="name" :searchable="true"
            :close-on-select="true" :clear-on-select="false" :hide-selected="true" :preserve-search="true"
            :multiple="false" :taggable="true" :select-on-tab="true"
            :class="{ 'is-open': isOpen, 'is-danger': categoryError }" @open="isOpen = true" @close="isOpen = false"
            :tag-placeholder="'Press enter to create a new category'" @tag="addTag">
            <template #tag="props">
              <span class="custom-tag">
                {{ props.option.name }}
                <i @click="props.remove(props.option)">×</i>
              </span>
            </template>
          </multiselect>
        </div>

      </div>
      <p v-if="categoryError" class="help is-danger">{{ categoryError }}</p>
    </div>
    <!-- Submit Button -->
    <div class="field">
      <div class="control">
        <button @click="submitExpense" class="button is-primary is-fullwidth" :disabled="loading">
          <span v-if="loading" class="loader"></span>
          <span v-else>Submit</span>
        </button>
      </div>
    </div>
  </div>
</template>
<script>
import Multiselect from 'vue-multiselect';
import { useToast } from 'vue-toastification';
import expenseCategoriesJSON from './assets/expenseCategories.json';
import incomeCategoriesJSON from './assets/incomeCategories.json';
export default {
  components: {
    Multiselect
  },
  data() {
    return {
      amount: "",
      type: "Expense",
      description: "",
      selectedCategory: "",
      categories: [],
      submittedExpense: null,
      isOpen: false,
      isSubmitted: false,
      loading: false,
      expenseCategories:[],
      incomeCategories:[],
    }
  },
  created() {
    // this.expenseCategories = expenseCategoriesJSON;
    this.setCategories();
  },
  setup() {
    const toast = useToast()
    return { toast }
  },
  computed: {
    descriptionError() {
      return !this.description && this.isSubmitted ? 'Description is required.' : '';
    },
    categoryError() {
      return !this.selectedCategory && this.isSubmitted ? 'Category is required.' : '';
    },
  },
  methods: {
    addTag(newTag) {
      const tag = { name: newTag };
      this.selectedCategory = tag;
      if(this.type === 'Income'){
        this.incomeCategories.push(tag);
        localStorage.setItem('incomeCategories', JSON.stringify(this.expenseCategories));
      }else{
        this.expenseCategories.push(tag);
        localStorage.setItem('expenseCategories', JSON.stringify(this.expenseCategories));
      }
    },
    isFormValid() {
      return this.description && this.selectedCategory;
    },
    mounted() {
      
    },
    async setCategories(){
      let incomeCategories = JSON.parse(localStorage.getItem('incomeCategories')) || [];
      let expenseCategories = JSON.parse(localStorage.getItem('expenseCategories')) || [];
      if(incomeCategories.length == 0){
        localStorage.setItem('incomeCategories', JSON.stringify(incomeCategoriesJSON));
      }
      if(expenseCategories.length == 0){
        localStorage.setItem('expenseCategories', JSON.stringify(expenseCategoriesJSON));
      }
      this.incomeCategories = JSON.parse(localStorage.getItem('incomeCategories')) || [];
      this.expenseCategories = JSON.parse(localStorage.getItem('expenseCategories')) || [];
    },
    generateUniqueId() {
      return `${Date.now().toString(36)}`;
    },
    async saveToLocal(data) {
      let expenses = JSON.parse(localStorage.getItem('expenses')) || [];
      expenses.push(data);
      localStorage.setItem('expenses', JSON.stringify(expenses));
    },
    async submitExpense() {
      this.isSubmitted = true;
      if (this.isFormValid()) {
        this.loading = true;
        try {
          let formData ={};
          formData.id = this.generateUniqueId();
          formData.amount = this.amount;
          formData.type = this.type;
          formData.description=  this.description;
          formData.category = this.selectedCategory ? this.selectedCategory.name : '';
          await this.saveToLocal(formData);
          this.resetFileds();
          this.toast.success('Data submitted successfully!');
          this.isSubmitted = false;
          this.loading = false;
          await this.processLocalStorage();
        } catch (error) {
          console.error('Error submitting form:', error);
        }
      }
    },
    async processLocalStorage() {
      const expenses = JSON.parse(localStorage.getItem('expenses')) || [];
      const scriptUrl = 'https://script.google.com/macros/s/AKfycbz1ntHueHZPMGDmsSLLkgIEXl7GiygF1L1c0mJ4IeoQZCpMlRMpFvByr4iixs7AFNJo/exec';
      for (const expense of expenses) {
        try {
          const formData = new FormData();
          formData.append('amount', expense.amount);
          formData.append('type', expense.type);
          formData.append('description', expense.description);
          formData.append('category', expense.category);

          const response = await fetch(scriptUrl, {
            method: 'POST',
            body: formData,
            mode: 'no-cors'
          });
          console.log(response);
          let storedExpenses = JSON.parse(localStorage.getItem('expenses')) || [];
          storedExpenses = storedExpenses.filter(e => e.id !== expense.id);
          localStorage.setItem('expenses', JSON.stringify(storedExpenses));
        } catch (error) {
          console.error('Error submitting stored expense:', error);
        }
      }
    },
    resetFileds() {
      this.amount = '';
      this.type = "Expense";
      this.description = '';
      this.selectedCategory = "";
    }
  }
}
</script>

<style scoped>
#app {
  font-family: Arial, sans-serif;
  max-width: 400px;
  margin: 0 auto;
  padding: 20px;
}

.field {
  margin-bottom: 1.5rem;
}

/* Custom styles for radio buttons */
.radio {
  cursor: pointer;
  /* Make the cursor pointer for better usability */
  border-radius: 999px;
  /* Ensure round shape for large clickable area */
  padding: 0.75rem 1.5rem;
  /* Adjust padding for larger clickable area */
  margin-right: 1rem;
  /* Space between radio buttons */
  transition: background-color 0.3s, color 0.3s;
  /* Smooth transition for visual feedback */

  /* Optional: Adjust text size and weight */
  font-size: 1rem;
  font-weight: 500;
}

.radio input[type="radio"] {
  display: none;
}

.radio.has-background-primary {
  background-color: #3273dc;
  /* Custom primary color */
  color: white;
  /* Text color for active state */
}
/* Inactive state style (optional) */
.radio:not(.has-background-primary) {
  background-color: #ffffff; /* Default background color for inactive state */
  color: #4a4a4a; /* Default text color for inactive state */
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.1); /* Shadow effect for inactive state */
}
</style>
