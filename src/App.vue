<template>
   <div class="container">
      <div class="multiselect-wrapper">
         <h2>Multiselect with auto complete</h2>
         <div class="multiselect-holder">
            <div class="multiselect-button d-flex align-items-center flex-wrap" :class="{'active': isOpen}">
               <span class="multiselect-button__text" v-if="selectedCountries.length <= 0" @click="isOpen = !isOpen">Select</span>
               <span class="multiselect-button__tag d-flex-inline align-items-center" v-for="(sl, index) in selectedCountries" :key="index">
                  <span class="multiselect-button__tag--flag d-flex align-items-center">
                     <img :src="sl.flag" width="20" />
                  </span>
                  <span class="multiselect-button__tag--text d-flex align-items-center">{{ sl.name }}</span>
                  <span class="multiselect-button__tag--remove d-flex align-items-center justify-content-center" @click.stop="unselectCountries(sl)">&times;</span>
               </span>
            </div>
            <div class="multiselect-dropdown" :class="{'active': isOpen}" v-if="isOpen">
               <div class="multiselect-dropdown__search">
                  <input type="text" placeholder="Search by country name" v-model="search" @input="onChange" />
               </div>
               <div class="multiselect-dropdown__items">
                  <div v-if="filterCountries.length > 0">
                     <div class="multiselect-dropdown__items--item d-flex align-items-center" :class="{'active': selectedCountries.includes(country)}" v-for="country in filterCountries" :key="country.name">
                        <div class="custom-checkbox">
                           <input type="checkbox" :checked="selectedCountries.includes(country)" :value="country.name" @change="selectCountry(country)" />
                           <span class="custom-checkbox-area"></span>
                        </div>
                        <div class="country-info d-flex align-items-center">
                           <span class="country-info__flag d-flex align-items-center">
                              <img :src="country.flag" :alt="country.name" width="25" />
                           </span>
                           <span class="country-info__name d-flex align-items-center">
                              {{ country.name }}
                           </span>
                        </div>
                     </div>
                  </div>
                  <div v-else class="multiselect-dropdown__items--item">
                     <h3 class="mb-0">No matching found!</h3>
                  </div>
               </div>
            </div>
         </div>
      </div>
   </div>
</template>

<script>
import config from './config';
import axios from 'axios';
export default {
   name: 'App',
   data () {
      return {
         search: '',
         isOpen: false,
         countries: [],
         selectedCountries: [],
         filterCountries: [],
      }
   },
   mounted () {
      this.getCountries();
      document.addEventListener('click', this.closeAutoComplete)
   },
   methods: {
      closeAutoComplete(e) {
         if(!this.$el.firstElementChild.children[1].contains(e.target)) {
            this.isOpen = false;
         }
      },
      /* Filter countries on search input */
      searchCountries() {
         return this.filterCountries = this.countries.filter((country) => {
            const countryName = JSON.stringify(country.name);
            return (countryName.toLowerCase().includes(this.search.toLowerCase()));
         });
      },
      /* Trigger filterContries methods */
      onChange() {
         this.searchCountries();
      },
      /* Check country is selected or not */
      selectCountry(country) {
         if(!this.selectedCountries.includes(country)) {
            this.selectedCountries.push(country);
         } else {
            this.selectedCountries.splice(this.selectedCountries.indexOf(country), 1);
         }
      },
      unselectCountries(country) {
         if(this.selectedCountries.includes(country)) {
            this.selectedCountries.splice(this.selectedCountries.indexOf(country), 1);
         }
      },
      /* Get the all countries */
      async getCountries() {
         const countries = localStorage.getItem('countries');
         if(countries) {
            this.countries = JSON.parse(countries);
            this.filterCountries = this.countries;
            return;
         }
         const response = await axios.get(`${config.apiURL}`);
         try {
            if(response.status == 200) {
               localStorage.setItem('countries', JSON.stringify(response.data));
            }
         } catch (e) {
            console.log(e);
         }
      }
   }
}
</script>
