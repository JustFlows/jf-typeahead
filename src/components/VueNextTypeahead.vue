<template>
  <span
    class="vue-next-typeahead"
    :class="[className]">
    <span
      v-if="title && showTitle"
      class="vue-next-typeahead__title">
      {{ title }}
    </span>
    <div
      class="vue-next-typeahead__wrapper-input"
      :class="{
        'vue-next-typeahead__wrapper-input--focusing': focusing
      }"
      :style="{
        'background': searching || loadingParentComponent ? '#FAFAFA' : '#FFFFFF'
      }">
      <input
        type="text"
        autocomplete="false"
        :class="{ 'is-invalid': feedback }"
        :name="formName"
        :placeholder="placeholder"
        :value="value"
        @input="changeInput($event)"
        @paste="changeInput($event)"
        @focus="onFocusin()"
        @blur="onFocusout()" />
      <i class="ico-search" v-if="!searching && !loadingParentComponent" />
      <i class="ico-loader" v-else />
    </div>
    <ul
      v-if="value.length && !searching && !loadingParentComponent && showDropdown"
      class="vue-next-typeahead__dropdown">
      <template v-if="items.length">
        <li
          v-for="(item, ix) in items"
          :key="ix"
          @click="clickItem(item)">
          <span v-html="keyItem ? highlight(item[keyItem], value) : item.label" />
        </li>
      </template>
      <li v-else class="li-no-results">
        <slot name="no-results">{{ noResultsText }}</slot>
      </li>
    </ul>
    <br>
    <div class="invalid-feedback">{{feedback}}</div>

  </span>
</template>

<script>
import { useField } from 'vee-validate'
export default {
  name: 'VueNextTypeahead',
  props: {
    asyncFunc: {
      type: Function
    },
    feedback: {
      type: String
    },
    formName: {
      type: String
    },
    modelValue: {
      type: String
    },
    title: {
      type: String,
      default: null
    },
    className: {
      type: String,
      default: ''
    },
    items: {
      type: Array,
      default: () => []
    },
    keyItem: {
      type: String,
      default: null
    },
    showTitle: {
      type: Boolean,
      default: true
    },
    placeholder: {
      type: String,
      default: 'Qué necesitas...'
    },
    noResultsText: {
      type: String,
      default: 'No existen resultados para esa búsqueda...'
    },
    minChars: {
      type: Number,
      default: 2,
      validator: (value) => {
        return value > 0
      }
    },
    delay: {
      type: Number,
      default: 500
    },
    loadingParentComponent: {
      type: Boolean,
      default: false
    }
  },
  data () {
    const { value, handleChange } = useField(this.formName ?? '', undefined, {
      initialValue: this.modelValue
    })
    return {
      searching: false,
      focusing: false,
      showDropdown: true,
      timeout: '',
      handleChange,
      value
    }
  },
  mounted () {
    // Reset component when click outside.
    document.addEventListener('click', (e) => {
      if (!this.$el.contains(e.target)) {
        this.reset()
      }
    })
  },
  methods: {

    /**
     * Main method to make the query of the URL received as a prop.
     * The parent component is in charge of processing that first response.
     */
    search () {
      try {
        this.searching = true

        clearTimeout(this.timeout)

        this.timeout = setTimeout(async () => { // Begin timeout
          const response = await this.asyncFunc()
          this.$emit('response', response)
          clearTimeout(this.timeout) // Clean interval
          this.showDropdown = true
          this.searching = false
        }, this.delay)
      } catch (error) {
        this.$emit('error', error)
        this.searching = false
        console.error('Check your API URL you have introduced.')
        throw error
      }
    },

    /**
     * Emit value input each time you write.
     * It is only triggered if it has a value and if it meets the minimum number of characters.
     * When you write here the search event is triggered.
     */
    changeInput (event) {
      this.handleChange(event)
      if (event.target && event.target.value && event.target.value.length >= this.minChars) {
        this.search()
      }
      this.$emit('update:modelValue', event.target.value)
      if (event.target && event.target.value.length === 0) {
        this.reset()
      }
    },

    /**
     * When active focus input.
     */
    onFocusin () {
      this.focusing = true
    },

    /**
     * When remove focus input.
     */
    onFocusout () {
      this.focusing = false
      this.searching = false
    },

    /**
     * Emit event when select one option in list.
     */
    clickItem (item) {
      this.$emit('hit', item)
      this.value = item.label
      this.reset()
    },

    /**
     * Highlight in bold the text that matches our search.
     * Case insensitive.
     */
    highlight (text, term) {
      if (text && term) {
        var index = text.toLowerCase().indexOf(term.toLowerCase())
        if (index >= 0) {
          text = `${text.substring(0, index)}<span style="font-weight: 600;">${text.substring(index, index + term.length)}</span>${text.substring(index + term.length)}`
        }
      }
      return text
    },

    reset () {
      this.showDropdown = false
      this.searching = false
      // this.$emit('reset')
      // this.$emit('update:input', '')
    }
  }
}
</script>

