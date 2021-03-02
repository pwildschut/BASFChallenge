<template>
  <v-container>
    <v-row>
      <v-col>
        <v-autocomplete
          v-model="selectText"
          :items="items"
          :search-input.sync="search"
        >
        </v-autocomplete>
      </v-col>
      <v-col class="mt-8">
        <v-slider v-model="maxComplexity" label="Max complexity" min="0" :max="maxSeenComplexity" thumb-label="always"></v-slider>
        <v-slider v-model="minBondCount" label="Min normal bond count" min="0" max="1" step="0.001" thumb-label="always"></v-slider>
      </v-col>
    </v-row>
    <v-alert border="bottom" type="warning" v-show="displayComplexityAlert">
        The complexity is too high
    </v-alert>
    <v-alert border="bottom" type="warning" v-show="displayBondAlert">
        The bond count is too low
    </v-alert>
    <v-row v-if="!!this.compoundProperties">
      <v-col cols="2">
        <v-img :src="compoundImageSrc"></v-img>
      </v-col>
      <v-col>
        <p>Name: {{ this.compoundProperties.IUPACName }}</p>
        <p>InChl key: {{ this.compoundProperties.InChIKey }}</p>
        <p>Smiles: {{ this.compoundProperties.CanonicalSMILES }}</p>
        <p>Complexity: {{ this.compoundProperties.Complexity }}</p>
        <p>
          Normalized rotatable bound count:
          {{ this.compoundProperties.RotatableBondCount }} /
          {{ this.compoundProperties.ExactMass }} = {{ this.normalBoundCount }}
        </p>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
export default {
  name: 'ChemSelect',
  data: () => ({
    selectText: null,
    search: null,
    minBondCount: 0,
    maxComplexity: 0,
    maxSeenComplexity: 100,
    pubChemBaseUrl: 'https://pubchem.ncbi.nlm.nih.gov/rest',
    propertyList: [
      'IUPACName',
      'CanonicalSMILES',
      'InChIKey',
      'Complexity',
      'RotatableBondCount',
      'ExactMass'
    ]
  }),
  computed: {
    autocompleteUrl: function () {
      return this.search
        ? `${this.pubChemBaseUrl}/autocomplete/compound/${this.search}/json?limit=10`
        : null
    },
    compoundDataUrl: function () {
      return this.selectText
        ? `${this.pubChemBaseUrl}/pug/compound/name/${this.selectText}`
        : null
    },
    compoundPropertiesUrl: function () {
      return this.compoundDataUrl
        ? `${this.compoundDataUrl}/property/${this.propertyList.join(',')}/json`
        : null
    },
    compoundImageSrc: function () {
      return this.compoundDataUrl ? `${this.compoundDataUrl}/png` : null
    },
    items: function () {
      return this.autocompleteObj?.dictionary_terms?.compound ?? []
    },
    normalBoundCount: function () {
      if (this.compoundProperties) {
        return this.compoundProperties.RotatableBondCount /
              this.compoundProperties.ExactMass
      }
      return null
    },
    displayComplexityAlert: function () {
      return this.compoundProperties?.Complexity > this.maxComplexity
    },
    displayBondAlert: function () {
      return this.normalBoundCount < this.minBondCount
    }
  },
  asyncComputed: {
    autocompleteObj: {
      lazy: true,
      async get () {
        if (this.autocompleteUrl) {
          const response = await fetch(this.autocompleteUrl)
          return response.json()
        }
      }
    },
    compoundProperties: async function () {
      if (this.compoundPropertiesUrl) {
        const response = await fetch(this.compoundPropertiesUrl)
        const objJson = await response.json()
        const properties = objJson?.PropertyTable?.Properties[0]
        this.maxSeenComplexity = this.maxSeenComplexity < properties.Complexity ? properties.Complexity : this.maxSeenComplexity
        return properties
      }
    }
  }
}
</script>

<style>
</style>
