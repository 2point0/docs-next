<template>
  <v-toolbar
    class="v-bar--underline"
    flat
  >
    <v-list-item class="pa-0 mx-n1">
      <vuetify-logo class="mr-4" />

      <v-list-item-content>
        <headline path="vuetify" />

        <latest-release />
      </v-list-item-content>

      <notifications />

      <advanced-toggle />
    </v-list-item>
  </v-toolbar>
</template>

<script>
  export default { name: 'DocumentationDrawerPrepend' }
</script>
