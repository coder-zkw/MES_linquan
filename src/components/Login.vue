<template>
    <div class="login">
        <div class="loginWrap">
            <div class="loginHeader">
                <img src="../assets/logo.png" alt="林全科技">
                <h1>林全<span @dblclick="update">MES</span>系统</h1>
            </div>
            <el-form class="form-login" :rules="rules" ref="form" :model="form">
                <h2 class="form-title">用 户 登 录</h2>
                <el-form-item prop="userName">
                    <el-select v-model="form.userName" @change="handleChangeFocus" placeholder="请选择设备">
                        <i slot="prefix" class="iconTitle el-icon-s-platform"></i>
                        <el-option v-for="(item,i) in equipmentsList" :key="i" :label="item.BM_NAME" :value="item.BM_NAME"></el-option>
                    </el-select>
                    <el-button size="mini" v-show="isShow" plain class="changeEquip" @click="ChangeEquip">重新选择设备</el-button>
                </el-form-item>
                <el-form-item prop="operator">
                    <el-select v-model="form.operator" @change="handleChangeFocus" placeholder="请选择操作">
                        <i slot="prefix" class="iconTitle el-icon-s-order"></i>
                        <el-option v-if="isKeyboard" label="生产" value="0"></el-option>
                        <el-option label="调机" value="1"></el-option>
                        <el-option v-if="!isKeyboard" label="品检" value="2"></el-option>
                        <el-option v-if="!isKeyboard" label="审核" value="3"></el-option>
                    </el-select>
                </el-form-item>
                <el-form-item prop="jobNum" class="clearMargin">
                    <el-input v-model="form.jobNum" v-focus ref="inputSao" @keyup.enter.native="scanLogin()" placeholder="请扫码登录">
                        <i slot="prefix" class="iconTitle el-icon-user"></i>
                    </el-input>
                    <el-button 
                        v-if="!isKeyboard"
                        class="saoma"
                        size="mini" 
                        circle 
                        type="primary" 
                        icon="el-icon-mobile-phone"
                        @click="handleSaoma('jobNum')">
                    </el-button>
                </el-form-item>
                <el-form-item prop="remember" class="clearMargin">
                    <el-checkbox label="记住设备名" v-model="remember" disabled></el-checkbox>
                </el-form-item>
            </el-form>
        </div>
        <scan-frame v-show="scanShow" ref="scan" @getScan="getScan" @closeScan="closeScan"></scan-frame>
    </div>
</template>
<script>
import Vue from 'vue'
import axios from 'axios'
import ScanFrame from '../utils/ScanFrame'

