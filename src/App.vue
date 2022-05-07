<template>
  <div class="app">
    <div class="logo_and_title_container">
      <img class="logo" src="./assets/images/api-logo.png" alt="API logo">
      <h2 class="title">Public APIs</h2>
    </div>
    <hr>
    <h3 class="categories_title">Categories</h3>
    <div class="api_categories_container">
      <button v-bind:class="{ active: chosenCategory === 'All APIs' }" @click="selectCategory('All APIs')">All APIs</button>
      <button v-for="(category, index) in categories.slice(firstCategoryID, lastCategoryID)" :key="index" v-bind:class="{ active: chosenCategory === category }" :title="category" @click="selectCategory(category)">{{category}}</button>
    </div>
    <div v-if="lastCategoryID < categories.length" @click="lastCategoryID = categories.length">
      <i class='angle down icon'></i>
      <a href="#/" class="link" >
      Show more
      </a>
    </div>
    <div v-if="lastCategoryID === categories.length" @click="lastCategoryID = 9">
      <i class='angle up icon'></i>
      <a href="#/" class="link">
      Show less
      </a>
    </div>
    <hr>
    <div v-if="savedEntries.length > 0" class="chosen_category_and_search_container">
      <h3 class="chosen_category">{{chosenCategory}} ({{entries.length}})</h3>
      <div class="search_container">
        <input placeholder="Search all APIs" type="text" v-model="searchInput" @keyup="search()">
      </div>
    </div>
    <div class="api_list_container" id="api_list_container">
      <a v-for="(entry, index) in entries.slice(firstEntryID, lastEntryID)" :key="index" :href="entry.Link" :title="entry.API" target="_blank">
        <h4 class="api_title">{{entry.API}}</h4>
        <p :href="entry.Link" class="api_description">{{entry.Description}}</p>
      </a>
    </div>
    <div v-if="entries.length > 8 && lastEntryID < entries.length" class="show_more_apis" @click="showMoreAPIs()" >
      <i class='angle down icon'></i>
      <a href="#/" class="link">
      Show more
      </a>
    </div>
  </div>
</template>

<script>


export default {
  name: 'app',
  mounted() {
    this.fetchGetCategories();
  },
  methods: {
    fetchGetCategories() {
      fetch('https://api.publicapis.org/categories')
        .then(response => response.json())
        .then((data) => {
          this.categories = data.categories;
          this.fetchGetAPIs();
        });
    },
    fetchGetAPIs() {
      fetch('https://api.publicapis.org/entries')
        .then(response => response.json())
        .then((data) => {
          this.entries = data.entries;
          this.savedEntries = this.entries;
        })
        .catch((err) => {
          document.getElementById('api_list_container').innerHTML = `Server error. Please, reload the page. <br> Details: ${err}`;
        });
    },
    clickCallback(currentPage) {
      if (this.currentPage !== currentPage) {
        localStorage.setItem('pagination', currentPage);
        this.currentPage = currentPage;
      }
    },
    selectCategory(chosenCategory) {
      this.searchInput = '';
      this.lastEntryID = 8;
      if (chosenCategory === 'All APIs') {
        this.entries = this.savedEntries;
        this.chosenCategory = 'All APIs';
      } else {
        this.entries = this.savedEntries.filter(entry => entry.Category === chosenCategory);
        this.chosenCategory = chosenCategory;
      }
    },
    search() {
      if (this.searchInput.length < 1) {
        this.chosenCategory = 'All APIs';
        this.entries = this.savedEntries;
      } else {
        this.chosenCategory = `Searching "${this.searchInput}" in all APIs`;
        this.entries = this.savedEntries.filter(entry => entry.API.toLowerCase().includes(this.searchInput));
      }
    },
    showMoreAPIs() {
      this.lastEntryID = this.lastEntryID + 8;
    },
  },
  data() {
    return {
      entries: [],
      savedEntries: [],
      firstEntryID: 0,
      lastEntryID: 8,
      pagesInPagination: 1,
      currentPage: 1,
      prevPage: '',
      nextPage: '',
      pages: [],
      categories: [],
      chosenCategory: 'All APIs',
      firstCategoryID: 0,
      lastCategoryID: 9,
      searchInput: '',
    };
  },
};
</script>

<style>

@import './assets/styles/styles.css';

</style>
