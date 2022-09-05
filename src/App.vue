<template>
  <div id="app">
    <SvgWrapper src="map.svg" isAdmin="true" />
  </div>
</template>

<script>
//import SvgWrapper from './components/SvgWrapper.vue';
import SvgWrapper from './components/SvgWrapper.vue';

export default {
  name: 'App',
  components: {
    SvgWrapper,
  },
};
</script>
