<template>
  <v-navigation-drawer
      v-model="drawer"
      app
      class="pt-4"
      color="grey"
      mini-variant
  >
    <router-link v-for="item in navItems" :to="item.to" :key="item.title">
      <v-avatar
          class="d-block text-center mx-auto"
          tile
      >
        <v-icon dark>{{ item.icon }}</v-icon>
      </v-avatar>
    </router-link>
  </v-navigation-drawer>
</template>

<script>
export default {
  name: 'SideNav',
  data: ()=>({
    drawer: null,
    navItems: [
      {
        title: 'Home',
        icon: 'fa-home',
        to: '/'
      },
      {
        title: 'Graph',
        icon: 'fa-code',
        to: '/graph'
      },
      {
        title: 'About',
        icon: 'fa-question',
        to: '/about'
      },
    ]
  })
}
</script>

<style lang="scss" scoped>
</style>
