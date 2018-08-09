<template>
<div>
  <v-container>
    <v-layout column>
      <v-flex xs2 offset-xs10>
        <v-select
          v-model="selectedWeek"
          :items="weeks"
          label="انتخاب هفته"
          @change="update()"
        ></v-select>
      </v-flex>

    <v-flex class="table-data" xs12 light="true">

      <!-- header -->
      <v-layout row wrap>
        <v-flex v-for="item in headers" :key="item.value" :class="item.width | xs_">
          <span style="padding-bottom: 10px;display: inline-block;">{{item.text}}</span>
        </v-flex>
      </v-layout>
      <v-divider></v-divider>
      <v-divider></v-divider>
      <v-progress-linear 
        v-if="loading"
        style="	margin: -2px 0 2px;"
        :indeterminate="true" 
        color="primary" 
        height="3"
        ></v-progress-linear>
      
      <!-- rows -->
      <div v-for="(item, cid) in courses" :key="item">
        <v-layout align-center justify-start class="text-xs-right">
          <v-flex style="padding: 10px;" xs2>{{item}}</v-flex>

          <v-flex 
            v-for="(row, did) in rows[cid]" 
            :key="row.key" 
            :class="row.width | xs_"
            @click="toggle(cid, did, $event)"
            class="text-xs-center">
            
            <!-- data -->
            <span v-if="rowsModelEnable[cid][did]" >{{rowsModel[cid][did]}}</span>
            <!-- @click="setFocus($event)" -->
            <!-- input -->
            <v-text-field
              v-model="rowsModel[cid][did]"
              solo
              class="text-xs-center"
              :data-id="'' + cid + did"
              v-if="!rowsModelEnable[cid][did]"
              @keyup.native.enter="setfield(cid, did, $event.target.value)"
              :loading="inputLoading"
              :disabled="inputLoading">
              <v-progress-linear
                v-model="inputLoading"
                :indeterminate="true" 
                color="primary" 
                height="3"
                ></v-progress-linear>         
            </v-text-field>
            
          </v-flex>
        </v-layout>
        <v-divider></v-divider>
      </div>

    </v-flex>

    <v-flex xs12 class="student-desc" @click="toggle(6, 0, $event)">
      <v-subheader >
        توضیحات تکمیلی این هفته
      </v-subheader>
      <div class="desc-data text-xs-right" v-if="rowsModelEnable[6][0]">{{rowsModel[6][0]}}</div>
      
      <v-textarea
        label="توضیحات تکمیلی این هفته"
        v-model="rowsModel[6][0]"
        hint="توضیحات مربوط به این هفته ی خود را در این قسمت برای مشاور بنویسید."
        solo
        v-if="!rowsModelEnable[6][0]"
        :loading="inputLoading"
        :disabled="inputLoading"        
      >
        <v-progress-linear
          v-model="inputLoading"
          :indeterminate="true" 
          color="primary" 
          height="3"
          ></v-progress-linear>     
      </v-textarea>
      
      <v-btn 
        color="info" small absolute
        @click="setfield(6, 0, $event.target.parentNode.parentNode.getElementsByTagName('textarea')[0].value)"
        v-if="!rowsModelEnable[6][0]"
      >ذخیره</v-btn>

    </v-flex>

    <v-flex xs12 class="teacher-desc" @click="toggle(7, 0, $event)">
      <v-subheader >
        بازخورد مشاور
      </v-subheader>
      <div class="desc-data text-xs-right" v-if="rowsModelEnable[7][0]">{{rowsModel[7][0]}}</div>
      
      <v-textarea
        label="بازخورد مشاور"
        v-model="rowsModel[7][0]"
        hint="بازخورد خود را در رابطه با این هفته دانشجو در این قسمت بنویسید."
        solo
        v-if="!rowsModelEnable[7][0]"
        @keyup.native.enter="setfield(7, 0, $event.target.value)"
        :loading="inputLoading"
        :disabled="inputLoading"        
      >
        <v-progress-linear
          v-model="inputLoading"
          :indeterminate="true" 
          color="primary" 
          height="3"
          ></v-progress-linear>     
      </v-textarea>
    </v-flex>

    </v-layout>
  </v-container>
  
  <v-snackbar
    v-model="snackbar"
    :color="success"
    :timeout="6000">
    {{ snackbarText }}
    <v-btn dark flat @click="snackbar = false">بستن</v-btn>
  </v-snackbar>

