<template>
    <div class="examine">
        <el-page-header @back="goBack" content="品检表单"></el-page-header>
        <div class="exitBut">
            <exit-btn :isFullscreen="isfullScreen"></exit-btn>
        </div>
        <el-card>
            <el-form ref="form" :model="form" label-width="70px">
                <el-row :gutter="12">
                    <el-col :span="6" >
                        <el-form-item label="工号：">
                            <el-input size="mini" :value="form.workId" :disabled="true"></el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6" >
                        <el-form-item label="计划日期：">
                            <el-input size="mini" :value="form.datePlan" :disabled="true"></el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6" >
                        <el-form-item label="机台：">
                            <el-input size="mini" :value="form.machine" :disabled="true"></el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6" >
                        <el-form-item label="制令：">
                            <el-input size="mini" :value="form.orderVal" :disabled="true"></el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6" >
                        <el-form-item label="成品编号：">
                            <el-input size="mini" :value="form.productNum" :disabled="true"></el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6" >
                        <el-form-item label="成品名称：">
                            <el-input size="mini" :value="form.productName" :disabled="true"></el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="12" >
                        <el-form-item label="成品规格：">
                            <el-input size="mini" :value="form.productType" :disabled="true"></el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6" >
                        <el-form-item label="客户编号：">
                            <el-input size="mini" :value="form.client" :disabled="true"></el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6" >
                        <el-form-item label="图纸日期：">
                            <el-input size="mini" :value="form.dateDraw" :disabled="true"></el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6" >
                        <el-form-item label="图纸版次：">
                            <el-input size="mini" :value="form.versionDraw" :disabled="true"></el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6" >
                        <el-form-item>
                            <el-button size="mini" type="primary">查看图纸</el-button>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6" >
                        <el-form-item label="工序序号：">
                            <el-select v-model="form.processNum" @change="processChange" size="mini" placeholder="请选择">
                                <el-option
                                v-for="item in form.processData"
                                :key="item.WS_IDX"
                                :value="item.WS_IDX">
                                </el-option>
                            </el-select>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6" >
                        <el-form-item label="工序名称：">
                            <el-input size="mini" :value="form.processName" :disabled="true"></el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="12" >
                        <el-form-item label="工序描述：">
                            <el-input size="mini" :value="form.processDesc" :disabled="true"></el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6" class="margin-top">
                        <el-form-item label="实物编号：">
                            <el-input size="mini" v-model="identifier"></el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="18" >
                        <el-form-item label="备注：">
                            <el-input type="textarea" size="mini" :rows="1" v-model="remark"></el-input>
                        </el-form-item>
                    </el-col>
                </el-row>
            </el-form>
        </el-card>
        <el-table :data="tableData" ref="examineRef" size="mini" border style="width: 100%">
            <el-table-column type="index" width="40"></el-table-column>
            <el-table-column prop="CATE_NAME" label="品检类别"></el-table-column>
            <el-table-column prop="QC_ITEM" label="品检项目"></el-table-column>
            <el-table-column prop="WL_SPEC" label="规格" :show-overflow-tooltip="true"></el-table-column>
            <el-table-column prop="QC_REMARK" label="品检备注" width="100"></el-table-column>
            <el-table-column prop="TOOL_NAME" label="判定工具" width="100"></el-table-column>
            <el-table-column prop="INPUT_MODE" label="录入方式" width="100"></el-table-column>
            <el-table-column label="标准值" width="100">
                <template slot-scope="scope">
                    {{scope.row.IS_SHOW_BZ==='0' ? '###' : scope.row.QC_VALUE}}
                </template>
            </el-table-column>
            <el-table-column label="检验值" width="100">
                <template slot-scope="scope">
                    <el-select 
                        v-if="scope.row.CHECK_MODE === 'Y/N'" 
                        size="mini" 
                        v-model="scope.row.INPUTVAL" 
                        @change="getStatus(scope.row, scope.$index)"
                        placeholder="">
                        <el-option label="Y" value="Y"></el-option>
                        <el-option label="N" value="N"></el-option>
                    </el-select>
                    <el-input v-else v-model="scope.row.INPUTVAL" size="mini" @input="getStatus(scope.row, scope.$index)"/>
                </template>
            </el-table-column>
            <el-table-column prop="STATUS" label="状态" width="60">
                <template slot-scope="scope">
                    <el-tag size="mini" 
                        v-if="scope.row.STATUS != ''"
                        :type="scope.row.STATUS === 'OK' ? 'success' : 'danger'">
                        {{scope.row.STATUS}}
                    </el-tag>
                </template>
            </el-table-column>
        </el-table>
        <div class="btnwrap">
            品检结果：
            <template>
                <el-radio v-model="resExamine" label="Y" :disabled="true">通过</el-radio>
                <el-radio v-model="resExamine" label="N" :disabled="true">不通过</el-radio>
            </template>
            <el-button size="mini" type="success" :disabled="isAllCheck" @click="resultSubmit">提交</el-button>
        </div>
    </div>
