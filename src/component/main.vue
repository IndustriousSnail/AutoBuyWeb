<template>
    <div class="main">
        <div v-show="showLoginDiv">
            <el-image
                    style="display: block; width: 800px; height: 600px; margin: 0 auto; overflow-x: scroll; overflow-y: scroll"
                    :src="qrUrl"
                    v-show="showQrImg"
                    fit="none"></el-image>
            <div v-show="showQrImg">
                <el-tooltip class="item" effect="dark" content="若二维码显示不全，可到该地址查看二维码图片" placement="top-start">
                    <el-link :href="qrUrl" target="_blank">二维码地址：{{ qrUrl }}</el-link>
                </el-tooltip>
            </div>
            <el-button @click="qrLogin" v-show="showLoginButton" :loading="qrLoginButtonLoading" type="primary">{{
                loginButtonText }}
            </el-button>
            <el-button @click="finishScanQr" v-show="showQrButton" type="primary">我已扫码</el-button>
        </div>
        <el-input placeholder="请输入商品地址，如：https://item.jd.com/100003489399.html" v-model="goodsUrl"
                  v-show="showGoodsUrlInput">
            <el-button slot="append" @click="enterGoods" :loading="goodsLoading">确认</el-button>
        </el-input>
        <div v-show="showGoodsCard">
            <el-card class="box-card">
                <div slot="header" class="clearfix">
                    <span>{{ goods.name }}</span>
                </div>
                <div class="text item">
                    {{'价格： ' + goods.price }}
                </div>
                <div class="text item">
                    {{'是否有货： ' + goods.in_stock }}
                </div>
                <div class="text item">
                    {{'配送地： ' + goods.delivery_place }}
                </div>
            </el-card>
            <el-card class="box-card">
                <div slot="header" class="buy-condition-header">
                    <span>购买条件，至少选择一项</span>
                    <el-button style="float: right; padding: 8px 14px; font-size: 11px;"
                               @click="startBuy"
                               :loading="buyLoading"
                               type="primary">开始抢购
                    </el-button>
                </div>
                <div class="buy-condition">
                    <el-input
                            placeholder="小于等于该价格时购买"
                            v-model="buyPrice">
                    </el-input>
                </div>
                <div class="buy-condition">
                    <el-switch
                            v-model="buyWhenStock"
                            active-text="有货时购买"
                    >
                    </el-switch>
                </div>
                <div class="buy-condition">
                    <el-date-picker
                            v-model="buyTime"
                            type="datetime"
                            :editable="false"
                            format="yyyy-MM-dd HH:mm:ss"
                            placeholder="选择秒杀开始时间">
                    </el-date-picker>
                </div>
            </el-card>

        </div>
    </div>
</template>