</div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'studyTable',
  data() {
    return {
      success: 'success',
      loading : true,
      week : [
        'sat',
        'sun',
        'mon',
        'tue',
        'wed',
        'thu',
        'fri',          
      ],
      courses : [
        'طراحی سازه های فولادی', 
        'طراحی سازه های بتنی', 
        'آیین نامه 2800 و بارگزاری', 
        'مصالح بنایی و ساختمان صنعتی',
        'تحلیل سازه ها', 
        'جمع کل مطالعات هفتگی'
      ],
      uid : 3300,
      rows : [],
      rowsModel : [],
      rowsModelEnable : [],
      result : [],
      snackbar : false,
      snackbarText : '',
      inputLoading: false,
      weeks: [1,2],
      selectedWeek:1,
      //description section
      studentDesc: "توضیحات مربوط به این هفته ی خود را در این قسمت برای مشاور بنویسید.",
      teacherDesc: '',
      descLoading : false
    }
  },
  mounted() {
    this.update()
  },
  computed:{
    headers: function(){
      var temp = [{ text: 'دروس', value: 'courses', width: 2}]
      this.week.map((Key) => {
        temp.push({ text: this.day_names(Key), value: Key , width: 1})
      })
      temp.push(
        {text: 'جمع کلاس هفتگی', value: 'sum', width: 2}, 
        {text: 'نتیجه آزمونک', value: 'result', width: 1}, 
      )
      return temp
    },
  },
  methods: {
    day_names(value) {
        if (!value) return ''
        value = value.toString()
        if     (value == 'sat') return 'شنبه'
        else if(value == 'sun') return 'یکشنبه'
        else if(value == 'mon') return 'دوشنبه'
        else if(value == 'tue') return 'سه شنبه'
        else if(value == 'wed') return 'چهارشنبه'
        else if(value == 'thu') return 'پنج شنبه'
        else if(value == 'fri') return 'جمعه'
        else return value
    },
    toggle(cid, did, e) {
      var prev = false
      var waiting = 1000

      //check previuse enabled cell
      this.rowsModelEnable.map((course, c) => {
        course.map((day, d) => {
          //if there was a previuse one
          if((cid != c || did != d) && this.rowsModelEnable[c][d] == false){
            //update its value
            this.setfield(c, d, this.rowsModel[c][d])
            prev = true
            setTimeout(()=>{
              this.$set(this.rowsModelEnable[c], d, true)
            },waiting)
          }
        })
      })

      //wait for previuse loading to finish
      //this is for prevent appearing loading bar in the newly clicked text field
      if(prev)
        setTimeout(()=>{this.$set(this.rowsModelEnable[cid], did, false)},waiting)
      else
        this.$set(this.rowsModelEnable[cid], did, false)

      var inputs = e.target.parentNode.getElementsByTagName('input')
      var textareas = e.target.parentNode.getElementsByTagName('textarea')
      setTimeout(()=>{
        if(inputs.length) inputs[0].focus()
        if(textareas.length) textareas[0].focus()
      },30)
    },
    setfield(cid, did, val){
      console.log(val)
      this.inputLoading = true
      var rawData = {
        value: val,
        cid: cid,
        did: did,
        type: 'edit',
        week: this.selectedWeek,
      }
      var formData = new FormData()
      formData.append('data', JSON.stringify(rawData))
      axios.post('http://civil808.com/user/'+ this.uid +'/moshavereh', formData, {
        headers: {
          'Content-Type': 'multipart/form-data'
        }
      })
      .then(response => response.data)
      .then(data => {
        if(data.status == 101){
          this.snackbar = true
          this.snackbarText = data.message
        }else{
          this.snackbar = true
          this.snackbarText = 'failed'
        }
        this.$set(this.rowsModelEnable[cid], did, true)
        this.updateVals(cid,did)
      })

    },
    updateVals(cid, did){
      //haldling row
      var sum = 0
      this.rowsModel[cid].map((day, d)=>{
        if(d < 7 ) //last item
          sum += parseInt(day)
      })
      this.$set(this.rowsModel[cid], 7, sum)

      //handling column
      sum = 0
      var total = 0
      this.rowsModel.map((course, c)=>{
        if(c < 5 ){ //last item
          course.map((day, d)=>{
            if(d < 7 && did == d) //last item
              sum += parseInt(day)
            else if(7 == d)
              total += parseInt(day)
          })
        }
      })
      this.$set(this.rowsModel[5], did, sum)
      this.$set(this.rowsModel[5], 7, total)
      this.inputLoading = false
    },
    setFocus(e){
      console.log(e.target.parentNode)//.getElementsByTagName('div')[0]
      e.target.nextSibling.getElementsByTagName('input')[0].focus()
    },
    update(){
      this.loading = true
      this.rowsModelEnable = [...Array(8)].map(e => Array(9).fill(true))
      axios.get('http://civil808.com/user/'+ this.uid +'/moshavereh', {
        params: {
          week: this.selectedWeek
        }
      })
      .then(response => response.data)
      .then((data) => {
        var col = []
        var dataModel = [...Array(8)].map(e => Array(9).fill("0"))
        
        if(data.length > 0){
          var index = 0
          Object.keys(data[0]).map((key) => {
            var val = data[0][key]
            if(key == 'sat' || key == 'sun' || key == 'mon' || key == 'tue' || key == 'wed' || key == 'thu' || key == 'fri'){
              col[index] = val.split("@")
              var colSum = 0
              col[index].map(v => {
                colSum += parseInt(v)
              })
              col[index].push(colSum)
              index++
            }
            if(key == 'user_description'){
              dataModel[6][0] = val
            }
            if(key == 'teacher_eval'){
              dataModel[7][0] = val
            }
          });
        }else{
          col = [...Array(8)].map(e => Array(6).fill("0"))
        }

        var temp = [[],[],[],[],[],[]]
        col.map((day, did)=>{
          day.map((course, cid)=>{
            temp[cid].push({val: course, width: 1, key: "" + did + cid})
            dataModel[cid][did] = course
          })
        })

        var sum = 0
        if(data.length > 0){
          temp.map((course, cid)=>{
            sum = 0
            course.map((day)=>{
              sum += parseInt(day.val)
            })
            var did = course.length 
            temp[cid].push({val: sum, width: 2, key: "" + did + cid})
            dataModel[cid][did] = sum
            did++
            temp[cid].push({val: '', width: 1, key: "" + did + cid})
            dataModel[cid][did] = ''
          })
        }

        this.rows = temp
        this.rowsModel = dataModel
        this.loading = false
      })
    }
  },
  filters:{
    xs_ : function (value) {
      return 'xs'+ value
    }
  }
}
</script>