</template>
<script>
import ExitBtn from './ExitButton'
import getCurrentTime from '../utils/currentTime'
import axios from 'axios'
import {machines, infos, products} from './examine.json'

export default {
    components: { ExitBtn },
    data() {
        return {
            userName: localStorage.getItem('userName'),
            // 品检数据列表
            tableData: [],
            form: {
                workId: localStorage.getItem('operator'),
                datePlan: this.$route.query.date,
                machine: localStorage.getItem('userName'),
                orderVal: this.$route.query.work,
                productNum: '',
                productName: '',
                productType: '',
                client: '',
                dateDraw: '',
                versionDraw: '',
                // 工序序号下拉列表数据
                processData: [],
                // 工序序号
                processNum: '',
                processName: '',
                processDesc: ''
            },
            // 品检表单备注信息
            remark: '',
            // 实物编号
            identifier: '',
            // 品检最终是否通过结果
            resExamine: 'N',
            isFull: false,
            isfullScreen: false,
            timer: null,
            isAllCheck: true
        }
    },
    created() {
        this.getProductInfo()
    },
    mounted() {
        window.onresize = () => {
            this.$nextTick(() => {
                this.isFull = document.fullscreenElement || 
                document.msFullscreenElement || 
                document.mozFullScreenElement ||
                document.webkitFullscreenElement || false
            })
        }
    },
    methods: {
        getProductInfo(){
            // 用工单号查询成品信息和成品工序
            axios.get(' http://mengxuegu.com:7300/mock/5ea245bd2a2f716419f892c5/GetProductend?wo='+this.form.orderVal)
            // axios.get(this.httpUrl + 'MES/GetProductend?wo='+this.form.orderVal)
            .then(res => {
                // console.log(res)
                if(res.status === 200) {
                    const {ITEM_CODE, ITEM_NAME, ITEM_SPEC, CUST_CODE, CAD_DATE, CAD_VER} = res.data.V_MES_PRODUCT[0]
                    const processArr = res.data.v_mes_gongxu
                    this.form.productNum = ITEM_CODE
                    this.form.productName = ITEM_NAME
                    this.form.productType = ITEM_SPEC
                    this.form.client = CUST_CODE
                    this.form.dateDraw = CAD_DATE.replace(/-/g, '/')
                    this.form.versionDraw = CAD_VER
                    // 工序序号列表渲染
                    this.form.processData = processArr
                    // 默认工序序号、工序名称、工序描述选中第一道工序
                    this.form.processNum = processArr[0].WS_IDX
                    // this.form.processName = processArr[0].GX_NAME
                    // this.form.processDesc = processArr[0].WS_DESC
                    // 第一次选中制令号时，默认获取第一工序号的品检信息渲染
                    this.getProcessInfo(this.form.processNum)
                }else{
                    this.$message.error('获取成品信息失败！')
                }
            }).catch(err => err)
        },
        dateValueChange() {
            this.getOrderList(this.form.datePlan)
        },
        goBack() {
            this.$router.go(-1)
        },
        getStatus(row, index) {
            // console.log(row)
            // 节流
            if(this.timer) {
                clearTimeout(this.timer)
            }
            this.timer = setTimeout(() => {
                // 校验检验值是否通过
                const {CHECK_MODE, IS_SHOW_BZ, QC_VALUE, QC_DIFF, INPUTVAL} = row
                let value = ''
                if(INPUTVAL != ''){
                    switch(CHECK_MODE) {
                        case 'Y/N':
                            if(INPUTVAL === 'Y'){
                                value = 'OK'
                            }else if(INPUTVAL === 'N'){
                                value = 'NG'
                            }
                            break
                        case '大于等于':
                            value = INPUTVAL >= QC_VALUE ? 'OK' : 'NG'
                            break
                        case '介于等于':
                            const nums = QC_VALUE.split('-')
                            value = (INPUTVAL-0 >= nums[0]-0 && INPUTVAL-0 <= nums[1]-0) ? 'OK' : 'NG'
                            break
                        case '不判定':
                            value = 'OK'
                            break
                    }
                }
                this.tableData[index].STATUS = value
                this.isAllExamine()
            }, 800)
        },
        isAllExamine() {
            // console.log(this.tableData)
            // 工序值变化后，重新获取数据并渲染
            const allCheckflag = this.tableData.findIndex(item => item.STATUS === '')
            if(allCheckflag === -1){
                this.isAllCheck = false
                // 当全部检验后判断最终结果是通过与否
                const isOK = this.tableData.findIndex(item => item.STATUS != 'OK')
                this.resExamine = (isOK === -1) ? 'Y': 'N'
            }else{
                this.isAllCheck = true
            }
        },
        handleSelect(item) {
            // console.log(item)
            this.getProductInfo(item.value)
        },
        processChange(newVal) {
            // 根据工序序号查询对应的工序名称、工序描述展示出来
            // console.log(newVal)
            this.getProcessInfo(newVal)
        },
        getProcessInfo(num) {
            // 制令选择新值后重新获取成品信息和成品工序
            this.form.processData.find(item => {
                if(item.WS_IDX === num) {
                    this.form.processName = item.GX_NAME
                    this.form.processDesc = item.WS_DESC
                    return true
                }
            })
            // 根据成品编号和工序号查询出需要品检项目的列表
            axios.get('http://mengxuegu.com:7300/mock/5ea245bd2a2f716419f892c5/GetProductendGx?Productend=V0U0600770A&gx=01')
            // axios.get(this.httpUrl+ 'MES/GetProductendGx?Productend='+ this.form.productNum + '&gx='+num)
            .then(res => {
                // console.log(res)
                if(res.status === 200) {
                    // 提取品检数据
                    let examineArr = res.data
                    // 每组数据加上输入框值和状态属性
                    examineArr.map(item => {
                        // console.log(item)
                        item.INPUTVAL = '',
                        item.STATUS = ''
                    })
                    // 再把新数据赋值给渲染表格的数据
                    this.tableData = examineArr
                }else{
                    this.$message.error('获取品检列表失败！')
                }
            }).catch(err => err)
        },
        resultSubmit() {
            let formData = {}
            formData.QP_USERID = this.userName
            formData.QP_WORKER = this.form.orderVal
            formData.QP_PLAN_DATE = this.form.datePlan
            formData.QP_MACHINE = this.form.machine
            formData.ITEM_CODE = this.form.productNum
            formData.ITEM_NAME = this.form.productName
            formData.ITEM_SPEC = this.form.productType
            formData.CUST_CODE = this.form.client
            formData.CAD_DATE = this.form.dateDraw
            formData.CAD_VER = this.form.versionDraw
            formData.WS_IDX = this.form.processNum
            formData.GX_NAME = this.form.processName
            formData.WS_DESC = this.form.processDesc
            formData.identifier = this.identifier
            formData.remark = this.remark
            formData.resExamine = this.resExamine
            formData.WS_DATA = this.tableData
            // console.log(formData)
            axios.post(this.httpUrl + 'MES/QCProductend', formData)
            .then(res => {
                if(res.status === 200){
                    this.$message.success('保存品检结果成功！')
                }else {
                    this.$message.error('保存品检结果失败！请重试')
                }
            }).catch(err => err)
        },
        querySearch(queryString, cb) {
            var restaurants = this.form.optionsOrder
            var results = queryString ? restaurants.filter(this.createFilter(queryString)) : restaurants
            // 调用 callback 返回建议列表的数据
            cb(results)
        },
        createFilter(queryString) {
            return (restaurant) => {
            return (restaurant.value.toLowerCase().indexOf(queryString.toLowerCase()) != -1)
            }
        }
    },
    watch: {
        // 监听是否全屏，val为false时，当前不是全屏，点击后显示全屏按钮为true。否则为false变为非全屏按钮
        isFull(val) {
            if(val === false) {
                this.isfullScreen = false
            }else{
                this.isfullScreen = true
            }
        }
  }
}
</script>
<style scoped>
.examine{
    padding-bottom: 40px;
}
/deep/ .el-form-item__label{
    font-size: 12px;
}
/deep/ .el-card__body{
    padding: 12px;
}
.el-date-editor{
    width: 100%;
}
/deep/.el-form-item__label{
    padding-right: 2px;
}
.el-form-item{
    margin-bottom: 2px;
}
/deep/ .el-input__inner{
    padding: 0 6px;
}
.margin-top{
    margin-top: 5px;
}
/deep/.el-table .cell{
    padding-left: 5px;
    padding-right: 5px;
}
/deep/ .el-table .el-input__inner{
    padding: 0 5px;
}
/deep/ .el-autocomplete-suggestion li{
    padding: 0 15px;
    font-size: 12px;
}
.el-select-dropdown__item{
    padding: 0 10px;
    font-size: 12px;
}
/deep/ .el-input__suffix{
    right: -2px;
}
.el-radio{
    color: #fff;
}
.btnwrap{
    width: 100%;
    height: 40px;
    line-height: 40px;
    text-align: center;
    background: gray;
    color: #fff;
    z-index: 10;
    position: fixed;
    bottom: 0;
    right: 0;
}
</style>