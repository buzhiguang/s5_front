<template>
    <div class="bazi-paipan">
            <div class="input-box">
                <RadioGroup v-model="isLunar" type="button" @on-change="checkLunarSolar($event)"> 
                    <Radio label="1" style="padding:0px 7px">阳历</Radio>
                    <Radio label="0" style="padding:0px 7px">阴历</Radio>
                </RadioGroup>&nbsp;
                <DatePicker type="date" v-model="dateObj" placeholder="选择日期" style="width:15%" data-active="date" :editable="false"></DatePicker>
                <i-switch size="large" v-model="isLeap" v-show="showLeapY" style="line-height:24px;" > 
                    <span slot="open">闰月</span>
                    <span slot="close">平月</span>
                </i-switch>
                <TimePicker type="time" v-model="dateObj" placeholder="选择时间" style="width: 15%" data-active="date" :editable="false"></TimePicker>
                <RadioGroup v-model="sex" style="margin:0 8px">
                    <Radio label="1">男</Radio>
                    <Radio label="0">女</Radio>
                </RadioGroup> 
                <!-- <Button type="primary" icon="ios-search" @click="queryBazi">确&nbsp;&nbsp;定</Button> -->
                <!-- <Button type="primary" shape="circle" icon="ios-search" style="height:33px" @click="queryBazi">开始排盘</Button> -->
                <Button type="primary" shape="circle" icon="ios-search" style="height:33px" @click="queryBazi">开</Button>
                <div style="display:inline-block"></div>
                <div style="display:inline-block;height:32px;float:right">
                    <Button type="ghost" @click="saveBazi" shape="circle" icon="ios-cloud-upload-outline" style="margin-top:-23px">
                    保 存</Button>
                    <div style="display:inline-block;" class="name-box">
                        <Input v-model="nameValue" style="width:155px;margin-top:0">
                            <span slot="prepend">名字</span>
                        </Input>
                    </div>
                </div>
            </div>
            <div class="bazi-show-box">
                <div class="bazi-view">
                    <!-- <v-bazi-view :bzInfo="baziInfo" v-if="bzViewShow"  style="background:rgba(200,200,200,.2);"></v-bazi-view> -->
                    <v-bazi-view :bzInfo="baziInfo" v-if="bzViewShow"></v-bazi-view>
                </div>
                <div class="bazi-chart">
                    <v-bazi-chart v-if="bzViewShow"></v-bazi-chart>
                </div>

            </div>
    </div>
