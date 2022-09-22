<template>
  <v-app>
    <v-main>
      <DevForm/>
    </v-main>
  </v-app>
</template>

<script>
import DevForm from './components/DevForm';

export default {
  name: 'App',

  components: {
    DevForm,
  },

  data: () => ({
    //
  }),
};
</script>
