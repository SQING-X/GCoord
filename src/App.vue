<template>
  <div id="app">
    <gcoord-view></gcoord-view>
  </div>
</template>

<script>
import gcoordView from './components/gcoord'
export default {
  name: 'App',
  components: {
    gcoordView
  }
}
</script>

<style>
html,body{
  height:100% ;

}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  height: 100%;
}
</style>
