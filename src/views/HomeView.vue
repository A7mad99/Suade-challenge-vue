<script setup>
import { ref, computed, onMounted } from 'vue'
const apiKey = import.meta.env.VITE_API_KEY

const rows = ref([])
const loading = ref(false)
const error = ref('')

const selectedEyeColor = ref('')
const selectedPet = ref('')
const selectedFruit = ref('')
const sortBy = ref('')
const sortOrder = ref('asc')

function getLastName(fullName) {
  if (!fullName) return ''
  const parts = fullName.trim().split(' ')
  return parts[parts.length - 1]
}
function getFirstName(fullName) {
  if (!fullName) return ''

  const parts = fullName.trim().split(' ')
  return parts[0]
}

onMounted(async () => {
  loading.value = true
  error.value = ''

  try {
    const response = await fetch(apiKey)

    if (!response.ok) {
      throw new Error('Error failed to fetch data')
    }

    const result = await response.json()
    console.log(result)

    rows.value = Array.isArray(result) ? result : result.data || []
  } catch (err) {
    error.value = err.message || 'Error while fetching data'
  } finally {
    loading.value = false
  }
})

const eyeColors = computed(() => {
  return [...new Set(rows.value.map((item) => item.eyeColor).filter(Boolean))]
})

const pets = computed(() => {
  return [...new Set(rows.value.map((item) => item.preferences?.pet).filter(Boolean))]
})

const fruits = computed(() => {
  return [...new Set(rows.value.map((item) => item.preferences?.fruit).filter(Boolean))]
})

const filteredRows = computed(() => {
  let result = [...rows.value]

  if (selectedEyeColor.value) {
    result = result.filter((item) => item.eyeColor === selectedEyeColor.value)
  }

  if (selectedPet.value) {
    result = result.filter((item) => item.preferences?.pet === selectedPet.value)
  }

  if (selectedFruit.value) {
    result = result.filter((item) => item.preferences?.fruit === selectedFruit.value)
  }

  if (sortBy.value === 'age') {
    result.sort((a, b) => {
      return sortOrder.value === 'asc' ? a.age - b.age : b.age - a.age
    })
  }

  if (sortBy.value === 'lastName') {
    result.sort((a, b) => {
      const aLastName = getLastName(a.name)
      const bLastName = getLastName(b.name)

      return sortOrder.value === 'asc'
        ? aLastName.localeCompare(bLastName)
        : bLastName.localeCompare(aLastName)
    })
  }

  return result
})

function clearFilters() {
  selectedEyeColor.value = ''
  selectedPet.value = ''
  selectedFruit.value = ''
  sortBy.value = ''
  sortOrder.value = 'asc'
}
</script>

<template>
  <main class="page">
    <h1>Research Data Table</h1>

    <div class="controls">
      <select v-model="selectedEyeColor">
        <option value="">All eye colors</option>
        <option v-for="color in eyeColors" :key="color" :value="color">
          {{ color }}
        </option>
      </select>

      <select v-model="selectedPet">
        <option value="">All pets</option>
        <option v-for="pet in pets" :key="pet" :value="pet">
          {{ pet }}
        </option>
      </select>

      <select v-model="selectedFruit">
        <option value="">All fruits</option>
        <option v-for="fruit in fruits" :key="fruit" :value="fruit">
          {{ fruit }}
        </option>
      </select>

      <select v-model="sortBy">
        <option value="">No sorting</option>
        <option value="lastName">Sort by last name</option>
        <option value="age">Sort by age</option>
      </select>

      <select v-model="sortOrder">
        <option value="asc">Ascending</option>
        <option value="desc">Descending</option>
      </select>

      <button @click="clearFilters">Clear</button>
    </div>

    <p v-if="loading">Loading data...</p>
    <p v-else-if="error">{{ error }}</p>

    <div v-else class="table-wrapper">
      <table>
        <thead>
          <tr>
            <th>Name</th>
            <th>Last Name</th>
            <th>Age</th>
            <th>Eye Color</th>
            <th>Pet Preference</th>
            <th>Fruit Preference</th>
            <th>Gender</th>
          </tr>
        </thead>
        <tbody>
          <tr v-if="filteredRows.length === 0">
            <td colspan="7">No results found.</td>
          </tr>

          <tr v-for="person in filteredRows" :key="person._id">
            <td>{{ getFirstName(person.name) }}</td>
            <td>{{ getLastName(person.name) }}</td>
            <td>{{ person.age }}</td>
            <td>{{ person.eyeColor }}</td>
            <td>{{ person.preferences?.pet }}</td>
            <td>{{ person.preferences?.fruit }}</td>
            <td>{{ person.gender }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </main>
</template>

<style scoped>
.page {
  max-width: 1100px;
  margin: 0 auto;
  padding: 24px;
}

h1 {
  margin-bottom: 20px;
}

.controls {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(160px, 1fr));
  gap: 12px;
  margin-bottom: 20px;
}

select,
button {
  padding: 10px;
  font-size: 14px;
}

button {
  cursor: pointer;
}

.table-wrapper {
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
  background: white;
}

th,
td {
  text-align: left;
  padding: 12px;
  border-bottom: 1px solid #ddd;
}

th {
  font-weight: 600;
}
</style>
