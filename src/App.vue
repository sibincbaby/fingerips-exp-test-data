<template>
  <div id="app" class="section">
    <h1 class="title">Expense Tracker Form</h1>

    <!-- Amount Input -->
    <div class="field">
      <label class="label">Amount:</label>
      <div class="control">
        <input v-model.number="amount" type="number" id="amount" step="0.01" class="input">
      </div>
    </div>

    <!-- Type Radio Buttons -->
    <div class="field">
      <label class="label">Type:</label>
      <div class="control">
        <label class="radio">
          <input type="radio" id="expense" name="type" value="Expense" v-model="type">
          Expense
        </label>
        <label class="radio">
          <input type="radio" id="income" name="type" value="Income" v-model="type">
          Income
        </label>
      </div>
    </div>

    <!-- Description Input -->
    <div class="field">
      <label class="label">Description <span class="has-text-danger">*</span>:</label>
      <div class="control">
        <input v-model="description" type="text" id="desc" class="input" :class="{ 'is-danger': descriptionError }">
      </div>
      <p v-if="descriptionError" class="help is-danger">{{ descriptionError }}</p>
    </div>

    <!-- Category Select (Searchable Dropdown using Vue-multiselect) -->
    <div class="field">
      <label class="label">Category:</label>
      <div class="control">
        <div class="select is-fullwidth">

          <multiselect 
          v-model="selectedCategory" 
          :options="categories" 
          label="name" 
          track-by="name" 
          :searchable="true"
          :close-on-select="true"
          :clear-on-select="false"
          :hide-selected="true"
          :preserve-search="true"
          :multiple="false"
          :taggable="true"
          :select-on-tab="true"
          :class="{ 'is-open': isOpen, 'is-danger': categoryError }"
          @open="isOpen = true"
          @close="isOpen = false"
          @tag="addTag"
        >
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

    <!-- Remarks Input -->
    <div class="field">
      <label class="label">Remarks:</label>
      <div class="control">
        <textarea v-model="remarks" id="remarks" class="textarea"></textarea>
      </div>
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
// Import Vue-multiselect
import Multiselect from 'vue-multiselect';
import { useToast } from 'vue-toastification';
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
      remarks: "",
        categories: [
        { name: 'Food' },
        { name: 'Transportation' },
        { name: 'Entertainment' },
        { name: 'Utilities' },
        { name: 'Shopping' }
      ],
      submittedExpense: null,
      isOpen: false,
      isSubmitted:false,
      loading: false
    }
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
      this.categories.push(tag);
      this.selectedCategory=tag;
    },
    isFormValid() {
      return this.description && this.selectedCategory;
    },
    async submitExpense() {
      this.loading = true;
      this.isSubmitted = true;
      if(this.isFormValid()){
        
     
      const scriptUrl = 'https://script.google.com/macros/s/AKfycbz1ntHueHZPMGDmsSLLkgIEXl7GiygF1L1c0mJ4IeoQZCpMlRMpFvByr4iixs7AFNJo/exec';
      try {
        const formData = new FormData();
        formData.append('amount', this.amount);
        formData.append('type', this.type);
        formData.append('description', this.description);
        formData.append('category', this.selectedCategory ? this.selectedCategory.name : '');
        formData.append('remarks', this.remarks);
        const response = await fetch(scriptUrl, {
          redirect:'follow',
          method: 'POST',
          body: formData,
          mode: 'no-cors'
        });
        console.log(response);
        this.resetFileds();
        this.toast.success('Data submitted successfully!');
        this.isSubmitted = false;
        this.loading = false;
        // if (response.ok) {
        //   console.log('Data sent successfully');
        // } else {
        //   console.error('Failed to send data:', response.statusText);
        // }
      } catch (error) {
        console.error('Error submitting form:', error);
      }
    }
    },
    resetFileds(){
      this.amount = '';
      this.type = "Expense";
      this.description = '';
      this.selectedCategory = "";
      this.remarks = '';
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
</style>