<script>
    import {Loading} from 'element-ui';
    import axios from 'axios';
    import utils from "../utils/utils"

    export default {
        name: "SnailMain",
        methods: {
            qrLogin() {
                const that = this;
                this.qrLoginButtonLoading = true;
                this.loginButtonText = "正在获取登录二维码";
                axios.post("/auto_buy/login_qr").then(function (resp) {
                    if (resp.data.returnCode === 0) {
                        that.qrUrl = "/auto_buy/get_qr_img"
                        that.showQrImg = true;
                        that.showQrButton = true;
                        that.showLoginButton = false;
                        that.qrLoginButtonLoading = false;
                        that.loginButtonText = "扫码登录";
                    } else {
                        that.$notify.error({
                            title: '获取二维码失败！',
                            message: resp.data.returnMessage
                        });
                        that.qrLoginButtonLoading = false;
                        that.loginButtonText = "重新获取登录二维码";
                    }
                });
                // this.showLoginDiv = false;
                // this.showGoodsUrlInput = true;
            },
            enterGoods() {
                const that = this;
                this.goodsLoading = true;
                axios.post("/auto_buy/get_goods_info", {
                    goodsUrl: this.goodsUrl
                }).then(function (resp) {
                    if (resp.data.returnCode === 0) {
                        that.goods = resp.data.beans;
                        that.showGoodsCard = true;
                    } else {
                        that.$notify.error({
                            title: '获取商品信息失败！',
                            message: resp.data.returnMessage
                        });
                    }
                    that.goodsLoading = false;
                });
            },
            finishScanQr() {
                const that = this;
                // 获取登录用户
                axios.post("/auto_buy/finish_scan_qr").then(function (resp) {
                    if (resp.data.returnCode === 0) {
                        that.showLoginDiv = false;
                        that.showGoodsUrlInput = true;
                        that.$notify({
                            title: '登录成功！',
                            message: resp.data.returnMessage,
                            type: 'success'
                        });
                    } else {
                        that.$notify.error({
                            title: '登录失败！',
                            message: resp.data.returnMessage
                        });
                        that.showQrImg = false;
                        that.showLoginButton = true;
                        that.showQrButton = false;
                    }
                });
            },
            startBuy() {
                const that = this;
                let price = null;
                if (this.buyPrice && this.buyPrice.trim() !== "") {
                    if (utils.isNumber(this.buyPrice)) {
                        price = parseFloat(this.buyPrice);
                    } else {
                        this.$message.error('输入的抢购价格不是数字');
                        return;
                    }
                } else {
                    price = undefined;
                }

                let buyTime = null;
                if (this.buyTime) {
                    buyTime = Date.parse(this.buyTime)/1000;
                }else {
                    buyTime = undefined;
                }


                this.buyLoading = true;
                axios.post("/auto_buy/start_rush_buy", {
                    buyTime: buyTime,
                    price: price,
                    inStock: this.buyWhenStock
                }).then(function (resp) {
                    if (resp.data.returnCode === 0) {
                        that.$message({
                            message: '抢购开始',
                            type: 'success'
                        });
                    } else {
                        that.$notify.error({
                            title: '抢购开始失败！',
                            message: resp.data.returnMessage
                        });
                        that.buyLoading = false;
                    }
                });
            },
        },
        mounted: function () {
            const that = this;
            const initOptions = {
                text: "正在初始化...",
                background: 'rgba(0, 0, 0, 0.3)'
            };
            let loadingInstance = Loading.service(initOptions);
            loadingInstance.close();
            this.showLoginDiv = true;

            axios.post("/auto_buy/get_logined_username").then(function (resp) {
                if (resp.data.returnCode === 0) {
                    that.showLoginDiv = false;
                    that.showGoodsUrlInput = true;
                    that.$notify({
                        title: '登录成功！',
                        message: resp.data.returnMessage,
                        type: 'success'
                    });
                } else if (resp.data.returnCode === 404) {
                    that.showQrImg = false;
                    that.showLoginButton = true;
                    that.showQrButton = false;
                    that.showLoginDiv = true;
                } else {
                    that.$notify.error({
                        title: '登录失败！',
                        message: resp.data.returnMessage
                    });
                    that.showQrImg = false;
                    that.showLoginButton = true;
                    that.showQrButton = false;
                }
            });
            // todo
            // axios.post("/auto_buy/init").then(function () {
            //     loadingInstance.close();
            //     that.showLoginDiv = true;
            // }).catch(function (error) {
            //     loadingInstance.close();
            //     that.$notify.error({
            //         title: '错误',
            //         message: '初始化失败！'
            //     });
            // })
        },
        data() {
            return {
                showLoginButton: true,
                showQrButton: false,
                showLoginDiv: false,
                showGoodsUrlInput: false,
                qrLoginButtonLoading: false,
                loginButtonText: "扫码登录",
                goodsUrl: '',
                goodsLoading: false,
                buyLoading: false,
                showGoodsCard: false,
                goods: {},
                buyTime: null,
                buyPrice: null,
                buyWhenStock: null,
                qrUrl: null,
                showQrImg: false
            }
        }
    }
</script>

<style scoped>
    .main {
        width: 100%;
        height: 100%;
        /*background-color: cadetblue;*/
    }

    .text {
        font-size: 14px;
    }

    .item {
        margin-bottom: 18px;
    }

    .clearfix:before,
    .clearfix:after {
        display: table;
        content: "";
    }

    .clearfix:after {
        clear: both
    }

    .box-card {
        width: 100%;
        margin-top: 20px;
        text-align: left;
    }

    .buy-condition-header {
        height: 30px;
        line-height: 30px;
    }

    .buy-condition {
        height: 50px;
        line-height: 50px;
    }
</style>