<style>
.primary {
	background-color: #9C27B0 !important;
	border-color: #9C27B0 !important;
}
.secondary {
	background-color: #b0bec5 !important;
	border-color: #b0bec5 !important;
}
.accent {
	background-color: #8c9eff !important;
	border-color: #8c9eff !important;
}
.error {
	background-color: #b71c1c !important;
	border-color: #b71c1c !important;
}
.success {
	background-color: #4CAF50 !important;
	border-color: #4CAF50 !important;
}
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
.v-text-field input {
	text-align: center;
  direction: ltr;
	font-size: 14px;
}
.v-text-field.v-text-field--solo .v-input__control {
	min-height: auto;
}
.table-data .v-input__slot {
	width: 50px;
	margin: 0 auto;
}
.v-snack__content .v-btn:first-of-type {
	margin: 0;
}
.v-menu>.v-menu__content {
	top: 0 !important;
	left: 0px !important;
}
.table-data .v-text-field__details {
	display: none;
}
.student-desc {
	margin: 30px 0 !important;
}
textarea {
	font-size: 14px !important;
}
.v-textarea label.v-label {
	font-size: 14px;
}
.v-subheader {
	cursor: pointer;
}
.student-desc {
	position: relative;
}
.student-desc button {
	position: absolute !important;
	background: #2196F3 !important;
  left: 0;
}
</style>
