<script setup lang="ts">
import { useRoute } from 'vue-router';

const route = useRoute();
const endStation = route.query.endStation;
const endStationLine = route.query.endStationLine;

</script>

<template>
  <p> {{ endStation }} + {{ endStationLine }}</p>
</template>