<template>
  <div class="vue-type-ahead">
    <VueSelect
      :open="finalOpen"
      @update:open="val => open = val"
      trigger="manual"
      v-model="valueModel"
      :auto-hide="!focused"
      no-popover-focus
      @popover-mousedown="onPopoverContentMousedown"
      @popover-mouseup="onPopoverContentMouseup"
    >
      <VueInput
        slot="trigger"
        v-bind="$attrs"
        v-model="tempValueModel"
        :loading-right="loading"
        :suggestion="suggestion"
        @focus="onFocus"
        @blur="onBlur"
      />

      <template v-if="shownSuggestions.length">
        <VueSelectButton
          v-for="(suggestion, index) of shownSuggestions.slice(0, showMax)"
          :key="index"
          :icon-left="suggestion.icon"
          :value="suggestion.value"
        >
          {{ suggestion.value }}
        </VueSelectButton>
      </template>
    </VueSelect>
  </div>
</template>

<script>
export default {
  name: 'VueTypeAhead',

  inheritAttrs: false,

  props: {
    loading: {
      type: Boolean,
      default: false,
    },

    restrictChoice: {
      type: Boolean,
      default: false,
    },

    showAll: {
      type: Boolean,
      default: false,
    },

    showMax: {
      type: [String, Number],
      default: 10,
    },

    suggestions: {
      type: Array,
      required: true,
    },

    value: {
      default: '',
    },
  },

  data () {
    return {
      directSelect: false,
      dirty: false,
      focused: false,
      open: false,
      tempValue: this.value,
      lastSuggestions: this.suggestions,
    }
  },

  computed: {
    filteredSuggestions () {
      const value = this.tempValue
      if (value) {
        return this.suggestions.filter(
          s => s.value && s.value.startsWith(value)
        )
      } else if (this.showAll) {
        return this.suggestions
      } else {
        return []
      }
    },

    finalOpen () {
      return !!(this.open && this.shownSuggestions.length && (
        this.shownSuggestions.length !== 1 ||
        this.shownSuggestions[0].value !== this.value
      ))
    },

    shownSuggestions () {
      if (this.open) {
        return this.filteredSuggestions
      } else {
        return this.lastSuggestions
      }
    },

    suggestion () {
      if (this.shownSuggestions.length) {
        return this.shownSuggestions[0].value
      }
    },

    tempValueModel: {
      get () {
        return this.tempValue
      },
      set (value) {
        this.tempValue = value
        this.dirty = true
        if (value && !this.open) {
          this.showSuggestions()
        }
      },
    },

    valueModel: {
      get () {
        return this.value
      },
      set (value) {
        this.directSelect = true
        this.tempValue = value
        console.log(value)
        this.$emit('input', value)
      },
    },
  },

  watch: {
    filteredSuggestions (value) {
      if (this.open) {
        this.lastSuggestions = value
      }
    },

    value (value, oldValue) {
      if (value !== oldValue && value !== this.tempValue) {
        this.tempValue = value
      }
    },
  },

  methods: {
    getFinalChoice (value) {
      if (this.restrictChoice) {
        if (this.suggestions.find(
          s => s.value === value
        )) {
          return value
        } else {
          return ''
        }
      } else {
        return value
      }
    },

    async onBlur () {
      await this.$nextTick()

      if (!this.$_popoverMousedown) {
        this.focused = false
        this.open = false
        if (!this.directSelect) {
          // Get final value depending on restrictChoice
          let value = this.getFinalChoice(this.tempValue)
          // Reset to initial value
          if (this.restrictChoice && !value) {
            value = this.getFinalChoice(this.value)
          }
          // Emit new value
          if (this.dirty && value !== this.value) {
            this.$emit('input', value)
            // Reset temp value
            this.tempValue = value
          } else {
            // Reset temp value
            this.tempValue = this.value
          }
        }
      }
    },

    async onFocus () {
      this.focused = true
      this.directSelect = false
      this.dirty = false
      this.tempValue = ''
      this.$_popoverMousedown = false

      // Suggestions
      if (this.value || this.showAll) {
        this.showSuggestions()
      }
    },

    onPopoverContentMousedown (event) {
      this.$_popoverMousedown = true
    },

    async onPopoverContentMouseup (event) {
      this.$_popoverMousedown = false
      setTimeout(() => {
        this.onBlur()
      }, 100)
    },

    showSuggestions () {
      this.open = true
    },
  },
}
</script>

<style lang="stylus">
@import "../style/imports"

.vue-type-ahead
  display inline-block
  vertical-align middle
</style>