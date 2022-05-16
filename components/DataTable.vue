<template lang="pug">
v-data-table(
  :headers="headers"
  :items="items"
  :loading="isLoading"
  item-key="id"
  :options.sync="options"
  :server-items-length="serverItemsLength"
  dense
).elevation-1.mt-10 {{ options }}
  template(#body.prepend)
    tr()
      td()
        v-btn(
          style="margin: 1rem 0"
          @click="clearFilters"
          :disabled="!isFiltersModified"
        ) Clear filters
      td()
        v-text-field(
          v-model="filters.user"
          label="Search"
          clearable
          @input="(value) => updateFilter('user', value)"
        )
      td()
        v-text-field(
          v-model="filters.email"
          label="Search"
          clearable
          @input="(value) => updateFilter('email', value)"
        )
      td()
        v-select(
          v-model="filters.genders"
          :items="filtersGendersList"
          label="Select"
          clearable
          small-chips
          deletable-chips
          chips
          multiple
          @input="(value) => updateFilter('genders', value)"
        )
      td()
        RangePicker(
          v-model="filters.years"
          @input="(value) => updateFilter('years', value)"
        )
      td()
        RangePicker(
          v-model="filters.sales"
          :value-formatter="salesRangeFormatter"
          @input="(value) => updateFilter('sales', value)"
        )
      td()
        v-autocomplete(
          v-model="filters.countries"
          :items="filtersCountriesList"
          label="Select"
          clearable
          small-chips
          deletable-chips
          chips
          multiple
          @input="(value) => updateFilter('countries', value)"
        )
  template(#item.user="{ item }")
    div() {{ userFullName(item.user) }}
</template>

<script>
import RangePicker from '@/components/DataTableFiltersRangePicker'

const emptyFilters = Object.freeze({
  user: '',
  email: '',
  genders: [],
  years: [],
  sales: [],
  countries: []
})

const defineIsEqual = (x, y) => {
  const ok = Object.keys,
    tx = typeof x,
    ty = typeof y
  return x && y && tx === 'object' && tx === ty
    ? ok(x).length === ok(y).length &&
        ok(x).every((key) => defineIsEqual(x[key], y[key]))
    : x === y
}

export default {
  components: { RangePicker },

  props: {
    items: { type: Array, required: true },
    allItems: { type: Array, required: true },
    headers: { type: Array, required: true },
    isLoading: { type: Boolean, default: false },
    serverItemsLength: { type: Number, default: 0 }
  },

  data: () => ({
    options: {},
    filters: { ...emptyFilters }
  }),

  watch: {
    options: {
      handler(value) {
        this.$emit('update-options', value)
      },
      deep: true
    }
  },

  computed: {
    isFiltersModified() {
      return !defineIsEqual(this.filters, this.defaultFilters)
    },

    filtersGendersList() {
      const genders = this.allItems.map((item) => item.gender)
      const uniqGenders = [...new Set(genders)]

      return uniqGenders
    },

    filtersYearsRange() {
      const range = this.allItems.map((item) => item.year).sort((a, b) => a - b)

      return [range[0], range.at(-1)]
    },

    filtersSalesRange() {
      const range = this.allItems
        .map((item) => Number(item.sales))
        .sort((a, b) => a - b)
        .map((item, index) => (index ? Math.ceil(item) : Math.floor(item)))

      return [range[0], range.at(-1)]
    },

    filtersCountriesList() {
      const countries = this.allItems.map((item) => item.country)
      const uniqCountries = [...new Set(countries)]

      return uniqCountries.sort()
    }
  },

  created() {
    this.filters.years = this.filtersYearsRange
    this.filters.sales = this.filtersSalesRange

    this.defaultFilters = { ...emptyFilters, ...this.filters }
  },

  methods: {
    userFullName(user) {
      return `${user.title} ${user.first_name} ${user.last_name}`
    },

    salesRangeFormatter(range) {
      range = range.map((item) => {
        const bits = item.toLocaleString('en').split(',')
        const prefix = bits[0]
        const suffix = new Array(bits.length - 1).fill('k').join('')
        const formatted = [prefix, suffix].join('')

        return formatted
      })

      return range.join(' – ')
    },

    clearFilters() {
      this.filters = { ...this.defaultFilters }
      this.$emit('update-filters', this.filters)
    },

    updateFilter(key, toUpdate) {
      this.$emit('update-filters', { ...this.filters, ...{ [key]: toUpdate } })
    }
  }
}
</script>

<style lang="sass" scoped>
.v-data-table
  max-width: 100%
</style>
