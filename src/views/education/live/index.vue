<template>
  <div>
    <div class="container">
      <h2>使用教程</h2>
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
  </div>
</template>

<script>
export default {
  name: 'index'
}
</script>

<style scoped>
  .container{
    padding:  16px;
  }

  .container h2{
    margin: 16px 0 32px 0;
    padding-left: 10px;
    border-left: 10px solid #5CB76A;
    border-radius: 5px 0 0 5px;
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
