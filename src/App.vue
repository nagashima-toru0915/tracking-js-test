<template>
  <v-app>
    <v-main>
      <TestTracking/>
    </v-main>
  </v-app>
</template>

<script>
import TestTracking from './components/TestTracking.vue'

export default {
  name: 'App',

  components: {
    TestTracking,
  },

  data: () => ({
    //
  }),
}
</script>
