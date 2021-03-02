<template>
  <v-app>
    <v-main>
      <ChemSelect />
    </v-main>
  </v-app>
</template>

<script>
import ChemSelect from './components/ChemSelect'

export default {
  name: 'App',

  components: {
    ChemSelect
  },

  data: () => ({
    //
  })
}
</script>
