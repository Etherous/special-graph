<template>
  <v-app>
    <side-nav/>
    <v-main>
      <router-view/>
    </v-main>
  </v-app>
</template>

<script lang="ts">
import Vue from 'vue';
import SideNav from './components/SideNav.vue';

export default Vue.extend({
  name: 'SpecialGraphApp',
  components: {
    SideNav,
  },
});
</script>

<style type="scss">
@import "~jointjs/dist/joint.css";
@import "~vue-draggable-resizable/dist/VueDraggableResizable.css";
</style>
