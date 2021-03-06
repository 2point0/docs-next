<template>
  <v-text-field
    id="doc-search"
    ref="search"
    v-model="search"
    :background-color="(!theme.isDark && !isFocused) ? 'grey lighten-3' : undefined"
    :class="isSearching ? 'rounded-b-0' : ''"
    :flat="!isFocused && !isSearching"
    :placeholder="placeholder"
    class="doc-search mx-auto mx-md-4"
    dense
    hide-details
    solo
    @blur="onBlur"
    @clear="resetSearch"
    @focus="onFocus"
    @keydown.esc="onEsc"
  >
    <template #prepend-inner>
      <v-icon
        :color="!isFocused ? 'grey' : undefined"
        class="mr-2"
      >
        $mdiVuetify
      </v-icon>
    </template>
  </v-text-field>
</template>

<script>
  // This behavior should be easier to do with solo fields
  // TODO: Review this for v3
  export default {
    name: 'DocumentationSearch',

    inject: ['theme'],

    data: () => ({
      docSearch: {},
      isFocused: false,
      isSearching: false,
      search: '',
      timeout: null,
    }),

    computed: {
      placeholder () {
        return !this.isFocused
          ? 'Search Vuetify (press \'/\' to focus)'
          : ''
      },
    },

    watch: {
      isSearching (val) {
        if (val) {
          this.$refs.search.focus()

          return
        }

        this.resetSearch()
      },
      search (val) {
        this.isSearching = Boolean(val && val.length)

        if (val) return

        this.docSearch.autocomplete.autocomplete.close()
        this.docSearch.autocomplete.autocomplete.setVal('')
      },
    },

    mounted () {
      document.onkeydown = e => {
        e = e || window.event

        if (
          e.key === '/' &&
          e.target !== this.$refs.search.$refs.input
        ) {
          e.preventDefault()

          this.$refs.search.focus()
        }
      }

      // eslint-disable-next-line no-unused-expressions
      import(
        /* webpackChunkName: "docsearch" */
        'docsearch.js/dist/cdn/docsearch.min.css'
      )
      import(
        /* webpackChunkName: "docsearch" */
        'docsearch.js'
      ).then(this.init)
    },

    beforeDestroy () {
      document.onkeydown = null

      this.docSearch.autocomplete.autocomplete.close()
      this.docSearch.autocomplete.autocomplete.setVal('')
    },

    methods: {
      async init ({ default: docsearch }) {
        const vm = this

        this.docSearch = docsearch({
          apiKey: '259d4615e283a1bbaa3313b4eff7881c',
          autocompleteOptions: {
            appendTo: '#default-app-bar',
            autoselect: true,
            clearOnSelected: true,
            hint: false,
            debug: process.env.NODE_ENV === 'development',
          },
          handleSelected (input, event, suggestion) {
            vm.$router.push(suggestion.url.split('.com').pop())
            vm.resetSearch(400)
          },
          indexName: 'vuetifyjs',
          inputSelector: '#doc-search',
        })

        const { search } = this.$route.query

        if (!search) return

        this.search = search
        this.$refs.search.focus()

        await this.$nextTick()

        // Dispatch an event to trigger agolia search menu
        const event = new Event('input')

        this.$refs.search.$refs.input.dispatchEvent(event)
      },
      onBlur () {
        this.resetSearch()
      },
      onEsc () {
        this.$refs.search.blur()
      },
      onFocus () {
        clearTimeout(this.timeout)

        this.isFocused = true
      },
      resetSearch (timeout = 0) {
        clearTimeout(this.timeout)

        this.$nextTick(() => {
          this.search = undefined
          this.isSearching = false

          this.timeout = setTimeout(() => (this.isFocused = false), timeout)
        })
      },
    },
  }
</script>

<style lang="sass">
  @import '~vuetify/src/styles/settings/_elevations.scss'
  @import '~vuetify/src/styles/tools/_elevation.sass'

  .algolia-autocomplete
    // flex: 1 1 auto
    // position: fixed !important

  #app
    .algolia-docsearch-suggestion--title
      margin-bottom: 0

    .algolia-autocomplete

      .ds-dropdown-menu
        clip-path: inset(0px -12px -12px -12px) !important
        box-shadow: 0px 1px 4px 0px rgba(32, 33, 36, 0.28) !important
        border-radius: 0 0 16px 16px
        left: -48px !important
        top: 0px !important
        min-width: calc(100% + 60px)

        [class^=ds-dataset-]
          border-radius: 0 0 16px 16px
          border-color: transparent

        &:before
          display: none

      a
        text-decoration: none !important
    .doc-search
      border-radius: 16px

    &.theme--dark
      color: white

      [class^=ds-dataset-],
      .algolia-docsearch-suggestion
        background: #1E1E1E !important

        .algolia-docsearch-suggestion--highlight
          color: #2196f3

        .algolia-docsearch-suggestion--title
          color: #FFFFFF

        .algolia-docsearch-suggestion--category-header
          color: #a4a7ae
</style>
