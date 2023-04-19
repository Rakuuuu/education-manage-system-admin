<template>
<el-container v-loading="pageLoading" class="container">
  <div class="container-main">
    <div class="top">
      <div class="top-left">
        <h3>当前课堂状态：<span
          :class="currentStatus?'signing':'wait'"
        >{{currentStatus?"正在签到":"未开启签到"}}
      </span></h3>
        <h3 v-if="currentStatus">签到码：<span style="color:#fc6a6a;">{{this.code}}</span></h3>
        <div style="display: flex">
          <el-button type="primary" @click="startSignIn" v-if="!currentStatus">开始签到</el-button>
          <el-button type="warning" @click="closeSignIn" v-else>结束签到</el-button>
        </div>
      </div>
      <div class="top-right" id="echarts-rate"></div>
    </div>

    <el-table
      border
      fit
      highlight-current-row
      :data="studentList"
      v-if="studentList.length !== 0"
      style="width: 100%">
      <el-table-column
        align="center"
        width="150"
        prop="studentName"
        label="姓名">
      </el-table-column>
      <el-table-column
        align="center"
        width="150"
        label="签到状态">
        <template slot-scope="{row}">
          <p :style="{color: row.statues==='TRUE'?'green':'red'}">{{ row.statues==='TRUE'?'已签到':'未签到' }}</p>
        </template>
      </el-table-column>
      <el-table-column
        label="签到时间"
        align="center"
        width="200"
      >
        <template slot-scope="{row}">
          {{row.recordTime ? `${row.recordTime.date.year}年${row.recordTime.date.month}月${row.recordTime.date.day}日 ${row.recordTime.time.hour}:${row.recordTime.time.minute}:${row.recordTime.time.second}` :'——'}}
        </template>
      </el-table-column>
    </el-table>
  </div>
</el-container>
</template>

<script>
import * as echarts from 'echarts'
import userApi from '@/api/user'
export default {
  name: 'index',
  data () {
    return {
      pageLoading: true,
      studentList: [],
      currentStatus: false, // 当前是否开启签到
      code: null,
      timer: 0,
      echartsSignRate: null,
      chartOption: {
        tooltip: {
          trigger: 'item'
        },
        legend: {
          left: 'center'
        },
        series: [
          {
            name: '到课率',
            type: 'pie',
            radius: ['30%', '70%'],
            avoidLabelOverlap: false,
            itemStyle: {
              borderRadius: 5,
              borderWidth: 0
            },
            label: {
              show: false,
              position: 'center'
            },
            labelLine: {
              show: false
            },
            data: [
              { value: 1048, name: '到课' },
              { value: 735, name: '缺勤' }
            ]
          }
        ]
      }
    }
  },
  mounted () {
    this.echartsSignRate = echarts.init(document.getElementById('echarts-rate'), 'macarons')
    userApi.getAllRecord().then(res => {
      setTimeout(() => {
        this.pageLoading = false
      }, 1000)
      this.studentList = res.list
      this.echartsSignRate.setOption(this.option([{ value: this.signInRate, name: '到课' }, { value: 1 - this.signInRate, name: '缺勤' }]))
      console.log(res)
      if (res.isAllowed === true) {
        this.currentStatus = true
        this.code = res.token
        this.getAllRecord()
      } else if(res.isAllowed === false){
        this.$message.info('当前未开启签到')
      }
    })
  },
  computed:{
    signInRate(){
      return (this.studentList.reduce((prV, curV) => {
        if(curV.statues === 'TRUE')
          return prV + 1
        else
          return prV
      }, 0.0) / this.studentList.length)
    }
  },
  methods: {
    startSignIn () {
      userApi.createRecord({
        courseId: 123456,
        teacherId: 2
      }).then(res => {
        if (typeof res === 'number') {
          this.code = res
          this.$message.success('签到码开启成功')
          this.getAllRecord()
        } else { this.$message.warning('发生错误') }
        this.currentStatus = true
      })
    },
    closeSignIn () {
      userApi.closeRecord({ flag: true }).then(res => {
        if (res.code === 1) {
          this.$message.success('关闭签到成功')
          this.currentStatus = false
          this.getAllRecord()
        } else {
          this.$message.warning('发生错误')
        }
      })
    },
    getAllRecord () {
      userApi.getAllRecord({ flag: false }).then(res => {
        console.log(res)
        if(res.isAllowed) { // 若处于开放签到状态则轮询
          this.studentList = res.list
          this.currentStatus = true
          this.code = res.token
          this.echartsSignRate.setOption(this.option([{ value: this.signInRate, name: '到课' }, { value: 1 - this.signInRate, name: '缺勤' }]))
          setTimeout(() => {
            this.getAllRecord()
          }, 2000)
        } else {
          this.currentStatus = false
          this.studentList = res.list
          this.echartsSignRate.setOption(this.option([{ value: this.signInRate, name: '到课' }, { value: 1 - this.signInRate, name: '缺勤' }]))
        }
      })
    },
    option (data) {
      return {
        tooltip: {
          trigger: 'item'
        },
        legend: {
          top: '5%',
          left: 'center'
        },
        series: [
          {
            name: '到课率',
            type: 'pie',
            radius: ['30%', '60%'],
            avoidLabelOverlap: false,
            itemStyle: {
              borderRadius: 5,
              borderWidth: 0
            },
            label: {
              show: false,
              position: 'center'
            },
            labelLine: {
              show: false
            },
            data: data
          }
        ]
      }
    }
  }
}
</script>

<style scoped>
  .container{
    width: 100%;
    height: fit-content;
    padding: 30px 0;
    background: url(~@/assets/bg_sign.jpg) center no-repeat fixed;
    background-size: 100% 100%;
    display: flex;
  }

  .container-main{
    padding: 20px;
    border-radius: 10px;
    background: rgba(255,255,255,0.75);
    box-shadow: 0 10px 20px rgba(0,0,0,.2);
    width: fit-content;
    margin: 0 auto;
    display: flex;
    justify-content: center;
    flex-direction: column;
    align-items: center;
    flex-shrink: 0;
  }

  .top{
    display: flex;
    align-items: center;
    justify-content: space-between;
  }

  .top-left{
    margin-right: 10px;
    margin-left: 30px;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-direction: column;
  }

  .top-left h3{
    color: #5a5e66;
    margin: 10px 0;
  }

  .top-right{
    width: 200px;
    height: 200px;
    /*background: #ececec;*/
    margin-left: 10px;
    padding-top: 10px;
  }

  >>>.el-button{
    width: fit-content !important;
    margin-top: 10px;
  }

  >>>.el-table{
    width: fit-content !important;
    height: fit-content !important;
    margin: 30px auto;
    border-radius: 10px;
    overflow: hidden;
  }

  .wait{
    color: #8c939d;
  }

  .signing{
    color: #5CB76A;
  }

</style>
