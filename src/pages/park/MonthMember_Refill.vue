<template>
    <section class="right-wrapper-size shop-table-wrapper" id="scrollBarDom">
        <div class="shop-custom-operation">
            <header class="shop-custom-header">
                <p style="float: left">会员管理<span style="margin: 2px">-</span>月卡续费记录</p>
                <div class="float-right">
                    <el-button type="text" @click="resetForm" icon="el-icon-refresh" style="font-size: 14px;color: #1E1E1E;">刷新</el-button>
                </div>
            </header>
            <div style="display: block;background: #fff">
                <div class="refill-money-wrapper bgcolor-white" style="margin-top: 0;text-align: center">
                    <el-row type="flex" justify="center" align="center">
                        <el-col :span="12" style="border-right: 2px solid #DDDDDD;color: #F5B962;">
                            <div class="refill-money-left">
                                <i class="iconfont icon-jine money-icon" ></i>
                                <p class="refill-month-title">应收金额</p>
                                <p class="refill-month-count" >
                                    <countTo :startVal='0' :endVal='amountReceivable' :decimals="2" :duration='1000'></countTo>元
                                </p>
                            </div>
                        </el-col>
                        <el-col :span="12" style="color: #8693F3">
                            <div class="refill-money-left">
                                <i class="iconfont icon-jine money-icon" ></i>
                                <p class="refill-month-title">实收金额</p>
                                <p class="refill-month-count">
                                    <countTo :startVal='0' :endVal='actReceivable' :decimals="2" :duration='1000'></countTo>元
                                </p>
                            </div>
                        </el-col>
                    </el-row>
                </div>
            </div>
            <div class="shop-custom-console">
                <el-form :inline="true" :model="searchFormData" class="shop-custom-form-search">
                    <div class="advanced-options" v-show="isShow">
                        <el-form-item label="月卡编号" class="clear-style">
                            <el-input v-model="searchFormData.card_id" placeholder="" class="shop-custom-input"></el-input>
                        </el-form-item>
                        <el-form-item label="收费员" class="clear-style">
                            <el-input v-model="searchFormData.collector" placeholder="" class="shop-custom-input"></el-input>
                        </el-form-item>
                        <el-form-item label="购买流水号" class="clear-style">
                            <el-input v-model="searchFormData.trade_no" placeholder="" class="shop-custom-input"></el-input>
                        </el-form-item>
                    </div>
                    <div class="console-main">
                        <el-form-item>
                            <el-select v-model="searchFormData.time_type" placeholder="请选择" class="shop-custom-input shop-custom-suffix" style="width: 100px">
                                <el-option
                                        v-for="item in timerType"
                                        :key="item.value_no"
                                        :label="item.value_name"
                                        :value="item.value_no">
                                </el-option>
                            </el-select>
                            <el-date-picker
                                    style="width: 350px"
                                    class="shop-custom-datepicker"
                                    v-model="searchFormData.currentData"
                                    type="datetimerange"
                                    range-separator="至"
                                    :default-time="['00:00:00','23:59:59']"
                                    start-placeholder="请输入时间"
                                    end-placeholder="请输入时间"
                                    value-format="timestamp"
                                    @change="changeDateFormat"
                            >
                            </el-date-picker>
                        </el-form-item>
                        <el-form-item label="车牌号" class="clear-style margin-left-20">
                            <el-input v-model="searchFormData.car_number" placeholder="请输入车牌号" class="shop-custom-input"></el-input>
                        </el-form-item>
                        <el-form-item class="shop-clear-style">
                            <el-button type="primary" @click="searchFn" icon="el-icon-search">搜索</el-button>
                            <el-button type="text"
                                       @click="changeMore"
                                       style="color:#3C75CF;font-size: 16px;"><img :src="isShow ?offimg:noimg" style="display: inline-block;vertical-align: text-top"> 高级搜索</el-button>
                        </el-form-item>
                        <div class="float-right">
                            <el-form-item class="shop-clear-style">
                                <!--<el-button type="primary" @click="registerMember" v-if="showCustomizeAdd">注册会员</el-button>-->
                                <el-button type="primary"  @click="exportFn" native-type="button" v-if="!hideExport">导出</el-button>
                            </el-form-item>
                        </div>
                    </div>

                </el-form>
            </div>
        </div>

        <div class="table-wrapper-style">
            <tab-pane
                    :queryapi="queryapi"
                    :exportapi="exportapi"
                    :orderfield="orderfield"
                    :fieldsstr="fieldsstr"
                    :table-items="tableitems"
                    align-pos="right"
                    bts-width="200"
                    :searchForm="searchForm"
                    fixedDom="scrollBarDom"
                    ref="tabPane"
                    v-on:transferData="transferDataFn"
            ></tab-pane>
        </div>
    </section>
</template>


