<script setup lang="ts">
import Product from "~/components/common/Product.vue";
import { useAuthStore } from "~/stores/auth";

const auth = useAuthStore();

const search = computed(() => auth.$state.search);
</script>

<template>
  <product
    advertising="https://im.uniqlo.com/global-cms/spa/res590371311de4ffd98f31915ef71d0202fr.jpg"
    :search="search"
  >
    {{ search }}
  </product>
</template>

<style lang="scss"></style>
