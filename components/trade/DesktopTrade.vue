<template lang="pug">
.row.trading-terminal
  client-only
    .col(v-if='markets_layout != null && markets_layout.length > 0')
      grid-layout(
        :layout.sync='markets_layout',
        :col-num='24',
        :row-height='40',
        :is-draggable='true',
        :is-resizable='true',
        :is-mirrored='false',
        :vertical-compact='true',
        :margin='[2, 2]',
        :use-css-transforms='true'
      )
        grid-item.overflowbox(
          v-for='item in markets_layout.filter((item) => item.status)',
          :x='item.x',
          :y='item.y',
          :w='item.w',
          :h='item.h',
          :i='item.i',
          :key='item.i',
          :min-w='item.mw',
          drag-ignore-from='.el-tabs__item, .depth-chart, a, button, .orders-list',
          drag-allow-from='.el-tabs__header, .box-card'
        )
          .right-icons
            .icon-btn
              i.el-icon-setting(
                v-if='item.i == "chart"',
                @click='show_modal = !show_modal'
              )
              i.el-icon-setting(
                v-else-if='item.i == "time-sale"',
                @click='show_timesale_modal = !show_timesale_modal'
              )
              i.el-icon-setting(v-else)
            .icon-btn
              i.el-icon-close(@click='closegriditem(item.i)')
          top-line(v-if='item.i == "chart"')
          chart(v-if='item.i == "chart"')
          el-tabs.h-100(v-loading='loading', v-if='item.i == "order-depth"')
            el-tab-pane(label='Orderbook')
              order-book
            el-tab-pane(label='Depth Chart', @click='showmessage')
              depth-chart(
                @click='showmessage',
                :is-draggable='false',
                :is-resizable='false',
                :is-mirrored='false',
                :vertical-compact='false',
                :margin='[10, 10]',
                :use-css-transforms='false',
                ondragend='myFunction(event)'
              )
          el-tabs.h-100.time-sale(v-if='item.i == "time-sale"', :min-w='3')
            el-tab-pane(label='Times and Sales')
              LatestDeals(:timeformat='timeformat')
          //- markets(v-if="item.i=='3'")
          alcor-tabs.h-100(v-if='item.i == "open-oder"', v-model='tab')
            template(slot='right')
              .d-flex
                el-button.red.hover-opacity.ml-3.mt-1(
                  type='text',
                  v-show='tab == 0',
                  size='small',
                  @click='cancelAll'
                ) Cancel All Orders
            el-tab-pane(label='Open order')
              my-orders(v-if='user', v-loading='loading')
            el-tab-pane(label='Order history')
              my-history(v-if='user', v-loading='loading')
            el-tab-pane(label='Trade History')
              my-history
            el-tab-pane(label='Funds')
              my-history
          .not-history.limit-market(v-if='item.i == "limit-market"')
            .tabs-right
              el-switch.mr-2(
                v-if='["eos"].includes(network.name) && user',
                v-model='payForUser',
                inactive-text=' Free CPU'
              )
              el-button.swap-button.rounded-0(
                v-if='relatedPool',
                type='button',
                @click='goToPool'
              ) SWAP ({{ relatedPool.rate }} {{ base_token.symbol.name }})
              FeeRate.feebutton
            el-tabs.h-100.no_drag
              el-tab-pane.h-10(label='Limit trade')
                .trade-box
                  limit-trade
              el-tab-pane(label='Market trade')
                .trade-box
                  market-trade
          //- .low-right(v-if="item.i=='6'")
          //-   .overflowbox.low-height.overflow-hidden
          //-     LatestDeals
    SettingModal(v-if='show_modal', :outofmodalClick='outofmodalClick')
    TimeSaleModal(
      v-if='show_timesale_modal',
      :outoftimesalemodalClick='outoftimesalemodalClick',
      :closemodal='closemodal',
      @changedtimeformat='showtimeformat'
    )
    #price_cancel_modal(v-if="orderdata.show_cancel_modal")
      .cancel-modal-content
        .price-info
          p Your order to:
          span.color-green &nbsp;{{orderdata.order_to}}
        .price-info
          p At a price of:
          span &nbsp;{{orderdata.price}}
        p Will be
          span.color-red &nbsp;cancelled
          |, do you wish to proceed?
        .alert-btn-group.d-flex.justify-content-between
          div(@click="cancel_confirm_order(true)") Yes
          div(@click="cancel_confirm_order(false)") No
        i.el-icon-close(@click="cancel_confirm_order(false)")
    #price_move_modal(v-if="orderdata.show_move_modal")
      .cancel-modal-content
        .price-info
          p Your order to:
          span.color-green &nbsp;{{orderdata.order_to}}
        .price-info
          p At a price of:
          span &nbsp;{{orderdata.price}}
        .price-info
          p.width-auto Will be moved to:
          span &nbsp;{{orderdata.new_price}}
        p Do you wish to proceed?
        .alert-btn-group.d-flex.justify-content-between
          div(@click="move_confirm_order(true)") Yes
          div(@click="move_confirm_order(false)") No
        i.el-icon-close(@click="move_confirm_order(false)")
