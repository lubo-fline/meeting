<!--
 * @Author: lubo
 * @Date: 2021-06-03 14:31:05
 * @LastEditTime: 2021-08-11 14:58:58
 * @LastEditors: Please set LastEditors
 * @Description: 首页
 * @FilePath: \fline-meeting-manage-wechat\src\pages\index\Index.vue
-->
<template>
    <view class='bgF7F8FA'>
        <view>
            <calendarWeek @setSelectDate='setSelectDate'></calendarWeek>
        </view>
        <view class='fx_listBox'>
            <view class='fx_listCard' v-for='item in meetingRoomList' :key='item.meetRoomId'>
                <view>{{item.meetRoomName}}
                    <van-button 
                        custom-style='float:right;' 
                        round='true' size="small" 
                        color="linear-gradient(to right, #1E6FFF, #40A8FF)" 
                        @click='orderMeetingRoom(item)'
                    >预约</van-button>
                </view>
                <view class='clearBoth'></view>
                <view class='marginT16'>
                    <view v-for='ele in 24' :key='ele' class='fx_timeSaquareBox'>
                        <view>{{ele%2!=0?(parseInt(ele/2)+8):'.'}}</view>
                        <view :class='{fx_square:true,fx_squareSelect:(item.meetingSelect.indexOf(ele-1)>-1)}' ></view>
                    </view>
                </view>
            </view>
        </view>
        <van-tabbar
            fixed='true'
            @change="handleClick"
            :active="current"
        >
            <van-tabbar-item 
                v-for='item in tabsList' :key='item.title' 
                :icon="item.iconType"
            >
                {{item.title}}
            </van-tabbar-item>
        </van-tabbar>
        <van-popup
            :show="isOpened"
            closeable
            position="bottom"
            custom-style="height: 60%"
            bind:close="handleClose"
        >
            <view class='fx_modalHeader'>会议室预约
                <van-icon 
                    name='cross' size='14' color='#ccc'
                    custom-style="margin-top:8px;" 
                    custom-class='fx_closeIcon'
                    @click='handleClose'
                ></van-icon>
            </view>
            <van-row custom-class='fx_dateSelectBox'>
                <van-col span="12" custom-class='fx_dateSelectItem borderRddd'>
                    <view @tap='showDatePicker'>
                        <van-icon  :name="calendarPng" size='18' color='#1E6FFF'/>
                        <text class='fx_date'>{{ form.reservationDay }}</text>
                    </view>
                </van-col>
                <van-col span="12" custom-class='fx_dateSelectItem'>
                    <view class='fx_timePicker' @tap='showStartTimePicker'>
                        <van-icon name="clock" size='18' color='#1E6FFF'/>
                        {{ form.reservationStartTime==''?'开始':form.reservationStartTime }}
                    </view>
                    <view class='fx_timePicker' @tap='showEndTimePicker'>
                        <text class='fx_timeLine'>-</text>
                        <text class='fx_time'>{{ form.reservationEndTime==''?'结束':form.reservationEndTime }}</text>
                    </view>
                </van-col>
            </van-row>
            <van-cell-group>
                <van-field
                    :value="form.theme"
                    placeholder="请输入主题"
                    @change="(e)=>handleChange(e.detail,'theme')"
                />
            </van-cell-group>
            <view class='paddingLR32'>
                <van-button 
                    custom-style="margin-top:60px;" 
                    round='true' size="middle" block='true' 
                    color="linear-gradient(to right, #1E6FFF, #40A8FF)" 
                    @click='checkForm'
                >预约</van-button>
            </view>
        </van-popup>
        <van-popup
            :show="isShowPicker=='date'"
            closeable
            position="bottom"
            custom-style="height: 60%"
            bind:close="onClose"
        >
            <van-datetime-picker
                type="date"
                :value="dateTimePicker.date"
                :min-date=" minDate "
                @confirm='dateConfirm'
                @cancel='isShowPicker=""'
                :formatter='formatter'
            />
        </van-popup>
        <van-popup
            :show="isShowPicker=='time'"
            closeable
            position="bottom"
            custom-style="height: 60%"
            bind:close="onClose"
        >
            <van-datetime-picker
                type="time"
                :value="dateTimePicker.time"
                :min-hour=" 8 "
                :max-hour='19'
                bind:filter=" filter "
                @confirm='timeConfirm'
                @cancel='isShowPicker=""'
            />
        </van-popup>
        <van-toast id="van-toast" />
    </view>
</template>

