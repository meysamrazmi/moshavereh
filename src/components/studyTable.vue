<template>
  <v-container>
    <v-layout column>
      <v-flex xs12 light="true">

      <!-- header -->
      <v-layout row wrap>
        <v-flex v-for="item in headers" :key="item.value" :class="item.width | xs_">
          <span style="padding-bottom: 10px;display: inline-block;">{{item.text}}</span>
        </v-flex>
      </v-layout>
      <v-divider></v-divider>
      <v-divider></v-divider>

      <!-- rows -->
      <div v-for="item in courses" :key="item">
        <v-layout align-center justify-center class="text-xs-right">
          <v-flex style="padding: 10px;" xs2>{{item}}</v-flex>
          <v-flex v-for="row in rows" :key="row" class="text-xs-center" :class="row.width | xs_">{{row.val}}</v-flex>
        </v-layout>
        <v-divider></v-divider>
      </div>

      </v-flex>
    </v-layout>
  </v-container>
</template>

<script>
export default {
  name: 'studyTable',
  props: {
    msg: String
  },
  data() {
    return {
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
      rows : [],
      result : [],
    }
  },
  mounted() {
    fetch('http://civil808.com/user/3300/moshavereh')
    .then(response => response.json())
    .then((data) => {
      var temp = []
      var sum = 0

      if(data.length > 0){
        Object.keys(data[0]).map((key) => {
          var val = data[0][key]
          if(key == 'sat' || key == 'sun' || key == 'mon' || key == 'tue' || key == 'wed' || key == 'thu' || key == 'fri'){
            temp.push({val: val, width: 1})
            sum += parseInt(val)
          }
        });
      }
      temp.push({val: sum, width: 2})
      temp.push({val: '', width: 1})
      this.rows = temp
      this.result = data
    })
  },
  computed:{
    headers: function(){
      var temp = [{ text: 'دروس', value: 'courses', width: 2}]
      this.week.map((Key) => {
        temp.push({ text: this.day_names(Key), value: Key , width: 1})
      })
      temp.push(
        { text: 'جمع کلاس هفتگی', value: 'sum', width: 2}, 
        { text: 'نتیجه آزمونک', value: 'result', width: 1}, 
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
  },
  filters:{
    xs_ : function (value) {
      return 'xs'+ value
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
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
</style>
