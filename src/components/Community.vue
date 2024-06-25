<template>
    <div>
      <div style="display: block;position: fixed;background-color: #ffff;width: 100%;top: 0;">
        <div class="avatar">
          <div style="margin-top: auto;">
            <h1>HI,{{ nick_name }} ({{ mode }})</h1>
          </div>
          <div>
            <img :src="avatarurl" alt="" style="width: 30px;height: 30px;border-radius: 50%; position: relative;margin-left: 10px;margin-top: 5px;">
          </div>
        </div>
      </div>

      <div style="margin-top: 10%;" @touchstart="touchStart($event)" @touchmove="touchMove($event)" @touchend="touchEnd($event)">

        <div style="display: flex;flex-direction: row;justify-content: space-between;">
          <h2 @click="$router.push('/Login')">{{ hello }}</h2>
          <div style="display: flex;margin-right: 0%;">
            <img @click="click('/chatroom')" class="notice" src="../assets/wechat.png" alt="" >
          </div>
          <div style="display: flex;margin-right: 5%;">
            <img @click="click('/announcementrecord')" class="notice" src="../assets/notice.png" alt="" >
          </div>
        </div>

        <el-tabs v-model="type" @tab-click="handleClick">
          <el-tab-pane label="公共社区" name="public"></el-tab-pane>
          <el-tab-pane label="我的社区" name="private"></el-tab-pane>
        </el-tabs>

        <div>
          <div class="wrap" v-infinite-scroll="loadMore" infinite-scroll-disabled="busy" infinite-scroll-distance="50">

            <div style="justify-content: left;display: flex;margin-top: 5%;margin-bottom: 15px;flex-direction: column;" v-for="(item,index_out) in items">

                <div class="name_avatar" :style="{display:MinDisplay}">
                  <img :src="item.avatar" alt="" style="width: 40px;height: 40px;border-radius: 50%; position: relative;margin-left: 10px;margin-top: 5px;">
                  <div style="margin-left: 15px;font-size: medium;font-weight: bolder;color:dimgray;">{{ item.nick_name }}</div>
                </div>

                <div class="content" :style="{display:MinDisplay}">{{ item.content }}</div>

                <div class="covers" :style="{display:MinDisplay}">
                    <div class="cover" v-for="(img,index) in item.images" :key='img'>
                      <el-image 
                        style="margin-right: 10px;"
                        :src="img.src" 
                        :preview-src-list=[img.src]>
                      </el-image>
                    </div>
                </div>
                <div class="max" :style="{display:display}">
                    <div @click="ZoomOut"  v-for="(img,index) in item.images" :key='img' :class="[index_out+'_'+index===ShowIndex?'active':'None']" ><img :src="img.src" width="100%"></div>
                </div>


                <div style="margin-left: 50%;display: flex;flex-direction: row;">
                  <el-badge :value="item.like_amount" class="comment_item">
                    <el-button @click="like_button(item.studio_get,item.id,index_out)" size="small">点赞</el-button>
                  </el-badge>
                  <el-badge :value="item.comment_amount" class="comment_item">
                    <el-button @click="comment_button(item.studio_get,item.nick_name,item.id,index_out)" size="small">评论</el-button>
                  </el-badge>
                </div>

                <el-drawer
                  :visible.sync="drawer"
                  :direction="direction"
                  :with-header="false">

                  <div style="font-size: medium;font-weight:bolder;color: rgb(22, 133, 229);">To： {{ comment_studio }}{{ comment_nickname }}</div>
                  <div>
                    <el-input
                      type="textarea"
                      :rows="1"
                      placeholder="请输入内容"
                      v-model="comment_input">
                    </el-input>
                    <el-button style="margin-left: 90%;" @click="comment_confirm()" type="text">发送</el-button>
                    <el-divider></el-divider>
                    <div v-for="(item,index_out) in comment_items">
                      <div style="display: flex;flex-direction: row;">
                        <div style="font-size=smaller;color: #7d56cb;" >From {{ item.nick_name }}： </div>
                        <div style="font-size=small;color: #2160b7;">{{ item.content }}</div>
                      </div>
                    </div>
                  </div>
                </el-drawer>
                

                <div  class="foot" :style="{display:MinDisplay}">    
                  <div style="display: flex;flex-direction: row;justify-content: space-between;width: 98%;margin-top: 10px;">
                    <div style="display: flex;flex-direction: row;font-size: small;font-weight: bold;color: #a3b2b3;margin-top: 5px; margin-left: 10px">{{ item.studio_get }}  {{ item.create_time}}</div>
                    <el-popconfirm v-if="item.openid_get == openid" title="确定删除吗？" style="margin-left:;" @confirm="deleteRow(item.id)">
                      <el-button slot="reference" icon="el-icon-delete" type="danger" circle size="mini"></el-button>
                    </el-popconfirm>
                  </div>

                  <el-divider></el-divider>
                </div>
            </div>
          
          </div>


        </div>

      </div>


    </div>
