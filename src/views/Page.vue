<template>
  <v-container
    tag="section"
    class="px-4 px-sm-8 px-md-12"
  >
    <v-responsive
      class="mx-auto overflow-visible"
      max-width="868"
    >
      <component :is="component" />
    </v-responsive>
  </v-container>
</template>

<script>
  // Utilities
  import { genMetaData } from '@/util/metadata'
  import { sync } from 'vuex-pathify'

  // This should only be extended by other pages
  export default {
    name: 'Page',

    metaInfo () {
      // Check it fm exists
      if (!this.frontmatter) return {}

      // Check if meta exists
      const { meta } = this.frontmatter

      if (!meta) return

      const {
        description = '',
        keywords = '',
        title = '',
      } = meta

      return genMetaData(
        title,
        description,
        keywords,
      )
    },

    data: () => ({ component: undefined }),

    computed: {
      frontmatter: sync('pages/frontmatter'),
      toc: sync('pages/toc'),
    },

    watch: { '$route.params.locale': 'update' },

    created () {
      this.update()
    },

    methods: {
      async load () {
        console.error('Missing load method for Page.vue')
      },
      async update () {
        const {
          attributes = {},
          toc = [],
          vue = {},
        } = await this.load(this.$route)

        this.component = vue.component
        this.frontmatter = attributes
        this.toc = toc
      },
    },
  }
</script>
