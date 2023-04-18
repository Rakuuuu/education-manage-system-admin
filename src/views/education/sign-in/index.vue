<template>
<el-container v-loading="pageLoading" class="container">
  <div class="container-main">
    <h3>当前课堂状态：<span
        :class="currentStatus?'signing':'wait'"
      >{{currentStatus?"正在签到":"未开启签到"}}
      </span></h3>
    <h3 v-if="code">签到码：<span style="color:#fc6a6a;">{{this.code}}</span></h3>
    <el-button v-if="!currentStatus" type="primary" @click="startSignIn">开始签到</el-button>
    <el-button v-else type="warning" @click="closeSignIn">结束签到</el-button>

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
          {{ row.statues==='TRUE'?'已签到':'未签到' }}
        </template>
      </el-table-column>
      <el-table-column
        label="签到时间"
        align="center"
        width="150"
      >
        <template slot-scope="{row}">
          {{row.recordTime ? row.recordTime :'——'}}
        </template>
      </el-table-column>
    </el-table>
  </div>
</el-container>
</template>

<script>
import userApi from '@/api/user'
export default {
  name: 'index',
  data () {
    return {
      pageLoading: true,
      studentList: [],
      currentStatus: false, // 当前是否开启签到
      code: '',
      timer: 0
    }
  },
  mounted () {
    userApi.getAllRecord({
      flag: true
    }).then(res => {
      this.pageLoading = false
      console.log(res.list)
      if (res.code) {
        this.code = res.code
        this.currentStatus = true
        this.studentList = res.list
        this.startIntervalGetAllRecord()
      } else if (res.code === null) {
        this.$message.info('当前未开启签到')
      } else {
        this.$message.warning('发生未知错误')
      }
    })
  },
  destroyed () {
    clearInterval(this.timer)
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
          this.startIntervalGetAllRecord()
        } else { this.$message.warning('发生未知错误') }
        this.currentStatus = true
      })
    },
    closeSignIn () {
      userApi.getAllRecord({ flag: false }).then(res => {
      // console.log(res)
        if (!res.code) {
          this.$message.success('关闭签到成功')
        } else {
          this.$message.warning('发生未知错误')
          console.log(res)
        }
      })
    },
    getAllRecord () {
      userApi.getAllRecord({ flag: true }).then(res => {
        // console.log(res)
        if (res.code) {
          this.code = res.code
          this.currentStatus = true
          this.studentList = res.list
        } else { this.$message.warning('发生未知错误') }
      })
    },
    startIntervalGetAllRecord () {
      this.timer = setInterval(() => {
        this.getAllRecord()
      }, 2000)
    },
    clearIntervalGetAllRecord () {
      clearInterval(this.timer)
    }
  }
}
</script>

<style scoped>
  .container{
    width: 100%;
    height: 100%;
    padding: 30px 0;
    background: url(~@/assets/bg_sign.jpg) center no-repeat fixed;
    background-size: 100%;
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
  }

  .container-main h3{
    color: #5a5e66;
  }

  >>>.el-button{
    width: fit-content !important;
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