<script>
    import {path, checkURL, checkUpload, checkNumber, payType} from '../../api/api';
    import util from '../../common/js/util'
    import common from '../../common/js/common'
    import {AUTH_ID} from '../../common/js/const'
    import TabPane from '../../components/table/TabPane';
    import countTo from 'vue-count-to';
    export default {
        components: {
            TabPane,countTo
        },
        data() {
            return {
                noimg:require('../../assets/images/no.png'),
                offimg:require('../../assets/images/off.png'),
                hideExport:false,
                searchForm:{},
                searchFormData:{
                    time_type:'3',
                    currentData:'',
                    ctime:'between',
                    ctime_start:'',
                    ctime_end:'',
                    car_number:'',
                    name:'',
                    card_id:'',
                    collector:'',
                    trade_no:''
                },
                isShow:false,
                actReceivable:0,
                amountReceivable:0,
                queryapi: '/carrenew/query',
                exportapi: '/carrenew/exportExcel',
                orderfield:'id',
                fieldsstr: 'id__trade_no__card_id__pay_time__amount_receivable__amount_pay__collector__pay_type__car_number__user_id__limit_time__resume__start_time',
                tableitems: [
                    {
                        hasSubs: false,
                        subs: [{
                            label: '',
                            nameType:'month-member-refill',
                            columnType:'expand',
                            align: 'center',
                            width:'50',
                        }]
                    },
                    {

                        hasSubs: false,
                        subs: [{
                            label: '月卡编号',
                            prop: 'card_id',
                            width: '100',
                            type: 'str',
                            editable: true,
                            searchable: true,
                            addable: true,
                            unsortable: true,
                            align: 'center'
                        }]
                    },
                    {

                        hasSubs: false,
                        subs: [{
                            label: '车牌号码',
                            prop: 'car_number',
                            width: '145',
                            type: 'str',
                            editable: true,
                            searchable: true,
                            addable: true,
                            unsortable: true,
                            align: 'center'
                        }]
                    },
                    {

                        hasSubs: false,
                        subs: [{
                            label: '购买流水号',
                            prop: 'trade_no',
                            width: '200',
                            type: 'str',
                            editable: true,
                            searchable: true,
                            addable: true,
                            unsortable: true,
                            align: 'center'
                        }]
                    },
                    {

                        hasSubs: false,
                        subs: [{
                            label: '缴费类型',
                            prop: 'pay_type',
                            width: '120',
                            type: 'str',
                            editable: true,
                            searchable: true,
                            addable: true,
                            unsortable: true,
                            align: 'center',
                        }]
                    },
                    {

                        hasSubs: false,
                        subs: [{
                            label: '实收金额',
                            prop: 'amount_pay',
                            width: '120',
                            type: 'str',
                            editable: true,
                            searchable: true,
                            addable: true,
                            unsortable: true,
                            align: 'center'
                        }]
                    },
                    {

                        hasSubs: false,
                        subs: [{
                            label: '月卡续费时间',
                            prop: 'pay_time',
                            width: '180',
                            type: 'date',
                            editable: true,
                            searchable: true,
                            addable: true,
                            unsortable: true,
                            align: 'center',
                            columnType:'render',
                            render: (h, params) => {
                                return h('div', [
                                    h('span', common.dateformat(params.row.pay_time))
                                ]);
                            }
                        }]
                    },
                    {
                        hasSubs: false,
                        subs: [{
                            label: '备注',
                            prop: 'resume',
                            type: 'str',
                            editable: false,
                            searchable: true,
                            addable: true,
                            unsortable: true,
                            align: 'center'
                        }]
                    },
                ],
                timerType:[
                    {
                        'value_name':'开始时间',
                        'value_no':'1'
                    },
                    {
                        'value_name':'结束时间',
                        'value_no':'2'
                    },
                    {
                        'value_name':'续费时间',
                        'value_no':'3'
                    },
                ],
            }
        },
        methods:{
            transferDataFn(val){
                console.log('val--------',val)
                this.actReceivable = val.actReceivable;
                this.amountReceivable = val.amountReceivable;
            },
            changeMore(){
                this.isShow = !this.isShow
            },
            searchFn() {
                /*
                * 点击搜索后，克隆一份表单数据进行查询，以触发table的查询事件
                * */
                let sform = this.searchFormData;
                sform.state_start = sform.state;
                sform.pay_type_start = sform.pay_type;
                this.searchForm = JSON.parse(JSON.stringify( sform ))
            },
            changeDateFormat(val){
                if(val == null){
                    this.searchFormData.ctime_start = '';
                    this.searchFormData.ctime_end = ''
                }else{
                    this.searchFormData.ctime_start = val[0];
                    this.searchFormData.ctime_end = val[1]
                }
            },
            resetForm(){
                this.initFn(this)
            },
            exportFn(){
                /*
                * 导出数据，通过ref 进行定位拉取
                * */
                this.$refs['tabPane'].handleExport()
            },
            initFn(that){
                /*
                * 初始化操作
                * 点击刷新时 和初进入页面时
                * */
                that.searchFormData ={
                    time_type:'3',
                    currentData:'',
                    ctime:'between',
                    ctime_start:'',
                    ctime_end:'',
                    car_number:'',
                    name:'',
                    card_id:'',
                    pid:''
                };
                that.searchForm = JSON.parse(JSON.stringify( that.searchFormData ));
            },
            setAuthorityFn(){
                let user = sessionStorage.getItem('user');
                if (user) {
                    user = JSON.parse(user);
                    for (var item of user.authlist) {
                        if (AUTH_ID.monthMember_Refill == item.auth_id) {
                            this.hideExport = !common.showSubExport(item.sub_auth);
                            break;
                        }
                    }

                }

            }
        },
        beforeMount(){

        },
        mounted() {
            this.setAuthorityFn()
            this.$refs['tabPane'].getTableData({},this);
        },
    }

</script>

<style scoped lang="scss">
    .refill-money-wrapper{
        padding:0 0 20px 0;
        margin: 0 20px;
        border-bottom: 1px dotted #979797;
    }
    .refill-money-left{
        display: inline-block;
        width: 250px;
        height: 60px;
        text-align: left;
    }
    .money-icon{
        float: left;
        font-size: 60px;
        margin-right: 26px;
    }
    .refill-month-title{
        font-size: 14px;
        font-weight:400;
        color:#666;
        padding-top: 9px;
    }
    .refill-month-count{
        font-size: 24px;
        font-weight: 400;
    }

</style>

