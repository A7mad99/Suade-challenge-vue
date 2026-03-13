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

function getFirstName(fullName) {
  if (!fullName) return ''
  return fullName.trim().split(/\s+/)[0]
}

function getLastName(fullName) {
  if (!fullName) return ''
  const parts = fullName.trim().split(/\s+/)
  return parts[parts.length - 1]
}

function clearFilters() {
  selectedEyeColor.value = ''
  selectedPet.value = ''
  selectedFruit.value = ''
  sortBy.value = ''
  sortOrder.value = 'asc'
}

function toggleSort(column) {
  if (sortBy.value === column) {
    sortOrder.value = sortOrder.value === 'asc' ? 'desc' : 'asc'
  } else {
    sortBy.value = column
    sortOrder.value = 'asc'
  }
}

function getSortSymbol(column) {
  if (sortBy.value !== column) return '↕'
  return sortOrder.value === 'asc' ? '↑' : '↓'
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
    rows.value = Array.isArray(result) ? result : result.data || []
  } catch (err) {
    error.value = err.message || 'Error while fetching data'
  } finally {
    loading.value = false
  }
})

const eyeColors = computed(() => {
  return [...new Set(rows.value.map((item) => item.eyeColor).filter(Boolean))].sort()
})

const pets = computed(() => {
  return [...new Set(rows.value.map((item) => item.preferences?.pet).filter(Boolean))].sort()
})

