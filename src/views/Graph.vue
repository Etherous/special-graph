<template>
  <div class="graph-view">
    <graph/>
  </div>
</template>

<script>
import Graph from "@/components/graph/Graph";

export default {
  name: 'SpecialGraphApp',
  components: {
    Graph
  },
}
</script>

<style lang="scss" scoped>
.graph-view {
  //position:relative;
  width: 100%;
  height: 100%;
}
</style>
