<template>
  <v-app-bar app color="white" light elevation="1">
    <p class="mr-3">  </p>
    <img src="/icon.png" alt="App Icon" height="30" class="mr-2" />
    <div class="text-h6 font-weight-bold">전세, 월세 뭐가 더 이득일까?</div>
    <v-spacer></v-spacer>
    <!-- Future buttons can go here -->
  </v-app-bar>
</template>

<script>
export default {
  name: 'TheHeader',
}
</script>

<style scoped>
.v-toolbar__title {
  font-size: 1.25rem;
}
</style>