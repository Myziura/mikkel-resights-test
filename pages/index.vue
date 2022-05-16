<template lang="pug">
v-container
  v-row
    v-col(cols)
      DataTable(
        :items="items"
        :all-items="sales"
        :is-loading="isLoading"
        :headers="headers"
        :server-items-length="filteredSales.length"
        @update-options="handleUpdateOptions"
        @update-filters="handleUpdateFiltersAndFetchData"
      )
</template>

<script>
import DataTable from '~/components/DataTable.vue'
import sales from '~/api/sales.js'

const delay = (ms) => new Promise((resolve) => setTimeout(resolve, ms))

export default {
  components: { DataTable },

  data: () => ({
    items: [],
    filteredSales: [],
    isLoading: false
  }),

  computed: {
    headers() {
      return [
        { text: '', value: 'id' },
        { text: 'Name', value: 'user', align: 'start', width: '300px' },
        { text: 'Email', value: 'email', align: 'start', width: '300px' },
        { text: 'Gender', value: 'gender', align: 'start', width: '250px' },
        { text: 'Year', value: 'year', align: 'start', width: '100px' },
        { text: 'Sales', value: 'sales', align: 'start', width: '200px' },
        { text: 'Country', value: 'country', width: '250px' }
      ]
    },

    filters() {
      return {
        user: (value) =>
          this.filteredSales.filter((item) => {
            const search = value.toLowerCase()

            const defineFullName = (user) => {
              const { title, first_name, last_name } = user
              const fullName = [title, first_name, last_name].join(' ')
              return fullName.toLowerCase()
            }

            const userFullName = defineFullName(item.user)

            return userFullName.includes(search)
          }),
        email: (value) =>
          this.filteredSales.filter((item) => {
            const search = value.toLowerCase()
            const email = item.email.toLowerCase()

            return email.includes(search)
          }),
        genders: (value) =>
          this.filteredSales.filter(({ gender }) => value.includes(gender)),
        years: ([min, max]) =>
          this.filteredSales.filter(({ year }) => year >= min && year <= max),
        sales: ([min, max]) =>
          this.filteredSales.filter(
            ({ sales }) => sales >= min && sales <= max
          ),
        countries: (value) =>
          this.filteredSales.filter(({ country }) => value.includes(country))
      }
    }
  },

  async created() {
    this.sales = this.filteredSales = sales.results
  },

  methods: {
    async handleUpdateOptions(options) {
      this.options = options || this.options
      const { page, itemsPerPage } = this.options
      this.items = await this.fetchData(page - 1, itemsPerPage)
    },

    async fetchData(page, size) {
      const start = page * size

      this.isLoading = true

      await delay(1000)

      this.isLoading = false

      return this.filteredSales.slice(start, start + size)
    },

    handleUpdateFiltersAndFetchData(filters) {
      this.updateFilters(filters)
      this.handleUpdateOptions()
    },

    updateFilters(filters) {
      this.filteredSales = this.sales

      if (filters.user) {
        this.filteredSales = this.filters.user(filters.user)
      }

      if (filters.email) {
        this.filteredSales = this.filters.email(filters.email)
      }

      if (filters.genders.length) {
        this.filteredSales = this.filters.genders(filters.genders)
      }

      this.filteredSales = this.filters.years(filters.years)
      this.filteredSales = this.filters.sales(filters.sales)

      if (filters.countries.length) {
        this.filteredSales = this.filters.countries(filters.countries)
      }
    }
  }
}
</script>

<style lang="sass" scoped>
.v-progress-circular
  position: absolute
  top: 50%
  left: 50%
</style>
