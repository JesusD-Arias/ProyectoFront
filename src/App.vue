<template>
  <v-app>
    <v-row justify="center" no-gutters>
      <v-col
        cols="12"
        sm="12"
        md="8"
      >
        <Customers />
      </v-col>
    </v-row>
  </v-app>
</template>

<script>
import Customers from './components/Customers';

export default {
  name: 'App',

  components: {
    Customers,
  },

  data: () => ({
    
  }),
};
</script>