const fruits = computed(() => {
  return [...new Set(rows.value.map((item) => item.preferences?.fruit).filter(Boolean))].sort()
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
</script>

<template>
  <main class="page">
    <section class="hero">
      <div>
        <h1>Research Data Table</h1>

      </div>

      <div class="summary-card">
        <span class="summary-label">Showing</span>
        <strong>{{ filteredRows.length }}</strong>
        <span class="summary-label">results</span>
      </div>
    </section>

    <section class="filters-panel">
      <div class="filters-header">
        <h2>Filters</h2>
        <button class="clear-btn" @click="clearFilters">Clear all</button>
      </div>

      <div class="controls">
        <div class="field">
          <label for="eyeColor">Eye color</label>
          <select id="eyeColor" v-model="selectedEyeColor">
            <option value="">All eye colors</option>
            <option v-for="color in eyeColors" :key="color" :value="color">
              {{ color }}
            </option>
          </select>
        </div>

        <div class="field">
          <label for="pet">Pet preference</label>
          <select id="pet" v-model="selectedPet">
            <option value="">All pets</option>
            <option v-for="pet in pets" :key="pet" :value="pet">
              {{ pet }}
            </option>
          </select>
        </div>

        <div class="field">
          <label for="fruit">Fruit preference</label>
          <select id="fruit" v-model="selectedFruit">
            <option value="">All fruits</option>
            <option v-for="fruit in fruits" :key="fruit" :value="fruit">
              {{ fruit }}
            </option>
          </select>
        </div>
      </div>
    </section>

    <section class="table-panel">
      <div class="table-topbar">
        <div>

          <p class="table-help">Click the Last Name or Age column header to sort.</p>
        </div>
      </div>

      <p v-if="loading" class="status-message">Loading data...</p>
      <p v-else-if="error" class="status-message error">{{ error }}</p>

      <div v-else class="table-wrapper">
        <table>
          <thead>
            <tr>
              <th>First Name</th>
              <th>
                <button
                  type="button"
                  class="sort-button"
                  @click="toggleSort('lastName')"
                >
                  <span>Last Name</span>
                  <span class="sort-icon">{{ getSortSymbol('lastName') }}</span>
                </button>
              </th>
              <th>
                <button
                  type="button"
                  class="sort-button"
                  @click="toggleSort('age')"
                >
                  <span>Age</span>
                  <span class="sort-icon">{{ getSortSymbol('age') }}</span>
                </button>
              </th>
              <th>Eye Color</th>
              <th>Pet Preference</th>
              <th>Fruit Preference</th>
              <th>Gender</th>
            </tr>
          </thead>

          <tbody>
            <tr v-if="filteredRows.length === 0">
              <td colspan="7" class="empty-state">No results found.</td>
            </tr>

            <tr v-for="person in filteredRows" :key="person._id">
              <td>{{ getFirstName(person.name) }}</td>
              <td>{{ getLastName(person.name) }}</td>
              <td>{{ person.age }}</td>
              <td>
                <span class="pill">{{ person.eyeColor }}</span>
              </td>
              <td>{{ person.preferences?.pet }}</td>
              <td>{{ person.preferences?.fruit }}</td>
              <td>{{ person.gender }}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </section>
  </main>
</template>

<style scoped>
.page {
  width: 100%;
  max-width: 100%;
  min-height: 100vh;
  padding: 32px 24px 48px;
  box-sizing: border-box;
  background: linear-gradient(180deg, #f8fafc 0%, #eef2f7 100%);
  color: #1f2937;
}

.hero,
.filters-panel,
.table-panel {
  width: 100%;
  max-width: 1400px;
  margin: 0 auto 24px;
}

.hero {
  display: flex;
  justify-content: space-between;
  align-items: end;
  gap: 20px;
}

.hero h1 {
  margin: 0;
  font-size: clamp(2rem, 3vw, 3rem);
  line-height: 1.1;
  color: #0f172a;
}

.subtitle {
  margin-top: 10px;
  font-size: 1rem;
  color: #475569;
}

.summary-card {
  flex-shrink: 0;
  min-width: 140px;
  padding: 16px 20px;
  border-radius: 16px;
  background: rgba(255, 255, 255, 0.85);
  border: 1px solid #dbe3ee;
  box-shadow: 0 8px 24px rgba(15, 23, 42, 0.06);
  text-align: center;
}

.summary-card strong {
  display: block;
  font-size: 2rem;
  line-height: 1;
  color: #0f172a;
  margin: 4px 0;
}

.summary-label {
  font-size: 0.9rem;
  color: #64748b;
}

.filters-panel,
.table-panel {
  background: rgba(255, 255, 255, 0.88);
  border: 1px solid #dbe3ee;
  border-radius: 20px;
  box-shadow: 0 10px 30px rgba(15, 23, 42, 0.06);
}

.filters-panel {
  padding: 24px;
}

.filters-header,
.table-topbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 16px;
  margin-bottom: 18px;
}

.filters-header h2,
.table-topbar h2 {
  margin: 0;
  font-size: 1.25rem;
  color: #0f172a;
}

.table-help {
  margin: 6px 0 0;
  color: #64748b;
  font-size: 0.95rem;
}

.controls {
  display: grid;
  grid-template-columns: repeat(3, minmax(180px, 1fr));
  gap: 18px;
}

.field {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.field label {
  font-size: 0.95rem;
  font-weight: 600;
  color: #334155;
}

select {
  width: 100%;
  min-height: 48px;
  padding: 0 14px;
  border-radius: 12px;
  border: 1px solid #cbd5e1;
  background: #fff;
  color: #0f172a;
  font-size: 0.95rem;
  outline: none;
  transition: border-color 0.2s ease, box-shadow 0.2s ease;
}

select:focus {
  border-color: #2563eb;
  box-shadow: 0 0 0 4px rgba(37, 99, 235, 0.12);
}

.clear-btn {
  border: none;
  background: #0f172a;
  color: white;
  padding: 11px 18px;
  min-height: 44px;
  border-radius: 12px;
  font-weight: 600;
  cursor: pointer;
  transition: transform 0.15s ease, opacity 0.15s ease;
}

.clear-btn:hover {
  opacity: 0.92;
}

.clear-btn:active {
  transform: translateY(1px);
}

.table-panel {
  padding: 24px;
}

.table-wrapper {
  width: 100%;
  overflow-x: auto;
  border-radius: 16px;
  border: 1px solid #e2e8f0;
  background: white;
}

table {
  width: 100%;
  min-width: 900px;
  border-collapse: collapse;
}

thead th {
  background: #f8fafc;
  color: #334155;
  font-size: 0.95rem;
  text-align: left;
  font-weight: 700;
  padding: 16px 18px;
  border-bottom: 1px solid #e2e8f0;
}

tbody td {
  padding: 16px 18px;
  border-bottom: 1px solid #eef2f7;
  color: #1e293b;
  vertical-align: middle;
}

tbody tr:hover {
  background: #f8fbff;
}

.sort-button {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 0;
  border: none;
  background: transparent;
  color: inherit;
  font: inherit;
  font-weight: 700;
  cursor: pointer;
}

.sort-icon {
  color: #2563eb;
  font-size: 0.95rem;
  min-width: 14px;
}

.pill {
  display: inline-flex;
  align-items: center;
  padding: 6px 10px;
  border-radius: 999px;
  background: #eff6ff;
  color: #1d4ed8;
  font-size: 0.88rem;
  font-weight: 600;
}

.status-message {
  margin: 16px 0 0;
  color: #475569;
}

.status-message.error {
  color: #b91c1c;
}

.empty-state {
  text-align: center;
  color: #64748b;
  padding: 28px 18px;
}

@media (max-width: 900px) {
  .page {
    padding: 24px 16px 40px;
  }

  .hero {
    flex-direction: column;
    align-items: stretch;
  }

  .summary-card {
    width: 100%;
  }

  .controls {
    grid-template-columns: 1fr;
  }

  .filters-header,
  .table-topbar {
    flex-direction: column;
    align-items: flex-start;
  }
}
</style>
