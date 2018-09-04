<template>
<div data-app>
  <v-container class="elevation-2" style="padding-bottom: 36px;">
    <v-layout row wrap>
      <v-flex xs2 offset-xs1>
        <v-select
          v-model="selectedWeek"
          :items="weeks"
          label="انتخاب هفته"
          @change="update()"
        ></v-select>
      </v-flex>
      <v-flex class="week-date" xs9 v-if="this.weekStart != ''">
        از 
        <span>{{this.weekStart}}</span>
         تا 
        <span>{{this.weekEnd}}</span>
        <v-icon light style="margin-right: 10px;color: #ff5722;">calendar_today</v-icon>
      </v-flex>
      
    <v-flex class="table-data" xs12 light="true">

      <!-- header -->
      <v-layout row wrap style="text-align: center;">
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
      <div v-for="(item, cid) in courses" :key="item" v-if="rowsModelEnable.length > 2" :class="'row' + cid">
        <v-layout align-center justify-start class="text-xs-right">
          <v-flex style="padding: 10px;" xs2>{{item}}</v-flex>

          <v-flex 
            v-for="(row, did) in rows[cid]" 
            :key="row.key" 
            :class="row.width | xs_"
            @click="toggle(cid, did, $event)"
            class="text-xs-center">
            
            <!-- data -->
            <span v-if="rowsModelEnable[cid][did] && (rowsModel[cid][did] > 0 || cid == 5 || did == 7)">{{rowsModel[cid][did]}}</span>
            <span v-if="rowsModelEnable[cid][did] && !(rowsModel[cid][did] > 0 || cid == 5 || did == 7)">.</span>
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

      <v-btn 
        color="error" small absolute
        class="cancel-btn"
        @click="cancelToggle()"
        v-if="(gcid > -1 && gcid < 6) || gcid == 8"
        >لغو</v-btn>
    </v-flex>

    <v-flex xs12 class="student-desc description-area">
      <v-subheader  @click="toggle(6, 0, $event)">
        <v-icon light>format_quote</v-icon>
        توضیحات تکمیلی این هفته

        <v-btn color="purple" dark icon flat v-if="rowsModelEnable[6][0]" style="opacity: 0;">
          <v-icon light>edit</v-icon>
        </v-btn>
      </v-subheader>
      <div class="desc-data text-xs-right description-text" v-if="rowsModelEnable[6][0]">{{rowsModel[6][0]}}</div>
      
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
        small absolute flat
        class="cancel-btn"
        @click="cancelToggle()"
        v-if="!rowsModelEnable[6][0]"
        >لغو</v-btn>
      <v-btn 
        color="info" small absolute
        class="send-btn"
        @click="setfield(6, 0, $event.target.parentNode.parentNode.getElementsByTagName('textarea')[0].value)"
        v-if="!rowsModelEnable[6][0]"
      >ذخیره</v-btn>

    </v-flex>

    <v-flex xs12 class="teacher-desc description-area">
      <v-subheader  @click="toggle(7, 0, $event)">
        <v-icon light>thumbs_up_down</v-icon>
        بازخورد مشاور

        <v-btn color="purple" dark icon flat v-if="rowsModelEnable[7][0] && this.isAdmin" style="opacity: 0;">
          <v-icon light>edit</v-icon>
        </v-btn>
      </v-subheader>
      <div class="desc-data text-xs-right" v-if="rowsModelEnable[7][0] && rowsModel[7]">{{rowsModel[7][0]}}</div>
      
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

      <v-btn 
        small absolute flat
        class="cancel-btn"
        @click="cancelToggle()"
        v-if="!rowsModelEnable[7][0]"
        >لغو</v-btn>
      <v-btn 
        color="info" small absolute
        class="send-btn"
        @click="setfield(7, 0, $event.target.parentNode.parentNode.getElementsByTagName('textarea')[0].value)"
        v-if="!rowsModelEnable[7][0]"
      >ذخیره</v-btn>

    </v-flex>

    </v-layout>
  </v-container>
  
  <v-snackbar
    v-model="snackbar"
    :color="snackbarColor"
    :timeout="6000">
    {{ snackbarText }}
    <v-btn style="background: rgba(255, 255, 255, 0.25); 	margin-right: 15px;" icon dark @click="snackbar = false"><v-icon dark>close</v-icon></v-btn>
  </v-snackbar>

