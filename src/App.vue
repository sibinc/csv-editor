<template>
  <div id="app">
    <JsonEditor />
  </div>
</template>

<script>
import JsonEditor from './components/JsonEditor.vue';

export default {
  name: 'App',
  components: {
    JsonEditor,
  },
};
</script>

<style>
@import './assets/styles.css';
</style>