<script>
// 按需引入, 更小的应用体积
import "./index.less";
import calendarWeek from '../../components/calendarWeek/Index.vue'
import Toast from '@vant/toast/toast';
import calendarPng from '@a/image/calendar.png'
import homeActivePng from '@a/image/homeActive.png'
import myPng from '@a/image/my.png'
export default {
    components: {
        calendarWeek,
    },
    data() {
        return {
            calendarPng,
            current:0,
            tabsList:[
                { title: '首页', iconType: homeActivePng,path:'/pages/index/Index'},
                { title: '我的', iconType: myPng,path:'/pages/my/Index' },
            ],
            meetingRoomList:[],
            isOpened:false,
            form:{
                theme:'',
                reservationDay:this.dateFormatter(new Date().getTime()),
                reservationStartTime: '',
                reservationEndTime:'',
                meetingRoomId:0,
                meetingRoomName:''
            },
            isShowPicker:'',//是否显示时间日期选择器
            dateTimePicker:{
                date:new Date().getTime(),
                time:"",
            },
            minDate: new Date().getTime(),
            timeSelectKey:'reservationStartTime',//判断现在是哪个时间选择
            formatter(type, value) {
                if (type === 'year') {
                    return `${value}年`;
                } 
                if (type === 'month') {
                    return `${value}月`;
                }
                return value;
            },
        };
    },
    onPullDownRefresh(){
        this.getAllMeeiting()
        this.$Taro.stopPullDownRefresh()
    },
    onShow(){
        this.getAllMeeiting()
    },
    methods: {
        filter(type, options){
            console.log(options)
            if (type === 'minute') {
                return options.filter((option) => option % 30 === 0);
            }
            return options;
        },
        //日历选择日期
        setSelectDate(date){
            this.form.reservationDay=date
            this.dateTimePicker.date=new Date(date).getTime()
            this.getAllMeeiting()
        },
        /**
         * @description: tab切换菜单
         * @param {*} current   选中项
         * @return {*}
         */        
        handleClick(e){
            let pagePath=this.tabsList[e.detail].path
            this.$switchTab(pagePath)
        },
        //关闭预约弹窗
        handleClose(){
            this.isOpened=false
        },
        //预约 打开预约弹窗
        orderMeetingRoom(datas){
            this.isOpened=true
            this.form.meetingRoomId=datas.meetRoomId
            this.form.meetingRoomName=datas.meetRoomName
        },
        /**
         * @description: 表单项切换赋值
         * @param {*} value  表单数据
         * @param {*} field  表单项字段名
         * @return {*}
         */        
        handleChange(value,field,test){
            if(field){
                this.form[field] = value
            }
        },
        //预约提交
        async orderSubmit(params){
            let datas = await this.$http.post(this.$api.orderMeetingUrl,params);
            let status=datas.code==200?'success':'error'
            let errorMsg=datas.code==200?'':datas.msg
            this.$navigateTo('/pages/orderResult/Index?status='+status+'&msg='+errorMsg)
        },
        /**
         * @description:校验表单 
         * @param {*}
         * @return {*}
         */        
        checkForm(){
            let flag=true;
            for(var item in this.form){
                if(this.form[item]==''){
                    flag=false
                    Toast('请输入完整表单信息！');
                }
            }
            if(flag){
                let params=JSON.parse(JSON.stringify(this.form))
                params.reservationStartTime=params.reservationDay+' '+params.reservationStartTime+':00'
                params.reservationEndTime=params.reservationDay+' '+params.reservationEndTime+':00'
                this.orderSubmit(params)
            }
        },
        /**
        * @description: 获取所有会议室的会议预约情况
        * @param {*}
        * @return {*}
        */        
        async getAllMeeiting(){
            let datas = await this.$http.get(this.$api.allMeetingUrl,{date:this.form.reservationDay});
            if(datas.code==200){
                this.meetingRoomList=datas.data||[];
                this.meetingRoomList.map(item=>{
                    item['meetingSelect']=this.getSelectTime(item.meeting||[]);
                })
            }
        },
        //根据会议开始结束时间处理数据
        getSelectTime(datas){
            let array=[];
            datas.forEach(item=>{
                let startTime=new Date(item.reservationStartTime).getTime();
                let endTime=new Date(item.reservationEndTime).getTime();
                let diffMinute=(endTime-startTime)/1000/60;
                let startHour=new Date(item.reservationStartTime).getHours()-8;
                for(var i =startHour;i<(diffMinute/30+startHour);i++){
                    array.push(startHour+i)
                }
            })
            return array;
        },
        /**
         * @description: 处理日期选择器后回显数据
         * @param {*} dateTime
         * @return {*}
         */        
        dateFormatter(dateTime){
            return new Date(dateTime).getFullYear()+'-'
            +((new Date(dateTime).getMonth()+1)<10?'0':'')+(new Date(dateTime).getMonth()+1)+'-'
            +(new Date(dateTime).getDate()<10?'0':'')+new Date(dateTime).getDate()
        },
        showStartTimePicker(){
            this.isShowPicker='time'
            this.timeSelectKey='reservationStartTime'
        },
        showEndTimePicker(){
            this.isShowPicker='time'
            this.timeSelectKey='reservationEndTime'
        },
        //开始时间和结束时间选择后时间的判断
        timeConfirm(e){
            let startTime='',startTimeStamp='',endTime='',endTimeStamp=''
            if(this.form.reservationStartTime!=''){
                startTime=this.form.reservationDay+' '+this.form.reservationStartTime+':00'
                startTimeStamp=new Date(startTime).getTime();
            }
            if(this.form.reservationEndTime!=''){
                endTime=this.form.reservationDay+' '+this.form.reservationEndTime+':00'
                endTimeStamp=new Date(endTime).getTime();
            }
            let selectTime=this.form.reservationDay+' '+e.detail+':00'
            let selectTimeStamp=new Date(selectTime).getTime();
            if(
                (
                    this.timeSelectKey=='reservationEndTime'
                    &&startTimeStamp!=''
                    &&selectTimeStamp<startTimeStamp
                )||
                (
                    this.timeSelectKey=='reservationStartTime'
                    &&endTimeStamp!=''
                    &&selectTimeStamp>endTimeStamp
                )
            ){
                Toast('开始时间应小于结束时间！');
            }else{
                this.form[this.timeSelectKey]=e.detail
                this.onClose()
            }

        },
        dateConfirm(e){
            this.form.reservationDay=this.dateFormatter(e.detail)
            this.onClose()
        },
        showDatePicker(){
            this.isShowPicker='date'
        },
        onClose(){
            this.isShowPicker=''
        },
    },
};
</script>