</template>

<script>

import { HttpGet } from '@/api'
import { HttpPost } from '@/api'
import { sendNotification } from '@/api'
import { swiper, swiperSlide } from 'vue-awesome-swiper'
import WebSocket from '@/components/WebSocket'
import { ImageUrl } from '@/api'

export default {
  name: 'Home',
  components: {
    swiper,
    swiperSlide,
    WebSocket,
  },
  data () {
    return {
      studio: '',
      nick_name: '你好',
      avatarurl: '',
      openid: this.$route.query.openid,
      role: this.$route.query.role,
      subject:'',
      mode: '游客模式',
      hello:'去登陆 👉',
      diff:0,
      startY: 0,
      weekday:'',
      isBoss:false,
      isClient:false,
      subscription:'',
      pulickey:'BGVksyYnr7LQ2tjLt8Y6IELBlBS7W8IrOvVszRVuE0F97qvcV6qB_41BJ-pXPaDf6Ktqdg6AogGK_UUc3zf8Snw',
      privatekey:'oc5e7TovuZB8WVXqQoma-I14sYjoeBp0VJTjqOWL7mE',
      subsctiption_status:'未订阅',
      registration:'',
      centerDialogVisible: false,
      textarea: '',
      send_time:'',
      isBell1:true,
      isBell2:false,
      campus:'',
      count: 0,
      items: [],
      comment_items:[],
      MinDisplay:'flex',
      ShowIndex:0,
      display: 'none',
      page:1,
      busy:false,
      type: 'public',
      isMine:true,
      drawer: false,
      direction:'btt',
      comment_input:'',
      comment_studio:'',
      comment_nickname:'',
      comment_postid:'',
      index_out:'',
      srcList: [
          'https://fuss10.elemecdn.com/8/27/f01c15bb73e1ef3793e64e6b7bbccjpeg.jpeg',
          'https://fuss10.elemecdn.com/1/8e/aeffeb4de74e2fde4bd74fc7b4486jpeg.jpeg'
        ]
    }
  },

  created () {
    this.getUser()
    this.subscriptionInit()
  },

  mounted() {

  },
  methods: {

    async like_button(studio,post_id,index){
      let that = this
      let param1 ={
          openid:that.openid,
          post_id:post_id
      }
      let res1 = await HttpPost("/getPostLikeByOpenid",param1)
      let length = res1.data.length
      if(length > 0){
        that.$message({
            message: '该帖已赞👍',
            type: 'success'
        });
      }else {
        let param ={
          studio:studio,
          openid:that.openid, 
          post_id:post_id,
        }
        await HttpPost('/insertPostLike', param)
        that.items[index].like_amount += 1
        }
      },

    async comment_confirm(){
      let that = this
      let param ={
        studio:that.studio,
        openid:that.openid, 
        post_id:that.comment_postid,
        content:that.comment_input
      }
      await HttpPost('/insertPostComment', param)
      that.items[that.index_out].comment_amount += 1
      that.getComments()
    },

    getComments(){
      let that = this
      that.comment_items = []
      let param ={
        post_id:that.comment_postid,
      }
      const res = HttpPost('/getPostComment', param)
      res.then(res => {
        let data= res.data
        that.comment_items = data
      })

    },

    comment_button(studio,nick_name,post_id,index_out){
      let that = this
      that.drawer = true
      that.comment_input = ''
      if(studio){
          that.comment_studio = '(' + studio + ')'
      }
      
      that.comment_nickname = nick_name
      that.comment_postid = post_id
      that.index_out  = index_out
      that.getComments()
    },

    deleteRow(id) {
      let that = this;
      let param ={
          studio:that.studio,
          id:id,
          role:that.role,
          openid:that.openid
        }
      let res = HttpPost("/deletePost",param)
      res.then(res => {
          if(res.data == 1){
            that.$message({
                message: '删除成功',
                type: 'success'
            });

            that.items = []
            that.page = 1
            that.getPost(that.page,that.type)
          }else {
            that.$message({
                message: '删除失败',
                type: 'warning'
            });
          }
      })
    },

    loadMore () {
      let that = this
      that.busy = true
      setTimeout(() => {
        that.page += 1
        console.log(that.page)

        that.getPost(that.page,that.type)
        that.busy = false;
      }, 1000);
    },

    async getUser () {
      let that = this
      if(this.openid){
          that.page  = 1
          that.type = 'public'
          const users = await HttpGet('/getUser?openid=' + this.openid)
          // console.log(users)
          that.studio = users.data[0].studio
          that.subscription = users.data[0].subscription
          that.campus = users.data[0].campus
          if(that.studio.length>0){
            that.hello = '欢迎来到《' + that.studio + "》！"
            if(that.campus != that.studio){
              that.hello = '欢迎来到《' + that.studio+ "("+ that.campus +")" + "》！"
            }
          }else {
            that.hello = '去登陆 👉'
          }

          that.nick_name = users.data[0].nick_name
          that.avatarurl = users.data[0].avatarurl
          that.role = users.data[0].role
          that.subject = users.data[0].subjects
          that.comment_style = users.data[0].comment_style
          that.send_time = users.data[0].send_time
          
          if('boss' == that.role){
            that.mode = '校长模式'
            that.isBoss = true
          }else if('teacher' == that.role){
            that.mode = '老师模式'
            that.isBoss = true
          }else if('client' == that.role){
            that.mode = '家长模式'
            that.isClient = true
          }else if('visit' == that.role){
            that.mode = '游客模式'
          }else {
            that.mode = '未登录'
          }
      }

      that.items = []
      that.getPost(that.page,that.type)
    },

    async handleClick() {
      let that = this
      that.items = []
      that.page = 1
      if(that.type == 'public'){
        await that.getPost(that.page,that.type)
      }else if(that.type == 'private'){
        await that.getPost(that.page,that.type)
      }
    },

    async getPost (page,type) {
      let that = this;
      let param ={
        studio:that.studio,
        page:page,
        openid:that.openid,
        type:type
      }
      // console.log(param)
      const post = await HttpPost('/getPost', param)
      let post_data = post.data;
      // console.log(post_data)

      for( var i in post_data){
          const uuids = post_data[i].uuids
          let images =[]
          if(uuids){
            let uuidslist =uuids.split(",");
            for(let i in uuidslist){
                let url = ImageUrl + uuidslist[i];
                var json = {
                  src:url
                }
                images.push(json);
            }
            post_data[i]["images"] = images;
          }
      }

      for(var i in post_data){
        that.items.push(post_data[i])
      }
      // console.log(that.items)
    },

    click (url) {
      this.$router.push({ path: url, query: { openid: this.openid,role:this.role,studio:this.studio,subject:this.subject,comment_style:this.comment_style,send_time:this.send_time } })
    },

    touchStart(e) {
      this.diff = 0
      this.startY = e.targetTouches[0].pageY
      this.startScroll = this.$el.scrollTop || 0
    },

    touchMove(e) {
      let diff = e.targetTouches[0].pageY - this.startY - this.startScroll
      // console.log(diff)
      this.diff = diff
    },

    touchEnd(e) {
      if(this.diff >= 200){
        this.isBoss = false
        this.isClient = false
        this.getUser()
        this.$message({
        message: '刷新成功',
        type: 'success'
      });
      }
    },

    calender (subject) {
      this.$router.push({ path: '/calendar', query: { subject: subject,studio: this.studio,role:this.role,openid:this.openid,student_string:this.student_string } })
    },

    subscribe_button(){
      let that = this
      if(that.subsctiption_status === '未订阅'){
        that.notifyMe()
        that.subscribeUser()
      }else{
        that.unsubscribeUser()
      }
    },

    unsubscribeUser(){
      let swRegistration = this.registration
      swRegistration.pushManager.getSubscription().then(function (pushSubscription) {
          if (!pushSubscription) {
              // 用户尚未订阅
              return
            }
            // 取消订阅
            return pushSubscription.unsubscribe()
      })
      .then(function () {
        console.log('取消订阅！')
      })

      this.subsctiption_status='未订阅'

    },

    urlB64ToUint8Array(base64String) {
        const padding = '='.repeat((4 - base64String.length % 4) % 4);
        const base64 = (base64String + padding)
          .replace(/\-/g, '+')
          .replace(/_/g, '/');

        const rawData = window.atob(base64);
        const outputArray = new Uint8Array(rawData.length);

        for (let i = 0; i < rawData.length; ++i) {
          outputArray[i] = rawData.charCodeAt(i);
        }
        return outputArray;
    },

    subscribeUser() {     
        let openid = this.openid
        let swRegistration = this.registration
        const applicationServerPublicKey = this.pulickey
        const applicationServerKey = this.urlB64ToUint8Array(applicationServerPublicKey)
        swRegistration.pushManager.permissionState({
            userVisibleOnly: true,
            applicationServerKey: applicationServerKey
        })
        .then(function(permission) {
            console.log('User is granted:', JSON.stringify(permission));
            if (permission === 'granted') {
              swRegistration.pushManager.subscribe({
                  userVisibleOnly: true,
                  applicationServerKey: applicationServerKey
              })
              .then(function(subscription) {
                  console.log('User is subscribed:', JSON.stringify(subscription));
                  if (subscription) {
                    console.log('已订阅');
                    console.log(JSON.stringify(subscription));
                    let param = {
                      openid:openid,
                      subscription:JSON.stringify(subscription)
                    }
                    HttpPost('/updateSubscription',param)
                  } 
              })
              .catch(function(err) {
                  console.log('no subscribed: ', err);
              });
            }
        })
        .catch(function(err) {
            console.log('no permission: ', err);
        });
        this.subsctiption_status = '已订阅'
    },

    async subscriptionInit(){
      let that = this
      let openid = that.openid
      if ('serviceWorker' in navigator) {
        navigator.serviceWorker.register('./service-worker.js', { scope: '/' }).then(function (registration) { 
        that.registration = registration
        // console.log(registration)

        registration.pushManager.getSubscription().then(function(subscription) {
              if (subscription) {
                console.log('已订阅');
                that.subsctiption_status = '已订阅'
                console.log(JSON.stringify(subscription));
                let param = {
                  openid:openid,
                  subscription:JSON.stringify(subscription)
                }
                HttpPost('/updateSubscription',param)
              } else {
                console.log('未订阅');
              }
        });
        console.log('ServiceWorker registration successful with scope: ', registration.scope);
      }).catch(function (err) {
        console.log('ServiceWorker registration failed: ', err);
      });


      }
    },

    announcement(){
      let that = this
      that.centerDialogVisible=true
      that.textarea = ''
    },

    async announcement_confirm(){
      let that = this
      that.centerDialogVisible=false
      const users = await HttpGet('/getAllUserByStudio?studio=' + that.studio)
      // console.log(users.data)
      for( var i in users.data){
          let subscription = users.data[i].subscription
          let message = that.textarea
          let json = {
            title:that.studio,
            message:message
          }
          if(subscription){
            let res = await sendNotification(subscription, JSON.stringify(json))
            console.log(res)
          }

          if(i == 1){
            let param ={
                studio:that.studio,
                content:message,
                openid:that.openid
            }
            let res = await HttpPost('/insertAnnouncement', param)
            console.log(res)
        }
        
      }

      that.$message({
        message: '通知成功',
        type: 'success'
      });   
    },

    async notifyMe(){
      let that = this
      if (!window.Notification) {
        return Promise.reject('系统不支持桌面通知')
      }

      let promise = Notification.requestPermission()
      console.log(promise)
      promise.then(res => {
        console.log(res)
        if(res == 'granted'){
          that.isBell1 = false
          that.isBell2 = true
        }else{
          that.isBell1 = true
          that.isBell2 = false
        }
      })
     },

     ZoomIn(i){
        this.display='block';
        this.MinDisplay='none';
        this.ShowIndex=i;
    },

    ZoomOut(){
        this.display='none';
        this.MinDisplay='flex';
    },

    select(i){
        this.ShowIndex=i;
    },

  }
}
</script>

