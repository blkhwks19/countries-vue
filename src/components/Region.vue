<template>
  <div>
    <v-list two-line>
      <v-list-item
        v-for="(country, index) in countries"
        :key="index"
        @click="handleClick(country)"
      >
        <v-list-item-avatar tile>
          <v-img contain :src="country.flag"></v-img>
        </v-list-item-avatar>

        <v-list-item-content>
          <v-list-item-title v-html="country.name"></v-list-item-title>
          <v-list-item-subtitle v-html="country.subregion"></v-list-item-subtitle>
        </v-list-item-content>

        <v-list-item-action>
          <v-icon>mdi-chevron-right</v-icon>
        </v-list-item-action>
      </v-list-item>
    </v-list>

  </div>
</template>

<script>
export default {
  name: 'Region',

  props: ['region'],

  data() {
    return {
      countries: [],
    }
  },

  created() {
    this.getCountries();
  },

  methods: {
    getCountries() {
      fetch(`https://restcountries.eu/rest/v2/region/${this.region}`)
        .then(res => res.json())
        .then(data => this.countries = data)
        .catch(err => {
          console.log('ERROR GETTING COUNTRIES BY REGION');
          console.log(err.message)
        });
    },

    handleClick(country) {
      this.$emit('showDialog', country);
    }
  }
  
}
</script>