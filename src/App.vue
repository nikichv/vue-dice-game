<template>
  <div id="app">
    <game/>
  </div>
</template>

<script>
import game from '@/components/blocks/game';

export default {
  name: 'App',
  components: {
    game,
  },
};
</script>

<style lang="scss">
  @import "@/assets/styles/main.scss";
</style>
