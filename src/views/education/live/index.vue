<template>
  <div class="live">
    <div class="live-container">
      <div id="video-player" class="video-player"></div>
      <div class="chat-box">
        <div class="chat-box-top">
          <i class="el-icon-chat-dot-square"></i>
          <p>互动讨论</p>
        </div>
        <div class="chat-box-center">
          <div class="message-content" v-if="messageList.length">
            <div class="message-item"
                 v-for="(item, index) in messageList"
                 :key="index"
                 :class="userInfo.userName === item.userInfo.userName ? 'self':''"
            >
              <p class="message-item-top">{{`${item.userInfo.userName}${item.from === 'teacher' ? '（老师）':''}`}}</p>
              <div class="message-item-bottom">
                <div class="message-info">
                  {{item.messageInput}}
                </div>
              </div>
            </div>
          </div>
        </div>
        <div class="chat-box-bottom">
          <el-input v-model="messageInput" :maxlength="100"/>
          <el-button type="primary" class="submit-btn" @click="sendMessage">发送</el-button>
        </div>
      </div>
      <!--      <el-button @click="this.toSocket"></el-button>-->
    </div>
    <el-button @click="drawer=true" class="guide-btn" type="primary" icon="el-icon-warning-outline">直播使用教程</el-button>
    <el-drawer :visible="drawer" size="75%" :before-close="handleClose">
      <template #title>
        <h2 class="guide-title">使用教程</h2>
      </template>
      <div class="guide-container">
        <el-steps direction="vertical" align-center>
          <el-step title="一、下载OBS Studio" icon="el-icon-s-promotion" status="finish">
            <template #description>
              <div class="description-container">
                <div class="logo">
                  <img  src="@/assets/obs.png" alt=""/>
                  <p>OBS Studio</p>
                </div>
                <div class="description">
                  <p>下载OBS Studio Windows客户端</p>
                  <el-button type="primary" icon="el-icon-download">
                    <a href="https://cdn-fastly.obsproject.com/downloads/OBS-Studio-29.0.2-Full-Installer-x64.exe">下载</a>
                  </el-button>
                </div>
              </div>
            </template>
          </el-step>
          <el-step title="二、配置OBS Studio基础功能" icon="el-icon-s-promotion" status="finish">
            <template #description>
              <div class="description-container">
                <p>1.添加来源。安装好OBS Studio后打开，点击图示的“+”键。</p>
                <img src="@/assets/steps/step2_1.png"  class="description-img"/>
                <p>2.选择添加“显示器采集”。（如需露脸，可选择“视频采集设备”）</p>
                <img src="@/assets/steps/step2_2.png"  class="description-img"/>

                <p>3.点击“确定”。</p>
                <img src="@/assets/steps/step2_3.png"  class="description-img"/>

                <p>4.跳转后参照图中配置项，再点击“确定”。</p>
                <img src="@/assets/steps/step2_3_1.png"  class="description-img"/>

                <p>5.添加视频采集设备，同步骤1、2。</p>
                <img src="@/assets/steps/step2_4.png"  class="description-img"/>

                <p>6.步骤5中点击“确定”后，进入该窗口，再次点击确定。</p>
                <img src="@/assets/steps/step2_3_2.png"  class="description-img"/>

                <p>7.配置好后如图所示。</p>
                <img src="@/assets/steps/step2_5.png"  class="description-img"/>
              </div>
            </template>
          </el-step>
          <el-step title="三、开启OBS Studio推流" icon="el-icon-s-promotion" status="finish">
            <template #description>
              <div class="description-container">
                <p>1.打开“设置”，如图标注所示。</p>
                <img src="@/assets/steps/step3_1.png"  class="description-img"/>

                <p>2.左侧分类栏选择“直播”。</p>
                <p>在图中的“服务器”输入框中输入：<el-link>rtmp://43.139.40.29/live/</el-link>；</p>
                <p>在图中的“推流码”输入框中输入：<el-link>livestream?secret=5e424f92060a41898c324c47a4a0999e</el-link>，点击右下方的“保存”。</p>
                <img src="@/assets/steps/step3_2.png"  class="description-img"/>

                <p>3.点击开始直播后就可以使用在线课堂。</p>
                <img src="@/assets/steps/step3_3.png"  class="description-img"/>
              </div>
            </template>
          </el-step>
        </el-steps>
      </div>
    </el-drawer>
  </div>
</template>

