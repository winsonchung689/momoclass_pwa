<template>
  <div>
    <div style="background-color: #fff;;position: fixed; top: 0;display: flex;flex-direction: row; width: 500px;">
      <div>
        <i class="el-icon-arrow-left" @click="goOff()"></i>
      </div>
      <div style=" width: 50%;font-size: medium;font-weight: bolder;justify-content: center;display: flex;margin-left: 30px;margin-top: 5px;">{{ header }}</div>
    </div>
  
    <div style="margin-top: 15%;">
      <div v-if="isCalender" style="height: 50%;width: 100%;">
        <el-calendar v-model="value">
        <template slot="dateCell" slot-scope="{data}">
        <!--自定义内容-->
          <div>
             <div class="calendar-day" >{{data.day.split('-').slice(2).join('-') }}</div>
          </div>
      </template>
      </el-calendar>
      </div>

      <el-dialog :title="leave_student" :visible.sync="dialogFormVisible" style="width: 400px">
        <div v-if="isLeave">
          <el-switch
            style="display: block"
            v-model="leave_value"
            active-color="#13ce66"
            inactive-color="#ff4949"
            active-text="请假"
            inactive-text="旷课">
          </el-switch>
        </div>
        <div v-if="isSignIn">
          <template>
            <el-input-number v-model="class_count" :precision="2" :step="0.1" :max="10" step-strictly></el-input-number>
          </template>
        </div>
        
        <el-input v-model="mark_leave" placeholder="请输入备注"></el-input>
        <div slot="footer" style="display: flex;flex-direction: row;justify-content: space-between;">
          <el-button @click="dialogFormVisible = false">取 消</el-button>
          <el-button type="primary" @click="confirm_buttom()">确 定</el-button>
        </div>
      </el-dialog>

      <el-table :data="tableData" style="width: 100%;font-size: small;">
        <el-table-column type="expand">
            <template slot-scope="props">
                <div v-for="(item,index) in props.row.children" :key="index">
                  <div style="display: flex;flex-direction: row;margin-left: 20px;justify-content: left;margin-bottom: 20px;">
                    <div style="margin-right: 5px;font-size: small;margin-top: 10px;">{{ item.student_name }}</div>
                    <el-button @click="dialogFunction('leave','',item.class_number,item.student_name,item.subject,item.leave,item.duration,props.$index,index,'')" type="primary" plain style="margin-right: 5px;font-size: small;">{{ item.leave }}</el-button>
                    <el-button v-if="isBoss" @click="dialogFunction('signin',item.sign_up,item.class_number,item.student_name,item.subject,'',item.duration,props.$index,index,'')" type="primary" plain style="margin-right: 5px;font-size: small;">{{ item.sign_up }}</el-button>

                    <el-button v-if="isBoss" @click="dialogFunction('comment','',item.class_number,item.student_name,item.subject,'',item.duration,props.$index,index,item.comment_status)" type="primary" plain style="margin-right: 5px;font-size: small;">{{ item.comment_status }}</el-button>
                  </div>
                </div>
            </template>
        </el-table-column>
        <el-table-column prop="subject" label="科目" width="120">
        </el-table-column>
        <el-table-column prop="class_number" label="班号" width="120">
        </el-table-column>
        <el-table-column prop="duration" label="时间段" width="120">
        </el-table-column>
      </el-table>


      <div v-if="isComment" style="margin-bottom: 30px;">
        <div>
          <el-button type="text"  @click="back">取消</el-button>
        </div>
        <el-upload
          list-type="picture-card"
          style="display: inline"
          :auto-upload="false"
          :on-change="handleChange"
          :file-list="fileList"
          :multiple="true"
          :type="file"
          action="#">
            <i slot="default" class="el-icon-plus"></i>
            <div slot="file" slot-scope="{file}">
              <img
                class="el-upload-list__item-thumbnail"
                :src="file.url" alt=""
              >
            </div>
        </el-upload>
        <div>学生名: {{ leave_student }}</div>
        <div>时间段: {{ leave_duration }}</div>

        <div style="width: 40%;">
          <el-input
            placeholder="请输入课堂名称"
            v-model="class_name"
            clearable>
          </el-input>
        </div>
        <div style="width: 60%;">
          <el-input
            type="textarea"
            :autosize="{ minRows: 2, maxRows: 4}"
            placeholder="请输入课堂目标"
            v-model="class_target">
          </el-input>
        </div>


        <div>
          积极度:
          <el-rate
            v-model="positive"
            show-text>
          </el-rate>
        </div>
        <div>
          纪律性:
          <el-rate
            v-model="discipline"
            show-text>
          </el-rate>
        </div>
        <div>
          开心值:
          <el-rate
            v-model="happiness"
            show-text>
          </el-rate>
        </div>

        <div style="width: 80%;">
          <el-input
            type="textarea"
            :autosize="{ minRows: 2, maxRows: 4}"
            placeholder="请输入老师点评"
            v-model="class_comment">
          </el-input>
        </div>

        <div>
          <div style="color: pink;">录音时长: {{ record_duration }} 秒</div>
          <el-button-group>
          <el-button @click="startRecorder" type="primary" >开始录音</el-button>
          <el-button @click="pauseRecorder" type="primary" >暂停录音</el-button>
          <el-button @click="resumeRecorder" type="primary" >继续录音</el-button>
          </el-button-group>
        </div>

        <div>
          <el-button-group>
            <el-button @click="stopRecorder" type="primary" >停止录音</el-button>
          <el-button @click="playRecorder" type="success" >播放录音</el-button>
          <el-button @click="stopPlayRecorder" type="success" >停止播放</el-button>
          </el-button-group>
        </div>

        <div style="display: flex;justify-content: center; margin-top: 20px;">
          <el-button type="primary" @click="comment">提交</el-button>
        </div>
        
      </div>

    </div>
  </div>
    
