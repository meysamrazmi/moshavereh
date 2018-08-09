<template>
  <div id="app" class="theme--light application--is-rtl">
    <studyTable/>
  </div>
</template>

<script>
import studyTable from './components/studyTable.vue'

export default {
  name: 'app',
  components: {
    studyTable
  }
}
</script>

<style>
@font-face {
    font-family: 'Material Icons';
    font-style: normal;
    font-weight: 400;
    src: url(/fonts/material.woff2) format('woff2');
}
#app {
  font-family: iransans;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
body{
    margin: 0px;
    width: 100%;
	font-family: iransans !important;
}
#app {
    margin-top: 20px;
    min-width: 1000px;
}
.inner{
    max-width: 1100px;
    margin: auto;
}
label {
	font-size: 14px;
}
</style>