<style scoped lang="scss">
  .vue-next-typeahead {
    display: block;
    position: relative;

    &__title {
      font-size: 14px;
      font-style: normal;
      font-weight: 600;
      line-height: 22px;
      letter-spacing: 0px;
      text-align: left;
      display: block;
      margin-bottom: 8px;
    }

    &__wrapper-input {
      height: 48px;
      background: #FFFFFF;
      border: 1px solid #D2D2D2;
      box-sizing: border-box;
      border-radius: 4px;
      padding: 0 15px 0 16px;
      position: relative;

      > input {
        height: 48px;
        border-radius: 0;
        border: 0;
        outline: none;
        font-size: 14px;
        font-style: normal;
        font-weight: 400;
        line-height: 22px;
        letter-spacing: 0px;
        text-align: left;
        margin-right: 8px;
        width: calc(100% - 32px);
        padding: 0;
        max-height: 100%;
        color: #1e2022;
        background: transparent;

        &::placeholder {
          color: #D2D2D2;
        }
      }

      .ico-search,
      .ico-loader {
        vertical-align: middle;
        background-size: cover;
        display: inline-block;
        width: 24px;
        height: 24px;
      }
      .ico-search {
        background-image: url('../assets/search.svg');
      }
      .ico-loader {
        background-image: url('../assets/loader-orange.svg');
        -webkit-animation:spin 4s linear infinite;
        -moz-animation:spin 4s linear infinite;
        animation:spin 4s linear infinite;
      }
      @-moz-keyframes spin { 100% { -moz-transform: rotate(360deg); } }
      @-webkit-keyframes spin { 100% { -webkit-transform: rotate(360deg); } }
      @keyframes spin { 100% { -webkit-transform: rotate(360deg); transform:rotate(360deg); } }

      // Hover state
      &:hover:not(.vue-next-typeahead__wrapper-input--focusing) {
        background: #FAFAFA;
        border: 1px solid #9B9B9B;

        > input::placeholder {
          color: #9B9B9B;
        }
      }

      // Focus state
      &--focusing {
          box-shadow: 0px 0px 8px 0px #0271A9 20%;
          border: 1px solid #0271A9;
          box-sizing: border-box;
          box-shadow: 0px 0px 8px rgba(105, 103, 229, 0.2);
          .ico-search {
            background-image: url('../assets/search-orange.svg');
          }
      }
    }

    &__dropdown {
      border: 1px solid #D2D2D2;
      background-color: #FFFFFF;
      list-style: none;
      width: calc(100% - 2px);
      padding: 16px 0 0;
      margin: -14px 0 0 0;
      max-height: 336px;
      overflow-y: auto;

      > li {
        min-height: 42px;
        padding: 0 16px;
        font-size: 14px;
        font-style: normal;
        letter-spacing: 0px;
        text-align: left;
        cursor: pointer;

        &:hover {
          background: #FAFAFA;
        }

        > span {
          line-height: 42px;
        }

        &.li-no-results {
          line-height: 42px;
          cursor: auto;
        }
      }

      /** SCSS Para el scroll */
      &::-webkit-scrollbar {
          -webkit-appearance: none;
          width: 5px;
          height: 6px;
      }

      &::-webkit-scrollbar-thumb {
          border-radius: 12px;
          border: 4px solid rgba(255, 255, 255, 0);
          background-color: #bdbdbd;
      }

      &::-webkit-scrollbar-corner {
          background-color: transparent;
      }
    }

    // Responsive
    @media screen and (max-width: 768px) {
      width: 100%;
    }
  }
</style>
