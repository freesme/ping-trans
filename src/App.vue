<template>
  <v-app>
    <v-main>
      <router-view/>
    </v-main>
  </v-app>
  <SpeedInsights/>
</template>

<script lang="ts" setup>
import {SpeedInsights} from '@vercel/speed-insights/vue';
</script>