<style scoped>
.main_buttons{
  height: 80px;
  display: flex;
  flex-direction: row;
  justify-content: space-between;
   width: 95%;
   margin-top: 20px;
   margin-bottom: 20px;
}

.items{
  width: 100px;
  height: 100px;
  /* margin-right: 6%; */
  background-color:#ffffff89;
  border-radius: 1rem;
  justify-content: center;
  display: flex;
  flex-direction: column;
}

.icon{
  width: 40px;
  height: 40px;
  border-radius: 0.5rem;
}

.notice{
  width: 30px;
  height: 30px;
  border-radius: 0.5rem;
  margin-top: 10px;
}


.itemtext {
  font-size:small;
  display: flex;
  justify-content: center;
  color:#767e69;
  font-weight: bolder;
  margin-top: 5px;
}

h1 {
  font-weight: bold;
  font-size: medium;
  color: #a1bfbf91;
}

h2 {
  font-weight: bolder;
  font-size: large;
  color: #5aabee;
  margin-left: 5%;
  height: auto;
}

.today {
  width: 60%;
  display: flex;
  justify-content: space-between;
  border-radius: 1rem;
  background-color:  #b7f4d9;

}

h3 {
  color: #ffffff;
  /* background-color:  #b7f4d9; */
  justify-content: center;
  margin-left: 20px;
}


