<template>
  <div id="app">
    <h1>Expense Tracker</h1>
    
    <div>
      <label for="amount">Amount:</label>
      <input v-model.number="amount" type="number" id="amount" step="0.01">
    </div>
    
    <div>
      <label for="category">Category:</label>
      <div class="custom-select">
        <input 
          v-model="searchQuery" 
          @focus="showOptions = true"
          @blur="onBlur"
          placeholder="Search or select a category"
        >
        <ul v-if="showOptions">
          <li 
            v-for="category in filteredCategories" 
            :key="category"
            @mousedown="selectCategory(category)"
          >
            {{ category }}
          </li>
        </ul>
      </div>
    </div>
    
    <button @click="submitExpense">Submit</button>
    
    <div v-if="submittedExpense">
      <h2>Submitted Expense:</h2>
      <p>Amount: ${{ submittedExpense.amount }}</p>
      <p>Category: {{ submittedExpense.category }}</p>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      amount: 0,
      selectedCategory: '',
      searchQuery: '',
      categories: ['Food', 'Transportation', 'Entertainment', 'Utilities', 'Shopping'],
      submittedExpense: null,
      showOptions: false
    }
  },
  computed: {
    filteredCategories() {
      return this.categories.filter(category =>
        category.toLowerCase().includes(this.searchQuery.toLowerCase())
      )
    }
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
    submitExpense() {
      if (this.amount && this.selectedCategory) {
        this.submittedExpense = {
          amount: this.amount,
          category: this.selectedCategory
        }
        // Here you would typically send this data to a backend or store it
        console.log('Submitted:', this.submittedExpense)
      } else {
        alert('Please enter both amount and category')
      }
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

div {
  margin-bottom: 15px;
}

label {
  display: block;
  margin-bottom: 5px;
}

input {
  width: 100%;
  padding: 5px;
  box-sizing: border-box;
}

.custom-select {
  position: relative;
}

.custom-select ul {
  position: absolute;
  width: 100%;
  max-height: 200px;
  overflow-y: auto;
  list-style-type: none;
  padding: 0;
  margin: 0;
  border: 1px solid #ccc;
  background-color: white;
}

.custom-select li {
  padding: 5px;
  cursor: pointer;
}

.custom-select li:hover {
  background-color: #f0f0f0;
}

button {
  background-color: #4CAF50;
  color: white;
  padding: 10px 15px;
  border: none;
  cursor: pointer;
}

button:hover {
  background-color: #45a049;
}
</style>