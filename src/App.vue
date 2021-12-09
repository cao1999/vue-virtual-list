<template>
  <virtual-list :list="list" />
</template>

<script setup lang="ts">
import VirtualList from "./components/virtucal-list.vue";

const list: any[] = [];

for (let i = 0; i < 100; i++) {
  list.push(i);
}
</script>

<style></style>