</template>

<script>
import { mapGetters, mapState } from 'vuex'

import MarketTrade from '~/components/trade/MarketTrade'
import LimitTrade from '~/components/trade/LimitTrade'
import MyOrders from '~/components/trade/MyOrders'
import MyHistory from '~/components/trade/MyHistory'
import OrderBook from '~/components/trade/OrderBook'
import DepthChart from '~/components/trade/DepthChart'
import Markets from '~/components/trade/Markets'
import LatestDeals from '~/components/trade/LatestDeals'
import Chart from '~/components/trade/Chart'
import TopLine from '~/components/trade/TopLine'
import MobileTrade from '~/components/trade/MobileTrade'
import FeeRate from '~/components/trade/FeeRate'
import SettingModal from '~/components/trade/SettingModal'
import TimeSaleModal from '~/components/trade/TimeSaleModal'

export default {
  layout: 'embed',
  path: 'desktoptrade',
  name: 'Desktoptrade',
  components: {
    MarketTrade,
    MyHistory,
    MyOrders,
    LimitTrade,
    OrderBook,
    LatestDeals,
    Chart,
    Markets,
    MobileTrade,
    TopLine,
    FeeRate,
    DepthChart,
    SettingModal,
    TimeSaleModal,
  },

  data() {
    return {
      tab: 0,
      price: 0.0,
      amount: 0.0,
      no_found: false,
      loading: false,
      show_modal: false,
      show_timesale_modal: false,
      timeformat: 'DD-MM HH:mm',
    }
  },

  computed: {
    ...mapState(['network', 'userOrders']),
    ...mapState('market', [
      'token',
      'id',
      'stats',
      'base_token',
      'markets_layout',
      'orderdata'
    ]),
    ...mapGetters('market', ['relatedPool']),
    ...mapGetters(['user']),
    payForUser: {
      get() {
        return this.$store.state.chain.payForUser
      },

      set(value) {
        this.$store.commit('chain/setPayForUser', value)
      },
    },
  },
  mounted() {
    console.log(this.$store.state)
  },

  methods: {
    cancel_confirm_order(isCancel) {
      this.orderdata.show_cancel_modal = false
      //
    },
    move_confirm_order(isMove) {
      this.orderdata.show_move_modal = false
      if (isMove) this.orderdata.price = this.orderdata.new_price
      //
    },
    showtimeformat(value) {
      this.timeformat = value
    },
    closegriditem(item_name) {
      this.markets_layout.map((item) => {
        if (item.i == item_name) {
          item.status = false
        }
      })
    },
    outofmodalClick(event) {
      if (event.target.classList.contains('body-container'))
        this.show_modal = false
    },
    outoftimesalemodalClick(event) {
      if (event.target.classList.contains('body-timesale-container'))
        this.show_timesale_modal = false
    },
    closemodal(event) {
      // if (event.target.classList.contains('body-timesale-container'))
      this.show_timesale_modal = false
    },
    cancelAll() {
      this.$store.dispatch(
        'market/cancelAll',
        this.userOrders.filter(
          (a) => a.account === this.user.name && a.market_id == this.id
        )
      )
    },
    showmessage() {},
    handlePress(e) {
      // e.stopPropagation()
    },
    goToPool() {
      this.$store.dispatch('swap/setPair', this.relatedPool.id)
      this.$store.dispatch('swap/updatePair', this.relatedPool.id)
      this.$store.commit('swap/setTab', 'Swap')
      this.$router.push('/swap')
    },
  },
}
</script>

<style lang="scss" scoped>
#price_cancel_modal,
#price_move_modal {
  width: 100vw;
  height: 100vh;
  position: fixed;
  top: 0;
  left: 0;
  font-size: 14px;
  background-color: rgba(0,0,0,0.7);
  .cancel-modal-content {
    width: 300px;
    border: 2px solid #333 !important;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    padding: 15px;
    border: 1px solid ;
    background-color: #212121;
    border-radius: 5px;
    color: white;
    .color-green {
      color: #00a308;
    }
    .price-info span {
      background-color: #282828;
      padding: 3px;
      border-radius: 5px;
      margin-left: 5px;
    }
    .price-info p {
      width: 87px;
      display: inline-block;
    }
    .color-red {
      color: #f96c6c;
    }
    .alert-btn-group div {
      border-radius: 5px;
      width: 47%;
      background-color: #333;
      padding: 4px;
      text-align: center;
      cursor: pointer;
    }
    .el-icon-close {
      background-color: #333;
      position: absolute;
      padding: 3px;
      right: 1px;
      top: 1px;
      cursor: pointer;
    }
    .price-info p.width-auto {
      width: auto;
    }
  }
}

