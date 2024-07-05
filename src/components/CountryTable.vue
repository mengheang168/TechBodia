<!-- src/components/CountryTable.vue -->
<script setup>
import axios from 'axios'
import { ref, onMounted, computed } from 'vue'
import { Modal } from 'bootstrap'

const countries = ref(null)
const searchCountryName = ref('')
const selectedCountry = ref(null)
let modalInstance = null

// Pagination variables
const currentPage = ref(1);
    const itemsPerPage = ref(25);

onMounted(() => {
  getCountry()
  const modalElement = document.getElementById('countryModal')
  modalInstance = new Modal(modalElement)
})

const getCountry = async () => {
  try {
    const response = await axios.get('https://restcountries.com/v3.1/all')
    countries.value = response.data
  } catch (error) {
    console.log('Error fetching data:', error)
  }
}

// Filter function
const filterCountries = () => {
  return countries.value
    ? countries.value.filter((country) =>
        country.name.official?.toLowerCase().includes(searchCountryName.value.toLowerCase())
      )
    : []
}

// Computed property to get filtered countries
const filteredCountries = computed(() => {
  return filterCountries()
})

// Sort Order
const sortByName = (order) => {
  countries.value.sort((a, b) => {
    if (order === 'asc') {
      return a.name.official.localeCompare(b.name.official)
    } else {
      return b.name.official.localeCompare(a.name.official)
    }
  })
}
// Show Country Detail
const selectCountry = (country) => {
  selectedCountry.value = country
  modalInstance.show()
}

const closeModal = () => {
  modalInstance.hide()
}

// Computed property for native names
const nativeName = computed(() => {
  if (selectedCountry.value) {
    const nativeNames = selectedCountry.value.name.nativeName
    return nativeNames
      ? Object.values(nativeNames)
          .map((name) => name.official)
          .join(', ')
      : 'N/A'
  }
  return ''
})


// Pagination Computed
// Computed property for paginated countries
const paginatedCountries = computed(() => {
      const start = (currentPage.value - 1) * itemsPerPage.value;
      const end = start + itemsPerPage.value;
      return filteredCountries.value.slice(start, end);
    });

    // Computed property for total pages
    const totalPages = computed(() => {
      return Math.ceil(filteredCountries.value.length / itemsPerPage.value);
    });

    // Method to go to a specific page
    const goToPage = (page) => {
      if (page >= 1 && page <= totalPages.value) {
        currentPage.value = page;
      }
    };




// Search data from API
// const getCountryName= async ()=>{
//         try{
//                 if(searchCountryName.value!=""){
//                         const searchName=await fetch(`https://restcountries.com/v3.1/name/${searchCountryName.value}`);
//                         const data= await searchName.json();
//                         if(searchName.ok){
//                                 countries.value=data;
//                                 error.value = null;
//                         }else{
//                                 countries.value=null;
//                                 error.value='Error: ${data.message}';
//                         }
//                 }else{
//                         getCountry();
//                 }

//         }catch(err){
//                 console.error("Error fetching data:", err);
//                 error.value = "An error occurred while fetching data.";
//         }
// }
</script>
<template>
  <div class="container-fluid mt-4">
        <h1 style="color: #FE0076;">TECHBODIA</h1>
    <input
      v-model="searchCountryName"
      placeholder="Search by Country Name"
      class="form-control mb-3"
    />
    <div class="d-flex justify-content-between mb-3">
      <button class="btn btn-primary" @click="sortByName('asc')">Sort Ascending</button>
      <button class="btn btn-secondary" @click="sortByName('desc')">Sort Descending</button>
    </div>
    <table class="table table-striped">
      <thead>
        <tr>
          <th>Flag</th>
          <th>Country Name</th>
          <th>2-Code</th>
          <th>3-Code</th>
          <th>Native Name</th>
          <th>Alternative Names</th>
          <th>Calling Codes</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="country in paginatedCountries" :key="country.cca3">
          <td><img :src="country.flags.png" alt="Flag" width="50" /></td>
          <td>
            <a @click="selectCountry(country)" style="cursor: pointer">{{
              country.name.official
            }}</a>
          </td>
          <td>{{ country.cca2 }}</td>
          <td>{{ country.cca3 }}</td>
          <td v-if="country.name.nativeName">
            {{ Object.values(country.name.nativeName)[0].official }}
          </td>
          <td v-else>N/A</td>
          <td>{{ country.altSpellings.join(', ') }}</td>
          <td>{{ country.idd.root }}</td>
        </tr>
      </tbody>
    </table>
    

<!-- Pagination Controls -->
<nav aria-label="Page navigation" v-if="totalPages > 1" class="mt-3">
      <ul class="pagination">
        <li class="page-item" :class="{ disabled: currentPage === 1 }">
          <a class="page-link" href="#" @click.prevent="goToPage(1)">First</a>
        </li>
        <li class="page-item" :class="{ disabled: currentPage === 1 }">
          <a class="page-link" href="#" @click.prevent="goToPage(currentPage - 1)">Previous</a>
        </li>
        <li class="page-item" v-for="page in totalPages" :key="page" :class="{ active: currentPage === page }">
          <a class="page-link" href="#" @click.prevent="goToPage(page)">{{ page }}</a>
        </li>
        <li class="page-item" :class="{ disabled: currentPage === totalPages }">
          <a class="page-link" href="#" @click.prevent="goToPage(currentPage + 1)">Next</a>
        </li>
        <li class="page-item" :class="{ disabled: currentPage === totalPages }">
          <a class="page-link" href="#" @click.prevent="goToPage(totalPages)">Last</a>
        </li>
      </ul>
    </nav>



  </div>

  <!-- Modal for displaying country details -->
  <div
    class="modal fade"
    id="countryModal"
    tabindex="-1"
    aria-labelledby="countryModalLabel"
    aria-hidden="true"
  >
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title" id="countryModalLabel">{{ selectedCountry?.name?.official }}</h5>
          <button
            type="button"
            class="btn-close"
            data-bs-dismiss="modal"
            aria-label="Close"
          ></button>
        </div>
        <div class="modal-body" v-if="selectedCountry">
          <img :src="selectedCountry.flags.png" alt="Flag" class="img-fluid mb-3" />
          <p><strong>2-character Country Code:</strong> {{ selectedCountry.cca2 }}</p>
          <p><strong>3-character Country Code:</strong> {{ selectedCountry.cca3 }}</p>
          <p><strong>Native Country Name:</strong> {{ nativeName }}</p>
          <p>
            <strong>Alternative Country Names:</strong>
            {{ selectedCountry.altSpellings.join(', ') }}
          </p>
          <p>
            <strong>Country Calling Codes:</strong> +{{ selectedCountry.idd.root
            }}{{ selectedCountry.idd.suffixes.join(', +') }}
          </p>
        </div>
        <div class="modal-footer">
          <button type="button" class="btn btn-secondary" @click="closeModal">Close</button>
        </div>
      </div>
    </div>
  </div>
</template>