header {
  padding: 0 20px;
  width: 100%;
  height: 40px;
  position: fixed;
  top: 0;
  left: 0;
  background: #598cda;
  z-index: 99;
}

.avatar {
  width: 90%;
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  margin-left: 5%;
  background-color: #ffff;
  
}

text{
  font-weight: bolder;color: #767e69;font-size: medium;
}

.covers{
    width: 95%;
    display: flex;
    justify-content: left;
    flex-wrap: wrap;
    margin-left: 5%;
}

.cover{
    display: flex;
    justify-content: center;
    width: 45%;
    margin: 10px 0;
}

.foot{
  font-size: small;
  font-weight: bold;
  color: #608bc9;
  display: flex;
  flex-direction: column;
  margin-bottom: 5px;
}

.max{
    cursor: zoom-out;
    width: 100%;

}

.small{
    display: flex;
    justify-content: space-between;
    flex-wrap: wrap;
}
.cover-small{
    display: flex;
    justify-content: center;
    width: 10%;
    margin: 10px 0;
    opacity: 0.6;
    cursor: pointer;
}
.cover-small:hover{
    opacity: 1;
}
.active{
    display: flex;
}
.None{
    display: none;
}
.smallActive{
    opacity: 1;
}

.wrap{
  height: 100vh;
  overflow-y: auto;
}

.name_avatar{
  display: flex;
  flex-direction: row
}

.content{
  font-size: medium;
  color: dimgray;
  margin-top: 10px;
  margin-bottom: 0px;
  margin-left: 5%;
  margin-right: 5%;
}
.comment_item {
  margin-top: 10px;
  margin-right: 40px;
}
</style>