</div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'studyTable',
  data() {
    return {
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
      uid : 0,
      rows : [],
      rowsModel : [...Array(8)].map(e => Array(10)),
      rowsModelEnable : [...Array(8)].map(e => Array(10).fill(true)),
      gcid : -1,
      gdid : -1,
      result : [],
      inputLoading: false,
      //week section
      weeks: [1],
      selectedWeek:1,
      weekStart : ' / ',
      weekEnd: ' / ',
      //snackbar section
      snackbar : false,
      snackbarText : '',
      snackbarColor : 'success',
      //description section
      studentDesc: "توضیحات مربوط به این هفته ی خود را در این قسمت برای مشاور بنویسید.",
      teacherDesc: '',
      descLoading : false,
      isAdmin : false
    }
  },
  mounted() {
    this.start()
  },
  created(){
    axios.get('http://civil808.com/user/get_uid',{
      params: {
        origin: window.location.href.indexOf('localhost') > 0 ? 'localhost' : ''
      }
    })
    .then(response => response.data)
    .then((data) => {
      this.uid = data.uid
      if(data.roles.hasOwnProperty('3') || data.roles.hasOwnProperty('9') || data.roles.hasOwnProperty('10')){
        this.isAdmin = true
      }
    })
  },
  computed:{
    headers(){
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
      if(did == 7) return
      if(cid == 7 && !this.isAdmin) return
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
        setTimeout(()=>{
          this.$set(this.rowsModelEnable[cid], did, false)
          this.gcid = cid
          this.gdid = did
        },waiting)
      else{
        this.$set(this.rowsModelEnable[cid], did, false)
        this.gcid = cid
        this.gdid = did
      }

      var inputs = e.target.parentNode.getElementsByTagName('input')
      var textareas = e.target.parentNode.getElementsByTagName('textarea')
      setTimeout(()=>{
        if(inputs.length){
          inputs[0].focus()
          inputs[0].select()
        } 
        if(textareas.length){
          textareas[0].focus()
          textareas[0].select()
        }
      },30)
    },
    cancelToggle(){
      this.rowsModelEnable.map((course, c) => {
        course.map((day, d) => {
          if(this.rowsModelEnable[c][d] == false){
              this.$set(this.rowsModelEnable[c], d, true)
          }
        })
      })
      this.gcid = -1
      this.gdid = -1
    },
    setfield(cid, did, val){
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
          this.snackbarColor = 'error'
        }
        this.$set(this.rowsModelEnable[cid], did, true)
        
        //only if user pressed enter we should close cancle button
        if(this.gcid == cid && this.gdid == did){
          this.gcid = -1
          this.gdid = -1
        }
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
            if((d < 7 || d > 7) && did == d) //last item
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
    update(){
      if(this.uid == 0){
        this.snackbar = true
        this.snackbarText = 'خطایی در رابطه با کاربر پیش آمده است'
        this.snackbarColor = 'error'
        return
      } 
      this.loading = true
      axios.get('http://civil808.com/user/'+ this.uid +'/moshavereh', {
        params: {
          week: this.selectedWeek
        }
      })
      .then(response => response.data)
      .then((data) => {
        var col = []
        var examCol = []
        var dataModel = [...Array(8)].map(e => Array(10).fill("0"))
        
        if(data.length > 0){
          this.weekStart = data[0].created
          this.weekEnd = data[0].week_end

          var index = 0, examIndex = 0
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
            if(key == 'examTrue' || key == 'examFalse'){
              examCol[examIndex] = val.split("@")
              var cSum = 0
              examCol[examIndex].map(v => {
                cSum += parseInt(v)
              })
              examCol[examIndex].push(cSum)
              examIndex++
            }
          });
        }else{
          //filling default values
          col = [...Array(8)].map(e => Array(6).fill("0"))
          examCol = [...Array(2)].map(e => Array(6).fill("0"))
        }

        //for each cours
        //temp holds data by rows based on courses
        var temp = [[],[],[],[],[],[]] 
        col.map((day, did)=>{
          day.map((course, cid)=>{
            if(cid > 5) return //to prevent pushing to temp[6] that will return error
            temp[cid].push({val: course, width: 1, key: "" + did + cid})
            dataModel[cid][did] = course
          })
        })

        var sum = 0
        if(data.length > 0){
          //adding each courses total
          temp.map((course, cid)=>{
            sum = 0
            course.map((day)=>{
              sum += parseInt(day.val)
            })
            var did = course.length 
            temp[cid].push({val: sum, width: 2, key: "" + did + cid})
            dataModel[cid][did] = sum
          })

          examCol.map((exam, did)=>{
            exam.map((course, cid)=>{
              var realdid = did + 8
              temp[cid].push({val: course, width: 0, key: "" + realdid + cid})
              dataModel[cid][realdid] = course
            })
          })
        }

        this.rows = temp
        this.rowsModel = dataModel
        this.loading = false
      })
    },
    updateWeeks(){
      axios.get('http://civil808.com/user/'+ this.uid +'/moshavereh/weeks')
      .then(response => response.data)
      .then((data) => {
        var tmp = []
        for(var i = 1; i <= data; i++)
          tmp.push(i)

        this.weeks = tmp
      })
    },
    start(){
      if(this.uid == 0){
        setTimeout(()=>{
          this.start()
        },1000)
      }else{
        this.updateWeeks()
        this.update()
      }
    }
  },
  filters:{
    xs_ : function (value) {
      return 'xs'+ value
    }
  }
}
</script>