</template>
<script>
import BaziView from './tpl/bazi_view';
import BaziChart from './tpl/bazi_chart';
export default {
  data() {
    return {
        // timeDate:new Date('2002-6-5 12:00:00'),
        dateObj:sessionStorage.getItem('dateStr')?new Date(sessionStorage.getItem('dateStr')):new Date(),
        // timeObj:sessionStorage.getItem('dateStr')?new Date(sessionStorage.getItem('dateStr')):new Date(),
        sex: sessionStorage.getItem('sex')?sessionStorage.getItem('sex'):'1', //1男，0女
        dateStr:'', //日期 2002-02-02
        timeStr:'12:00:00', //时间 12:02:02
        nameValue:'', //
        isLunar: sessionStorage.getItem('isLunar')?sessionStorage.getItem('isLunar'):'1', //输入的是否是阴历  1阳历，0阴历
        isLeap: sessionStorage.getItem('isLeap')?!!(sessionStorage.getItem('isLeap')-0):false, //false, //是否选择闰月
        isAuto23:'1', //是否开启23点进入下一天
        showLeapY: false,

        baziInfo:null,
        bzViewShow:false,
    };
  },
  components:{
      'v-bazi-view':BaziView,
      'v-bazi-chart':BaziChart
  },
  methods: {
    saveBazi(){
        if(!sessionStorage.getItem('token')){
            this.$Modal.warning({
                title: '提示',
                content: '<p>需要登录后才能对数据存储、查看或修改！</p>',
                onOk: () => {
                    //返回panpan页
                    // this.routerHandle('baziPaipan');
                    // this.$router.push({path:'/bazi/paipan'});
                    // this.$store.commit('setActiveName', { name: 'baziPaipan' });
                    // localStorage.setItem('activeName','baziPaipan');
                }
            });
        }else{
            let dateStr = this.formatDate(this.dateObj);//格式化日期
            let userInfo = {
                name:this.nameValue||['斯大林','螺蛳粉','罗斯福','丘吉尔','希特勒','张伯伦','罗斯福'][Math.floor(Math.random()*10)%7] + Math.random().toString().substr(5,2) ,
                sex:this.sex,
                solar:this.baziInfo.lunarInfo.solar,
                lunarCH:this.baziInfo.lunarInfo.lunarCH,
                lunar:this.baziInfo.lunarInfo.lunar,
                isRemind:true //是否提醒
            }
            this.$http.post(apiPath+"/api/bz/add",userInfo,{emulateJSON:true}).then(res=>{
                if(res.body.code=='0'){
                    this.success('保存成功！');
                }
            },res=>{
                // this.warning("连接异常！");
            });
        }
    },
    queryBazi(){
        // if(!this.dateObj || !this.timeObj){
        if(!this.dateObj){
            this.warning("请将关键信息填写完整！");
            return false;
        }else if(this.dateObj.getFullYear()<1891||this.dateObj.getFullYear()>2100){
            this.warning("超出查询范围(1891-2100年)");
            return;
        }
        let dateStr = this.dateObj.getFullYear()+"-"+(this.dateObj.getMonth()+1)+"-"+this.dateObj.getDate()+" "+this.dateObj.getHours()+":"+this.dateObj.getMinutes()+":"+this.dateObj.getSeconds();
        sessionStorage.setItem('dateStr',dateStr);
        this.$http.post(apiPath+"/api/bz/calc",{
            year:this.dateObj.getFullYear(),
            month:this.dateObj.getMonth()+1,
            day:this.dateObj.getDate(),
            hour:this.dateObj.getHours(),
            minute:this.dateObj.getMinutes(),
            second:this.dateObj.getSeconds(),
            sex:this.sex,
            isLeap:this.isLeap?'1':'0', 
            isLunar:this.isLunar,
            isAuto23:this.isAuto23
            },{emulateJSON:true}).then(res=>{
            if(res.body.code==0){
                this.baziInfo = res.body.data;
                this.bzViewShow = true;
                console.log(this.baziInfo);
                //数据暂存本地
                sessionStorage.setItem('baziInfo',JSON.stringify(res.body.data));
            }else{
                this.warning (res.body.msg)
            }
        },res=>{
            // this.warning("连接异常！");
        })
    },
    //获取日期
    getDate(value){
        this.dateStr = value;
        this.setInputStyle('date',value);
    },
    // 获取时间
    getTime(value){
        this.timeStr = value;
        this.setInputStyle('time',value);
    },
    //日期 时间输入框有、无内容边框状态
    setInputStyle(){
        let inputs =  document.querySelectorAll('div[data-active=date] input');
        inputs.forEach(function(v,i){
            v.style.borderColor="#2d8cf0"; //变蓝色
        });
            
    },
    //监测选择阴历或阳历,显示闰月按钮
    checkLunarSolar(e){
        sessionStorage.setItem('isLunar',e);
        this.showLeapY = e==0 ? true:false; //0是阴历
    },
    warning (msg) {
        this.$Message.warning(msg);
    },
    success (msg) {
        this.$Message.success(msg);
    },
  },
  mounted() {
      this.setInputStyle(); //日期框蓝色边框
  },
  watch:{
      'sex':function(){sessionStorage.setItem('sex',this.sex)},
      'isLeap':function(){sessionStorage.setItem('isLeap',Number(this.isLeap))},
  },
  created(){
        this.showLeapY = this.isLunar==='0'?true:false; //控制闰月按钮显示
        if(sessionStorage.getItem('origin','router')){ //判断是否从其他路由跳过来paipan
            sessionStorage.setItem('origin','');//抹除origin数据
            this.queryBazi();
        }else if(sessionStorage.getItem('baziInfo')){ //上一次搜索的数据，有的话就使用
            this.baziInfo = JSON.parse(sessionStorage.getItem('baziInfo'));
            this.bzViewShow = true;
        }
  }

};
</script>
<style lang="less" scoped>
.bazi-paipan {
    div.name-box .ivu-input-group{
        /* top: 12px; */
        margin-left:15px;
        margin-top:-22px;
    }
    .input-box {
        margin-bottom:8px;
    }
    .bazi-show-box {
        // background-color: red;
        .bazi-view {
            width:442px;
            min-height: 400px;
            background-color: yellow;
            float:left;
        }
        .bazi-chart {
            padding-left:462px;
            // background-color: pink;
            width:100%;
            min-height:400px;
            // float:left;
        }
    }
}
</style>