</template>

<script>

import { HttpGet } from '@/api'
import { HttpPost } from '@/api'
import { sendNotification } from '@/api'
import { UploadFile } from '@/api'
import Recorder from 'js-audio-recorder'

const recorder = new Recorder({
          sampleBits: 16,               
          sampleRate: 16000,             
          numChannels: 1,
        }   
      );

recorder.onprogress = function(params) {
// console.log('--------------START---------------')
// console.log('录音时长(秒)', params.duration);
// console.log('录音大小(字节)', params.fileSize);
// console.log('录音音量百分比(%)', params.vol);
// console.log('当前录音的总数据([DataView, DataView...])', params.data);
// console.log('--------------END---------------')
}



export default {
  name: 'Timetable',
  data () {
    return {
      subject:this.$route.query.subject,
      studio:this.$route.query.studio,
      role:this.$route.query.role,
      openid:this.$route.query.openid,
      student_string:this.$route.query.student_string,
      header: '签到处',
      tableData: [],
      isBoss:true,
      value: new Date(),
      date_time: '',
      dialogFormVisible: false,
      leave_value:true,
      leave_student:'',
      leave_duration:'00:00-00:00',
      leave_type:'',
      mark_leave:'无备注',
      leave_subject:'',
      index1:'',
      index2:'',
      isLeave:false,
      isSignIn:false,
      isComment:false,
      class_count:1,
      sign_class_number:'',
      button:'',
      fileList:[],
      uuids:[],
      positive:5,
      discipline:5,
      happiness:5,
      class_target:'',
      class_comment:'',
      class_name:'',
      isCalender:true,
      mp3_url:'',
      record_duration:'standby'
    }
  },

  watch: {
    value: function() {
        this.date_time=''
        var year = this.value.getFullYear()
        var month = this.value.getMonth() + 1
        var date=this.value.getDate()
        if (date >= 1 && date <= 9) {
            date = "0" + date;
        }
        if (month >= 1 && month <= 9) {
            month = "0" + month;
        }
        let date_time = year + '-' + month + '-' + date
        this.date_time = date_time
        // console.log(year + '-' + month + '-' + date) 
        this.getLesson()

    }
},

  created () {
    this.getLesson();
  },

  methods: {

    startRecorder () {
      recorder.start().then(() => {
      }, (error) => {
      // 出错了
      console.log(`${error.name} : ${error.message}`);
      });

      this.record_duration = 'recording'
    },

    pauseRecorder () {
      let toltime = recorder.duration
      this.record_duration = Math.round(toltime)

      recorder.pause();
    },

    resumeRecorder () {
      this.record_duration = 'recording'
      recorder.resume()
    },

    stopRecorder () {
      let toltime = recorder.duration
      this.record_duration = Math.round(toltime)

      recorder.stop()
      this.getRecorder()
    },

    playRecorder () {
      this.record_duration = 'playing'
      recorder.play()
    },

    stopPlayRecorder () {
      this.record_duration = 'stanby'
      recorder.stopPlay();
    },

    async getRecorder(){
      let that = this
      let toltime = recorder.duration;//录音总时长
      let fileSize = recorder.fileSize;//录音总大小
      console.log(toltime,fileSize)
      //录音结束，获取取录音数据
      let PCMBlob = recorder.getPCMBlob();//获取 PCM 数据
      let wav = recorder.getWAVBlob();//获取 WAV 数据
      // const mp3Blob = this.convertToMp3(recorder.getWAV());

      let formdata ={
          photo: wav,
          class_target:'录音文件'
        }
      let res = await UploadFile('/push_photo', formdata)
      that.mp3_url = res.data.split("/")[3]
      console.log(res,that.mp3_url)
    },

    async getLesson () {
      let that = this

      if(that.role == 'client'){
        that.isBoss = false
      }

      if(that.date_time == ''){
        var year = new Date().getFullYear()
        var month = new Date().getMonth() + 1
        var date= new Date().getDate()
        if (date >= 1 && date <= 9) {
            date = "0" + date;
        }
        if (month >= 1 && month <= 9) {
            month = "0" + month;
        }
        let date_time = year + '-' + month + '-' + date
        that.date_time = date_time
      }

      let param ={
          date_time: that.date_time,
          studio:that.studio,
          subject:that.subject,
          openid:that.openid,
          test:'0'
        }
      const lessons = await HttpPost('/getSchedule', param)
      let lessons_data = lessons.data;
      // console.log(lessons_data)
      that.tableData = []
      let tmp = []
      for( var i=1; i < lessons_data.length; i++ ){
          const subject = lessons_data[i].subject
          const class_number = lessons_data[i].class_number
          const duration = lessons_data[i].duration
          const id = lessons_data[i].id

          let label = subject+class_number+duration
          let index = tmp.indexOf(label)
          if(index < 0){
            var json = {};
            json.subject = subject
            json.class_number = class_number
            json.duration = duration
            json.id = id
            tmp.push(label)

            let param = {
              date_time: that.date_time,
              studio:that.studio,
              subject:subject,
              openid:that.openid,
              duration:duration,
              class_number:class_number.replace('(插班生)',''),
              openid:that.openid
            }
            const details = await HttpPost('/getScheduleByClass', param)
            let details_data = details.data;
            
            var children = []
            for(var i=0; i < details_data.length; i++ ){
              var json_detail = {}
              if(that.role == 'boss' || that.role === 'teacher'){
                  const student_name = details_data[i].student_name
                  const id = details_data[i].id
                  const leave = details_data[i].leave
                  const sign_up = details_data[i].sign_up
                  const comment_status = details_data[i].comment_status
                  const subject = details_data[i].subject
                  const duration = details_data[i].duration
                  const class_number = details_data[i].class_number

                  json_detail.student_name = student_name
                  json_detail.leave = leave
                  json_detail.sign_up = sign_up
                  json_detail.comment_status = comment_status
                  json_detail.id = id
                  json_detail.subject = subject
                  json_detail.duration = duration
                  json_detail.class_number = class_number
                  children.push(json_detail)
              }else if (that.role == 'client'){
                  console.log(that.student_string)
                  let studentlist = that.student_string.split(',')

                  const student_name = details_data[i].student_name
                  let index = studentlist.indexOf(student_name)

                  if(index > 0){
                    const id = details_data[i].id
                    const leave = details_data[i].leave
                    const sign_up = details_data[i].sign_up
                    const comment_status = details_data[i].comment_status
                    const subject = details_data[i].subject
                    const duration = details_data[i].duration
                    const class_number = details_data[i].class_number

                    json_detail.student_name = student_name
                    json_detail.leave = leave
                    json_detail.sign_up = sign_up
                    json_detail.comment_status = comment_status
                    json_detail.id = id
                    json_detail.subject = subject
                    json_detail.duration = duration
                    json_detail.class_number = class_number
                    children.push(json_detail)
                }
              }
            }
            json.children = children
            that.tableData.push(json)
          }
      }
      // console.log(that.tableData)
    },

    dialogFunction (type,sign_up,class_number,student_name,subject,leave,duration,index1,index2,comment_status) {
      let that = this
      that.isLeave = false
      that.isSignIn = false
      that.mark_leave = ''
      that.class_count = 1
      if(type =='leave'){
        if(leave == '缺席'){
          that.button = 'leave'
          that.dialogFormVisible = true
          that.isLeave = true
          that.leave_student = student_name
          that.leave_subject = subject
          that.leave_duration = duration
          that.index1 = index1
          that.index2 = index2
        }else{
          that.$message({
            message: '学生' + leave,
            type: 'warning'
          });

          that.leaveRecord(subject,that.studio,student_name,leave.replace('已',''))
        }
      }else if(type == 'signin'){
        if(sign_up == '签到'){
          that.button = 'signin'
          that.dialogFormVisible = true
          that.isSignIn = true
          that.leave_student = student_name
          that.leave_subject = subject
          that.leave_duration = duration
          that.sign_class_number = class_number
          that.index1 = index1
          that.index2 = index2
        }else{
          that.$message({
            message: '学生' + sign_up,
            type: 'warning'
          });
          that.signInRecord(subject,that.studio,student_name)
        }
      }else if(type == 'comment'){
        if(comment_status == '课评'){
          let string1 = that.leave_subject + ',' +that.sign_class_number + ',' +that.leave_duration
          let string2 = subject + ',' + class_number + ',' + duration
          console.log(string1,string2)

          if(string1 != string2){
            that.class_target = ''
            that.class_name = ''
          }

          that.button = 'comment'
          that.isComment = true
          that.isCalender = false
          that.leave_student = student_name
          that.leave_subject = subject
          that.leave_duration = duration
          that.sign_class_number = class_number
          that.index1 = index1
          that.index2 = index2
          that.uuids = []
          that.positive = 5
          that.discipline = 5
          that.happiness = 5
        
          that.class_comment = '',
          that.fileList = [],
          that.uuids =[]

        }else{
          that.$message({
            message: '学生' + comment_status,
            type: 'warning'
          });
          that.growthRecord(subject,that.studio,student_name)
        }
      }
      
    },

    back () {
      let that = this
      that.isComment = false
      that.isCalender = true
      that.uuids = []
      that.positive = 5
      that.discipline = 5
      that.happiness = 5
      that.class_target = ''
      that.class_comment = ''
      that.class_name = ''
    },

    async addLeave () {
      let that = this;
      if(that.leave_value == true){
        that.leave_type = '请假'
      }else{
        that.leave_type = '旷课'
      }

      if(that.mark_leave.trim().length == 0  || that.mark_leave == null){
        that.mark_leave = '无备注'
      }
      // console.log(that.leave_type,that.leave_subject,this.leave_student,this,this.leave_duration,that.mark_leave)
      
      let param ={
        student_name: that.leave_student,
        studio:that.studio,
        date_time:that.date_time,
        duration: that.leave_duration,
        leave_type:that.leave_type,
        mark_leave:that.mark_leave,
        subject:that.leave_subject,
        openid:that.openid
      }
      await HttpPost('/leaveRecord', param)
      that.$message({
        message: '操作成功',
        type: 'success'
      });


      let param1 = {
        date_time: that.date_time,
        studio:that.studio,
        student_name:that.leave_student,
        duration:that.leave_duration,
        openid:that.openid
      }
      const leave_verify = await HttpPost('/getLeaveByDateDuration', param1)
      let data = leave_verify.data[0]
      let leave_type_get = data.leave_type
      that.tableData[that.index1].children[that.index2].leave = '已'+leave_type_get
      

    },

    async signIn () {
      let that = this
      if(that.mark_leave.trim().length == 0  || that.mark_leave == null){
        that.mark_leave = '无备注'
      }

      let param ={
        student_name:that.leave_student,
        studio:that.studio,
        date_time:that.date_time,
        duration:that.leave_duration,
        class_number:that.sign_class_number,
        subject:that.leave_subject,
        openid:that.openid,
        mark:that.mark_leave,
        class_count:that.class_count,
      }

      // console.log(param)
      await HttpPost('/signUpSchedule', param)
      that.$message({
        message: '签到成功',
        type: 'success'
      });

      let param1 = {
        date_time: that.date_time,
        studio:that.studio,
        student_name:that.leave_student,
        duration:that.leave_duration,
        openid:that.openid
      }
      const signin_verify = await HttpPost('/getSignUpByDateDuration', param1)
      let data = signin_verify.data[0]
      console.log(data)
      let id = data.id
      let openid = data.openid
      let subscription = data.subscription
      if(id){
        that.tableData[that.index1].children[that.index2].sign_up = '已签到'

        let message = that.leave_student +'同学已签到 !!!\n日期:'+ that.date_time +'\n时间:'+ that.leave_duration+'\n本次扣课:' + that.class_count + '课时'
        let json = {
          title:that.studio,
          message:message
        }
        console.log(subscription)
        let res = await sendNotification(subscription, JSON.stringify(json))
        console.log(res)
        if(res.status == 200){
            that.$message({
            message: '通知成功',
            type: 'success'
        });
        }
      }
    },

    async comment () {
      let that = this
      let uuids = ''
      for(var i in that.uuids){
        if(uuids==''){
          uuids = that.uuids[i]
        }else{
          uuids = uuids.concat(',' + that.uuids[i])
        }
      }
      let param ={
        class_name:that.class_name,
        student_name:that.leave_student,
        class_target:that.class_target,
        class_target_bak:'课评',
        comment:that.class_comment,
        studio:that.studio,
        date_time:that.date_time,
        duration:that.leave_duration,
        positive:that.positive,
        discipline:that.discipline,
        happiness:that.happiness,
        uuids:uuids,
        mp3_url:that.mp3_url
      }
      await HttpPost('/push', param)

      let param1 = {
        date_time: that.date_time,
        studio:that.studio,
        student_name:that.leave_student,
        duration:that.leave_duration,
        openid:that.openid
      }
      const comment_verify = await HttpPost('/getCommentByDateDuration', param1)
      let data = comment_verify.data[0]
      // console.log(data)
      let student_name = data.student_name
      if(student_name == that.leave_student){
        that.tableData[that.index1].children[that.index2].comment_status = '已课评'
        // that.isComment = false
          that.$message({
          message: '发布成功',
          type: 'success'
        });
      }else{
        that.$message({
        message: '发布失败',
        type: 'danger'
      });
      }
      // that.isCalender = true

    },

    async confirm_buttom () {
      let that = this
      that.dialogFormVisible = false
      if (that.button == 'leave'){
        await that.addLeave ()
      }else if (that.button == 'signin'){
        await that.signIn ()
      } 

    },

    async handleChange(file, fileList) {
        let that = this
        let formdata ={
          photo: file.raw
        }

        let res = await UploadFile('/push_photo', formdata)
        let uuid  = res.data.split("/")[3]
        that.uuids.push(uuid)
        // console.log(that.uuids)
    },
    
    goOff () {
      this.$router.go(-1);
    },

    signInRecord (subject,studio,student_name) {
      this.$router.push({ path: '/signinrecord', query: { subject: subject,studio: studio,student_name: student_name,role:this.role,openid:this.openid } })
    },

    growthRecord (subject,studio,student_name) {
      this.$router.push({ path: '/growthrecord', query: { subject: subject,studio: studio,student_name: student_name,role:this.role,openid:this.openid } })
    },

    leaveRecord (subject,studio,student_name,leave_type) {
      this.$router.push({ path: '/leaverecord', query: { subject: subject,studio: studio,student_name: student_name,role:this.role,openid:this.openid,leave_type:leave_type } })
    },

  }

}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
.el-icon-arrow-left{
  margin-left: 5px;
  margin-top: 10px;
  scale: 1.5;
}

.el-calendar .el-calendar__body .el-calendar-table__row .el-calendar-day {
    -webkit-box-sizing: border-box;
    box-sizing: border-box;
    padding: 8px;
    height: 50px;
}

.demo-table-expand {
  font-size: 0;
}
.demo-table-expand label {
  width: 90px;
  color: #99a9bf;
}
.demo-table-expand .el-form-item {
  margin-right: 0;
  margin-bottom: 0;
  width: 50%;
}

</style>