export default {
    components: { ScanFrame },
    data() {
        return {
            // equipmentsList: [{BM_NAME: 'machine02'},{BM_NAME: 'machine03'}],
            equipmentsList: [],
            // 获取是否林全设备信息保留数据
            isDataLQ: [],
            form: {
                userName: '',
                operator: localStorage.getItem('author') || '',
                jobNum: ''
            },
            // 扫码后返回员工工号和姓名保留，待登录校验用
            rootNum: '',
            staff: '',
            // 表单前端校验
            rules: {
                userName: [
                    {required: true, message: '请选择设备！', tigger: 'blur'}
                ],
                operator: [
                    {required: true, message: '请选择操作项！', tigger: 'blur'}
                ]
            },
            // 是否记住设备名
            remember: true,
            // 重新选择设备显示隐藏
            isShow: false,
            equipmentsOnline: [],
            // 扫描框显示隐藏切换
            scanShow: false
        }
    },
    created() {
        // 获取设备类型
        this.getOpenType()
        // 获取在线设备数组(要先在下面的return前获取到)
        this.getEquipmentOnline()
        // 获取设备名,操作员列表
        this.getEquipments()
    },
    activated() {
        // 扫码后返回页面更改可扫码框的值
        // const result = this.$store.state.scanItems
        // result.map(item => {
        //     if(item.name === 'jobNum') {
        //         // 获取扫码结果
        //         this.form.jobNum = item.value
        //         return
        //     }
        // })
        // if(this.form.jobNum != '') {
        //     // ipad扫码后有值，则发送请求解码出工号和姓名
        //     this.scanLogin()
        // }
    },
    methods: {
        getEquipmentOnline() {
            axios.get(this.killBrowserUrl + 'server/getOnline')
            .then((res) => {
                // console.log(res)
                if(res.data.code === 200) {
                    this.equipmentsOnline = res.data.content
                } 
            }).catch(err => err)
            // 模拟有在线机台，用于测试
            // this.equipmentsOnline = [{name: 'machine02'},{name: 'machine03'}]
        },
        // 登录校验工号是否通过
        submitForm() {
            const {userName, operator, jobNum} = this.form
            this.$refs.form.validate(valid => {
                if(valid) {
                    // 判断已连接设备中是否当前设备
                    const index = this.equipmentsOnline.findIndex(item => item.name === userName)
                    if(index === -1) {
                        this.$message.error('当前机台未连接，请先上线')
                        return
                    }
                    
                    if(jobNum === this.rootNum) { //当前扫码框的值等于扫码后保留工号，才可成功登录
                        // 如果用户勾选了记住设备名则将用户名缓存
                        if(this.remember) {
                            localStorage.setItem('userName', userName)
                            const isUser = this.isDataLQ.find(item => item.BM_NAME === userName)
                            // 缓存是否林全公司机台
                            localStorage.setItem('company', isUser.BM_LQ)
                            // 缓存员工工号和姓名
                            localStorage.setItem('jobNum', jobNum)
                            localStorage.setItem('operator', this.staff)
                        }
                        // 页面离开时清空登录用户名
                        this.emptyDatas()
                        // 权限为0，跳至开始生产的工单页面，否则跳至调机、首末检的工单页面
                        const author = operator
                        // 当前操作权限存入缓存
                        localStorage.setItem('author', author)
                        // 权限0，跳至生产工单页面
                        if(author === '0'){
                            this.$router.push('/home/work_order')
                        }else if(author === '3') { //权限为3，品检审核页面
                            this.$router.push('/home/audit')
                        }else if(author === '1'){
                            this.$router.push('/home/work_order2')
                        }else if(author === '2'){
                            this.$router.push('/home/examworks')
                        }
                    }else{
                        this.$message.error('该用户不存在！请重新输入')
                    }
                }
            })
        },
        // 获取设备名称
        getEquipments() {
            // 获取缓存中的设备名
            const userName = localStorage.getItem('userName')
            // axios.get('http://mengxuegu.com:7300/mock/5e6a16f0e7a1bb0518bb7477/aps/GetUser')
            axios.get(this.httpUrl + 'MES/GetUser')
            .then((res) => {
                // 此设备数据保留，待获取是否林全机台用
                this.isDataLQ = res.data.machine
                // console.log(res)
                if(userName != null) {
                    this.form.userName = userName
                    // 有缓存，显示重新获取设备按钮，此时不获取设备列表直接return出去
                    this.isShow = true
                    return
                }
                // 无缓存，则设备信息渲染用
                this.equipmentsList = res.data.machine
            }).catch(err => err)
        },
        // 扫码登录
        scanLogin() {
            const value = this.form.jobNum.trim()
            if(value === '') {
                this.$message.error('扫描结果为空！')
                return
            }
            // 发送扫码结果到后台验证是否通过，并返回工号和姓名
            axios.get(this.httpUrl + 'MES/GetLogin?uid='+value)
            .then(res => {
                // console.log(res)
                if(res.data.code === 200) {
                    const result = res.data.data[0]
                    if(result.STATUS === '0') {
                        this.form.jobNum = result.EMP_NO
                        this.rootNum = result.EMP_NO
                        this.staff = result.EMP_NAME
                        // 扫码直接判断是否登录
                        this.submitForm()
                    }else{
                        this.$message.error('该用户不存在！')
                    }
                }else{
                    this.$message.error('判断用户是否存在失败！请重试')
                }
            }).catch(err => err)
        },
        // 重新选择设备
        ChangeEquip() {
            // 清缓存中的userName并刷新页面重新获取设备
            localStorage.removeItem('userName')
            localStorage.removeItem('company')
            localStorage.removeItem('operator')
            location.reload()
        },
        emptyDatas() {
            // 清空store中的jobNum
            this.form.jobNum = ''
            this.$store.dispatch('handleEmptyScanItems', '')
        },
        // 下拉框选择后，扫码框一直聚焦
        handleChangeFocus() {
            // 更改当前操作权限存入缓存
            this.$nextTick(() => {
                this.$refs.inputSao.focus()
            })
        },
        handleSaoma(item) {
            // this.$router.push('/device?name='+item)
            this.scanShow = true
            // 开启扫描框后，触发子组件的开始扫描方法
            this.$nextTick(() => {
                this.$refs.scan.handleScan()
            })
        },
        getScan(value) {
            this.form.jobNum = value
            this.scanLogin()
            // this.$nextTick(() => {
            //     this.scanShow = false
            // })
        },
        closeScan() {
            this.scanShow = false
        },
        update() {
            window.location.href = this.httpUrl+'app/appLinQuan.apk'
        },
        // 判断运行环境
        getOpenType() {
            let sUserAgent = navigator.userAgent.toLowerCase();
            let bIsIpad = sUserAgent.match(/ipad/i) == "ipad";//判断是否为iPad
            let bIsIphoneOs = sUserAgent.match(/iphone os/i) == "iphone os";//判断是否为iPhone用户
            let bIsMidp = sUserAgent.match(/midp/i) == "midp";
            let bIsUc7 = sUserAgent.match(/rv:1.2.3.4/i) == "rv:1.2.3.4";
            let bIsUc = sUserAgent.match(/ucweb/i) == "ucweb";
            let bIsAndroid = sUserAgent.match(/android/i) == "android";
            let bIsCE = sUserAgent.match(/windows ce/i) == "windows ce";
            let bIsWM = sUserAgent.match(/windows mobile/i) == "windows mobile";
        
            if (!(bIsIpad || bIsIphoneOs || bIsMidp || bIsUc7 || bIsUc || bIsAndroid || bIsCE || bIsWM)) {
                // console.log("当前是电脑打开")
                this.isKeyboard = true
                localStorage.setItem('isKeyboard', true)
            }else{
                // console.log("当前是手机打开")
                this.isKeyboard = false
                // this.operator = '2'
                localStorage.setItem('isKeyboard', false)
            }
        }
    },
    // 自定义指令输入框自动聚焦
    directives: {
        focus: {
            inserted: function (el) {
                el.querySelector('input').focus()
            }
        }
    }
}
</script>
<style>
    .login{
        position: absolute;
        width: 100%;
        height: 100%;
        top: 0;
        left: 0;
        background: url(../assets/bg.jpg) no-repeat center center;
        background-size: 100% 100%;
    }
    .loginWrap{
        width: 40%;
        position: absolute;
        left: 50%;
        top: 50%;
        transform: translate(-50%, -60%);
    }
    .loginHeader{
        width: 100%;
        display: flex;
        justify-content: center;
        align-items: center;
        font-size: 1rem;
        color:  #0766a1;;
    }
    .loginHeader img{
        width: 100px;
    }
    .form-login{
        background: rgba(0, 0, 0, .3);
        padding: 15px 30px;
        border-radius: 15px;
        box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.2);
    }
    .form-title{
        color: #fff;
        text-align: center;
    }
    .el-select{
        width: 100%;
    }
    .btn_wrap .el-button{
        width: 100%;
    }
    .changeEquip{
        position: absolute;
        right: 0;
        bottom: -20px;
        padding: 2px 5px;
    }
    .vue-touch-keyboard{
        width: 100%;
        position: fixed;
        left: 0;
        bottom: 0;
    }
    .iconTitle{
        margin-left: 2px;
        font-size: 20px;
        font-weight: 700;
        color:rgb(38, 181, 238);
    }
    .clearMargin{
        margin: 5px 0;
    }
    .saoma{
        position: absolute;
        right: 5px;
        top: 6px;
    }
</style>