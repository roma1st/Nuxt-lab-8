<script setup lang="ts">
import { useRoute } from 'vue-router'

const route = useRoute()
const id = Number(route.params.id)

const links = [{
  label: 'BlogPostsUI',
  to: '/posts/'
},{
  label: 'Categories',
  to: '/categories/'
}]
</script>
<template>
  <UHorizontalNavigation :links="links">
    <template #default="{ link }">
      <span class="group-hover:text-primary relative">{{link.label}}</span>
    </template>
  </UHorizontalNavigation>
  <RouterView />
</template>

<style>

</style>
