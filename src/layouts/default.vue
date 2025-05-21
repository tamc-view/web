<template>
  <v-main>
    <Appbar theme="dark"/>
    <router-view />
    <AppFooter theme="dark"/>
  </v-main>

  <!-- <AppFooter /> -->
</template>

<script setup>
import Appbar from '@/components/Appbar.vue';
import AppFooter from '@/components/AppFooter.vue';

  //
</script>