<script>
import HlsJsPlayer from 'xgplayer-hls.js'
import io from 'socket.io-client'
import userApi from '@/api/user'
export default {
  name: 'index',
  data () {
    return {
      drawer: false,
      videoPlayer: null,
      socket: null,
      userInfo: {},
      messageInput: '',
      messageList: []
    }
  },
  methods: {
    handleClose () {
      this.drawer = false
    },
    sendMessage () {
      this.socket.emit('send', { from: 'teacher', userInfo: this.userInfo, messageInput: this.messageInput })
      console.log(666)
    }
  },
  mounted () {
    userApi.getCurrentUser().then(res => {
      if (res.code === 1) {
        this.userInfo = res.response
      } else {
        this.$message.warning('发生错误')
      }
    })
    this.videoPlayer = new HlsJsPlayer({
      el: document.querySelector('#video-player'),
      url: 'http://43.139.40.29/live/livestream.m3u8',
      width: '75%',
      height: 540,
      volume: 0.6, // 初始音量
      autoplay: true, // 自动播放
      playsinline: true,
      isLive: true,
      cors: true,
      lang: 'zh-cn'
    })

    this.socket = io({
      path: '/socket.io',
      transports: ['websocket']
    })
    this.socket.on('connect', () => {
      console.log('connect！')
    })
    this.socket.on('disconnect', () => {
      console.log('disconnect!')
    })
    this.socket.on('resend', (data) => {
      console.log(data)
      this.messageList.push(data)
      this.messageInput = ''
    })
  }
}
</script>

<style scoped>
  .live-container{
    width: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
    padding-top: 40px;
    overflow: hidden;
  }

  .video-player{
    position: relative;
    z-index: 0;
  }

  .guide-btn{
    margin:20px 0 20px 28px;
  }

  .chat-box{
    width: 20%;
    background: #eee;
    display: flex;
    flex-direction: column;
    overflow: hidden;
  }

  .chat-box-top{
    display: flex;
    align-items: center;
    height: 40px !important;
    padding: 0 10px;
    /*background: #b6b6b6;*/
    border-bottom: 1px solid rgba(0,0,0, .1);
  }

  .chat-box-top p{
    color: #2d2f33;
    font-weight: bolder;
    margin-left: 5px;
  }

  .el-icon-chat-dot-square{
    font-size: 24px;
  }

  .chat-box-center{
    flex-grow: 1;
    width: 100%;
    height: 450px;
  }

  .chat-box-bottom{
    height: 50px !important;
    /*border-top: 1px solid silver;*/
    display: flex;
    box-shadow: 0 2px 4px rgba(0,0,0,.5);
    align-items: center;
    padding: 0 5px;
  }

  .submit-btn{
    margin-left: 5px;
  }

  .message-content{
    height: 100%;
    padding: 8px;
    display: flex;
    flex-direction: column;
    overflow-y: auto;
  }

  .message-item{
    width: 100%;
    display: flex;
    flex-direction: column;
  }
  .message-item+.message-item{
    margin-top: 16px;
  }

  .message-item-top{
    font-size: 12px;
    padding: 4px 0;
    margin: 0;
  }

  .message-item-bottom{
    display: flex;
    align-items: center;
    justify-content: start;
  }

  .message-info{
    background: white;
    padding: 5px 10px;
    max-width: 80%;
    border-radius: 5px;
    font-size: 14px;
  }

  .self .message-item-top{
    text-align: right;
  }

  .self .message-item-bottom{
    justify-content: end;
  }

  .self .message-info{
    background: #5bcb6d;
    color: #fff;
  }

  .guide-container{
    padding:  16px;
  }

  .guide-title{
    margin: 16px 0 32px 0;
    padding-left: 10px;
    border-left: 10px solid #5CB76A;
  }

  .description-container{
    padding: 20px 0;
    color: #000;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-direction: column;
  }
  .description-container>p{
    width: 75%;
    display: inline-block;
    align-items: center;
    font-size: 16px;
    padding: 5px 0;
    margin: 0;
  }

  .logo{
    display: flex;
    align-items: center;
  }

  .logo img{
    width: 50px;
    height: 50px;
  }

  .logo p{
    font-size: 36px;
    font-family: 黑体 ,serif;
    padding-left: 16px;
  }

  .description{
    width: fit-content;
    display: flex;
    align-items: center;
    flex-direction: column;
    justify-content: center;
  }

  .description-img{
    width: 75%;
    margin-bottom: 40px;
    border-radius: 10px;
  }

  .description p{
    font-size: 20px;
    color: #474747;
  }
</style>