.el-item {
  position: absolute;
  border: 2px solid rgb(63, 63, 63);
  margin: 2px;
  border-radius: 2px;
  margin: 1px;
}

.theme-dark .el-tooltip__popper.is-dark {
  background: #303133 !important;
  color: #fff !important;
}

.el-tooltip__popper.is-white {
  display: flex;
  border: 2px solid blue !important;
}

.right-icons {
  display: flex;
  justify-content: center;
  align-items: center;
  position: absolute;
  right: 0;
  z-index: 100;
  margin: 1px;
}

.alcor-inner .main .box-card {
  background-color: #212121;
}

.vue-grid-item.overflowbox {
  background-color: #212121;
}

.icon-btn {
  width: 20px;
  height: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: #3f3f3f;
  border-radius: 0px 0px 0px 3px;
  cursor: pointer;
  margin: 1px;
}

.red {
  color: var(--main-red);

  &:hover,
  &:focus {
    outline: none;
    color: var(--main-red);
  }
}
.top-level {
  display: flex;
  height: 500px;

  .top-left {
    flex: 4;
  }

  .top-center {
    margin: 0 10px;
    flex: 8;
  }

  .top-right {
    flex: 3;
  }
}

.low-level {
  display: flex;
  height: 300px;

  .low-left {
    flex: 5;
  }
  .not-history {
    flex: 9;
    display: flex;
  }
  .low-center {
    min-width: 490px;
    margin: 0 10px;
    flex: 2;
  }

  .low-right {
    flex: 1;
  }
}

.low-height {
  height: 300px;
}

.bordered {
  border-right: 1px solid;
}

.trade-window {
  min-height: 650px;
}

.el-form-item {
  margin-bottom: 0px;
}

.el-card__body {
  padding: 10px;
}

.display-4 {
  font-size: 2.5rem;
  font-weight: 230;
  line-height: 1;
}

.tabs-right {
  display: flex;
  height: 20px;
  margin: 1px 1px;
  align-items: center;
  position: absolute;
  top: 1px;
  right: 43px;
  z-index: 123;
  .swap-button {
    background: #3f3f3f;
    border: 0px 0px 0px 2px !important;
    padding: 0px 10px !important;
    margin-right: 2px;
    color: #bdbdbd !important;
    height: 100% !important;
  }
}
.feebutton {
  background: #3f3f3f;
  padding: 0px 2px !important;
  margin-right: 2px !important;
  height: 100% !important;
}
@media screen and (max-width: 1350px) {
  .top-level {
    height: auto;
  }
  .low-level {
    flex-direction: column-reverse;
    height: auto;
    .low-center {
      margin: 0px 10px 0px 0px;
    }
    .low-left {
      margin-top: 10px;
    }
  }

  .orders-panel {
    display: block;
  }
}

@media screen and (min-width: 1350px) {
  .orders-panel {
    display: none;
  }
}
</style>

<style lang="scss">
.trading-terminal {
  .vue-grid-layout {
    background-color: #212121;
  }

  .vue-grid-item {
    // background-color: #282828;
  }

  .vue-grid-item.vue-grid-placeholder {
    background: #66c167 !important;
    opacity: 0.25;
  }

  .orders-list {
    user-select: none;
  }

  .time-sale {
    min-width: 165px;
    max-height: 730px;
  }

  .pool-price {
    position: absolute;

    top: 5px;
    right: 5px;
  }

  .el-tabs__nav {
    margin-left: 20px;
  }

  .low-level {
    .el-tabs__nav {
      margin-left: 20px;
    }
  }

  .trade-box {
    padding: 0 15px;

    .el-input--prefix .el-input__inner {
      padding-left: 35% !important;
    }

    .el-form-item__content {
      margin-left: 0px;
    }

    .el-tabs__header {
      margin: 0 0 10px;
    }

    .el-input__prefix {
      left: 15px;
    }

    .el-form-item {
      margin-bottom: 3px;
    }

    // Slider
    .el-slider__runway {
      margin: 5px 0;
      height: 4px;
    }

    .el-slider__button {
      width: 10px;
      height: 10px;
    }

    .el-slider__marks-text {
      margin-top: 12px;
      font-size: 10px;
    }
  }

  .low-left {
    .el-tabs__header {
      margin: 0px;
    }

    .el-table {
      font-size: 10px;
    }
  }

  // Element tables
  .el-table td,
  .el-table th {
    padding: 5px 0;
  }

  .el-table {
    .cell {
      line-height: 12px;
      padding-left: 0px;
      padding-right: 0px;
    }

    .cell:first-child {
      padding-left: 5px;
    }

    .cell:last-child {
      padding-right: 5px;
    }
  }

  .el-table {
    font-size: 10px;

    .el-table__header-wrapper {
      th {
        font-weight: 100;
      }
    }
  }
}
</style>
