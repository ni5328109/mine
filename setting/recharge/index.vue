<template>
  <div class="msg-recharge-page">
    <title :name="'短信充值'"></title>
    <scroll-view class="msg-recharge-scroll" scroll-y="true"
                 :style="{height:'calc(100vh - '+titleHeight +'px)'}">
      <div class="msg-recharge-main" :style="{'min-height':'calc(90vh - '+titleHeight +'px)'}">
        <div class="msg-recharge-context">
          <div class="recharge-item" v-for="item in items" :key="index" @click="selectItem(item)">
            <img v-if="item.Id == currentItem.Id"
                 src="https://linkfit-pro.oss-cn-hangzhou.aliyuncs.com/Business/static/recharge-check.png"/>
            <img v-if="item.Id != currentItem.Id"
                 src="https://linkfit-pro.oss-cn-hangzhou.aliyuncs.com/Business/static/recharge-uncheck.png"/>
            <span>{{item.ItemQuantity}}条&nbsp;/&nbsp;{{item.Price}}元</span>
          </div>
          <div class="rechagre-remark">
            <span>*</span>
            提示: 已充值短信不支持退款，请选择适合的套餐
          </div>
          <div class="recharge-info" @click="accept=!accept">
            <span>
              <img v-if="accept" src="https://linkfit-pro.oss-cn-hangzhou.aliyuncs.com/Business/static/checked.png"/>
              <span v-if="!accept" class="unaccept" style="width: 4.4vw;height: 4.4vw;padding: 0;"></span>
            </span>
            <span>
              <div>根据运营商规则每70个字符占用一条短信数量，超过70字符按2条</div>
              <div>短信扣除，以此类推</div>
              <div>*请阅读短信扣除规则，并勾选</div>
            </span>
          </div>
          <div class="recharge-button">
            <span @click="doRecharge" :style="{background:accept?'':'#f0f0f0'}">确定充值</span>
          </div>
        </div>
      </div>
      <div class="demo-footer" style="padding-top: 0vh">
        <img class="demo-nutcards" src="https://linkfit-pro.oss-cn-hangzhou.aliyuncs.com/Business/static/nutcards.png"/>
      </div>
      <div class="demo-bottom"></div>
    </scroll-view>
  </div>
</template>

<script>
  import title from '@/components/title'

  export default {
    components: {
      title
    },

    data () {
      return {
        items: [],
        currentItem: {},
        accept: false,
        titleHeight: null
      }
    },
    methods: {
      selectItem (item) {
        this.currentItem = item
      },
      doRecharge () {
        if (!this.accept) {
          return
        }
        let that = this
        this.$post('/smsCenter/buySmsAddedService', {
          Uid: this.userId,
          StoreId: this.storeId,
          AddedServiceId: this.currentItem.Id
        }, true).then(res => {
          that.weixinPay(res.OrderId)
        })
      },
      weixinPay (id) {
        let that = this
        this.$post('/pay/getBusinessPayParam', {
          OrderId: id,
          PayType: 0,
          OrderType: 2,
          StoreId: this.storeId,
          Uid: this.userId
        }, true).then(result => {
          let res = JSON.parse(result.ParmJson)
          wx.requestPayment({
            timeStamp: res.timeStamp,
            nonceStr: res.nonceStr,
            package: res.package,
            signType: res.signType,
            paySign: res.paySign,
            success: function () {
              that.$success('订购成功', true)
            }
          })
        })
      },
      getItems () {
        let that = this
        this.$post('/smsCenter/getSmsAddedServices', {
          Uid: this.userId,
          StoreId: this.storeId
        }, true).then(res => {
          if (res.SmsAddedServiceList && res.SmsAddedServiceList.length > 0) {
            that.currentItem = res.SmsAddedServiceList[0]
          }
          that.items = res.SmsAddedServiceList
        })
      }
    },
    onLoad (option) {
      this.userId = option.userId
      this.storeId = option.storeId
      this.items = []
      this.currentItem = {}
      this.accept = false
      this.getItems()
      this.titleHeight = this.getGlobalData().titleHeight
    }
  }
</script>

<style lang="scss">
  @function rpx($value) {
    @return $value * 1rpx;
  }

  .msg-recharge-page {
    height: 100vh;
    background-color: #f0f0f0;
    .msg-recharge-scroll {
      height: 90vh;
      .msg-recharge-main {
        min-height: 80vh;
        background: white;
        .msg-recharge-context {
          padding: 10vw 12.4vw;
          .recharge-item {
            height: 4.6vw;
            line-height: 4.6vw;
            padding: 1.7vw 0;
            img {
              width: 4.6vw;
              height: 4.6vw;
              display: inline-block;
              vertical-align: top;
            }
            span {
              display: inline-block;
              vertical-align: top;
              padding-left: 2.8vw;
              height: 4.6vw;
              line-height: 4.6vw;
              color: #898989;
              font-size: 3.7vw;
            }
          }
          .rechagre-remark {
            padding-top: 6.5vw;
            color: #C48F21;
            font-size: 3.3vw;
            padding-bottom: 3.9vw;
            span {
              color: #FC493A;
            }
          }
          .recharge-info {
            width: 100vw;
            position: relative;
            box-sizing: border-box;
            background: #F0F0F0;
            left: -12.4vw;
            font-size: 2.8vw;
            img {
              width: 4.4vw;
              height: 4.4vw;
            }
            span {
              display: inline-block;
              vertical-align: top;
              &:nth-child(1) {
                width: 4.4vw;
                padding: 5.5vw 3.1vw 0 5.7vw;
              }
              &:nth-child(2) {
                width: 85vw;
              }
            }
            .unaccept {
              width: 4.4vw !important;
              height: 4.4vw !important;
              border-radius: 50%;
              box-sizing: border-box;
              border: 0.3vw solid rgba(192, 192, 192, 0.51);
            }
            div {
              line-height: 2.8vw;
              &:nth-child(1) {
                padding-top: 2.7vw;
              }
              &:nth-child(2) {
                padding-top: 1.4vw;
              }
              &:nth-child(3) {
                padding-top: 1.4vw;
                padding-bottom: 2.7vw;
                color: #FC493A;
              }
            }
          }
          .recharge-button {
            padding-top: 14vw;
            text-align: center;
            span {
              display: inline-block;
              width: 48.8vw;
              height: 11.1vw;
              line-height: 11.1vw;
              background: rgba(120, 188, 109, 1);
              border-radius: 5.5vw;
              color: white;
              font-size: 4.3vw;
            }
          }
        }
      }
    }
  }
</style>
