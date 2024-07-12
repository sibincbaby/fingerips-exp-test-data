<template>
  <div id="app">
    <h1>Expense Tracker Form</h1>
    <div>
      <label for="amount">Amount:</label>
      <input v-model.number="amount" type="number" id="amount">
    </div>
    <div class="radio-group">
      <label>Type:</label>
      <div>
        <input type="radio" id="expense" name="type" value="Expense" v-model="type">
        <label for="expense">Expense</label>
      </div>
      <div>
        <input type="radio" id="income" name="type" value="Income" v-model="type">
        <label for="income">Income</label>
      </div>
    </div>
    <div>
      <label for="description">Description:</label>
      <input v-model="description" type="text" id="description">
    </div>
    <div>
      <label for="category">Category:</label>
      <div class="custom-select">
        <input v-model="searchQuery" @focus="showOptions = true" @blur="onBlur"
          placeholder="Search or select a category">
        <ul v-if="showOptions">
          <li v-for="category in filteredCategories" :key="category" @mousedown="selectCategory(category)">
            {{ category }}
          </li>
        </ul>
      </div>
    </div>
    <div>
      <label for="remarks">If you have chosen "Others," add your category here (optional):</label>
      <input v-model="remarks" type="text" id="remarks">
    </div>

    <button @click="submitExpense">Submit</button>
  </div>
</template>

<script>
import './styles/style.css';
export default {
  data() {
    return {
      amount: "",
      type: 'Expense', // Default to "Expense"
      description: '',
      selectedCategory: '',
      remarks: '',
      searchQuery: '',
      categories: ['Food', 'Transportation', 'Entertainment', 'Utilities', 'Shopping'],
      submittedExpense: null,
      showOptions: false,
    };
  },
  computed: {
    filteredCategories() {
      return this.categories.filter((category) =>
        category.toLowerCase().includes(this.searchQuery.toLowerCase())
      );
    },
  },
  methods: {
    selectCategory(category) {
      this.selectedCategory = category;
      this.searchQuery = category;
      this.showOptions = false;
    },
    onBlur() {
      // Delay hiding options to allow for option selection
      setTimeout(() => {
        this.showOptions = false;
      }, 200);
    },
    async submitExpense() {
      const scriptUrl = 'https://script.google.com/macros/s/AKfycbyf9E06cPOkdDEAeuVyI4dNKKJWpbe6gvGEOP8ltem2Bl4D9uNEACmPF0OtjIRcqrd8/exec';
      try {
        const formData = new FormData();
        formData.append('amount', this.amount);
        formData.append('type', this.type);
        formData.append('description', this.description);
        formData.append('category', this.selectedCategory);
        formData.append('remarks', this.remarks);
        const response = await fetch(scriptUrl, {
          method: 'POST',
          body: formData,
         
        });

        if (response.ok) {
          const result = await response.json();
          console.log('Data sent successfully:', result);
          // Optionally reset form fields or show success message
          this.amount = '';
          this.type = 'Expense'; // Reset to default
          this.description = '';
          this.selectedCategory = '';
          this.remarks = '';
          // Reset other form fields as needed
        } else {
          console.error('Failed to send data:', response.statusText);
          // Handle error, e.g., show an error message to the user
        }
      } catch (error) {
        console.error('Error submitting form:', error);
        // Handle error, e.g., show an error message to the user
      }
    }

  },
};
</script>