<style lang="scss">
#app{
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
    background-color: #ff5252 !important;
    border-color: #ff5252 !important;
  }
  .success {
    background-color: #4CAF50 !important;
    border-color: #4CAF50 !important;
  }
  h3 { margin: 40px 0 0; }
  ul {
    list-style-type: none;
    padding: 0;
    li {
      display: inline-block;
      margin: 0 10px;
    }
  }
  .v-text-field{
    input {
      text-align: center;
      direction: ltr;
      font-size: 14px;
    }
    &.v-text-field--solo .v-input__control { min-height: auto; }
  }
  .table-data {
    position: relative;
    margin-top: 20px;
    .cancel-btn {
      left: 0;
      margin-top: 5px;
      z-index: 2;
    }
    .v-input__slot {
      width: 50px;
      margin: 0 auto;
    }
    .xs0 {
      width: 45px;
      .v-input__slot { margin: 0;}
    }
    .row5 .xs0{
      &:nth-child(10), 
      &:nth-child(11){ 
        span:before {
          font-family: material icons;
          font-size: 22px;
          position: absolute;
        }
      }
      &:nth-child(10) span:before {
        content: "check_circle_outline";
        margin: 0 -30px 0 0;
        color: #4CAF50;
      }
      &:nth-child(11) span:before {
        content: "highlight_off";
        margin: 0px 20px 0 0;
        color: #F44336;
      }
    }
    .v-text-field__details {display: none;}
  }
  
  .v-snack__content .v-btn:first-of-type {
    margin: 0;
  }
  .v-menu > .v-menu__content {
    top: 0 !important;
    left: 0px !important;
  }
  button.send-btn {
    position: absolute !important;
    background: #2196F3 !important;
    left: 0;
  }
  .description-area{
    position: relative;
    &:hover .v-btn--flat { opacity: 1 !important; }
    &.student-desc { margin: 30px 0 !important; }
    textarea {font-size: 14px !important; }
    .v-textarea label.v-label {font-size: 14px;}
    .v-subheader {cursor: pointer;}
    .desc-data {
      padding: 14px 10px 5px;
      white-space: pre-line;
    }
  }
  .v-subheader {
    position: relative;
    &:before {
      content: "";
      width: 53px;
      position: absolute;
      height: 2px;
      background: #FF5722;
      bottom: 6px;
      right: 0;
    }
    > i {
      margin-left: 17px;
    }
  }
  .cancel-btn {
    left: 95px;
  }
  .v-select-list .v-list__tile__title {
    text-align: right;
  }
  .week-date {
    display: flex;
    align-items: center;
    justify-content: flex-end;
    span {
      padding: 10px;
      font-size: 13px;
    }
  }
}
</style>
