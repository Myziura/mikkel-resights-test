<template lang="pug">
v-menu(
  v-model="isVisible"
  nudge-bottom="16"
  :close-on-content-click="false"
  offset-y
  auto
  min-width="600"
)
  template(#activator="{ attrs, on }")
    v-btn(v-bind="attrs" v-on="on") {{ buttonValue }}

  v-card()
    v-subheader() Select range
    v-card-text()
      v-row()
        v-col(class="px-4")
          v-range-slider(
            :value="value"
            :max="max"
            :min="min"
            hide-details
            class="align-center"
            @input="handleInputRange"
          )
            template(#prepend)
              v-text-field(
                :value="value[0]"
                class="mt-0 pt-0"
                hide-details
                single-line
                type="number"
                style="width: 100px"
                @change="(min) => handleInputRange([+min, value[1]])"
              )
            template(#append)
              v-text-field(
                :value="value[1]"
                class="mt-0 pt-0"
                hide-details
                single-line
                type="number"
                style="width: 100px"
                @change="(max) => handleInputRange([value[0], +max])"
              )
</template>

<script>
export default {
  props: {
    value: {
      type: Array,
      validator: (value) => value.filter(Number).length === 2
    },
    valueFormatter: { type: Function, default: () => null }
  },

  data: () => ({
    isVisible: false
  }),

  created() {
    this.min = this.value[0]
    this.max = this.value[1]
  },

  computed: {
    buttonValue() {
      return this.valueFormatter(this.value) || this.value.join(' – ')
    }
  },

  methods: {
    handleInputRange(range) {
      this.$emit('input', range)
    }
  }
}
</script>
