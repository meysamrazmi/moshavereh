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
#app {
  font-family: iransans;
  padding-bottom: 50px;
  margin-top: 20px;
  min-width: 1000px;
}
#app .inner{
  max-width: 1100px;
  margin: auto;
}
#app label {
	font-size: 14px;
}
.v-select__selections input[type="text"] {
	visibility: hidden;
}
hr.v-divider {
	margin: 0;
}
.v-input__slot input, .v-input__slot input:focus, .v-input__slot input:hover {
	margin: 0;
	border: none !important;
	box-shadow: none !important;
}
.v-input__slot input[disabled]{
  background: transparent;
}
.v-input__slot textarea{
  outline: none !important;
  box-shadow: none !important;
}
.cancel-btn.v-btn {
	color: #555 !important;
  box-shadow: none;
}
.cancel-btn.v-btn:hover {
	background: #eee !important;
}
.description-area .v-btn {
	background: transparent;
}
.table-data .cancel-btn.v-btn:hover {
	background: #ff5252 !important;
}
.table-data .cancel-btn.v-btn {
	color: #fff !important;
}
</style>
