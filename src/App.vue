<template>
  <div id="app">
    <h1>Order Book</h1>
    <!-- Sell section -->
    <div class="table sell" v-if="sell.length">
      <div class="tr">
        <div>Price(USD)</div>
        <div>Size</div>
        <div>Total</div>
      </div>
      <div
        class="td"
        v-for="(item, key) in sell"
        :key="`sell-${key}`"
        :class="{ new: item.isNew }"
      >
        <div class="price">{{ item.price | NumWithDigitsFormat }}</div>
        <div class="size" :key="item.size_change" :class="item.size_change">
          {{ item.size | NumFormat }}
        </div>
        <div class="total">
          <span class="bar" :style="setPercent(item.total, 'sell')" />
          {{ item.total | NumFormat }}
        </div>
      </div>
    </div>

    <!-- Current Price section -->
    <div class="current" :class="currentType">
      {{ currentPrice.price | NumFormat }}
      <svg
        v-if="currentType !== 'equal'"
        xmlns="http://www.w3.org/2000/svg"
        width="24"
        height="24"
        viewBox="0 0 24 24"
        role="presentation"
        fill="none"
        fill-rule="nonzero"
        stroke="currentColor"
        stroke-width="4"
        stroke-linecap="round"
        stroke-linejoin="round"
      >
        <line x1="12" y1="5" x2="12" y2="19"></line>
        <polyline points="19 12 12 19 5 12"></polyline>
      </svg>
    </div>

    <!-- Buy section -->
    <div class="table buy" v-if="buy.length">
      <div
        class="td"
        v-for="(item, key) in buy"
        :key="`buy-${key}`"
        :class="{ new: item.isNew }"
      >
        <div class="price">{{ item.price | NumWithDigitsFormat }}</div>
        <div class="size" :class="item.size_change">
          {{ item.size | NumFormat }}
        </div>
        <div class="total">
          <span class="bar" :style="setPercent(item.total, 'buy')" />
          {{ item.total | NumFormat }}
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data () {
    return {
      sell: [
        {
          price: 21699.0,
          size: 3691,
          total: 5657
        },
        {
          price: 21693.5,
          size: 461,
          total: 1966
        },
        {
          price: 21680.5,
          size: 53,
          total: 1505
        },
        {
          price: 21680.0,
          size: 836,
          total: 1452
        },
        {
          price: 21672.0,
          size: 40,
          total: 616
        },
        {
          price: 21669.0,
          size: 210,
          total: 576
        },
        {
          price: 21665.5,
          size: 331,
          total: 366
        },
        {
          price: 21665.0,
          size: 35,
          total: 35
        }
      ],
      buy: [
        {
          price: 21699.0,
          size: 3691,
          total: 5657
        },
        {
          price: 21693.5,
          size: 461,
          total: 1966
        },
        {
          price: 21680.5,
          size: 53,
          total: 1505
        },
        {
          price: 21680.0,
          size: 836,
          total: 1452
        },
        {
          price: 21672.0,
          size: 40,
          total: 616
        },
        {
          price: 21669.0,
          size: 210,
          total: 576
        },
        {
          price: 21665.5,
          size: 331,
          total: 366
        },
        {
          price: 21665.0,
          size: 35,
          total: 35
        }
      ],
      // 買賣榜各 50 筆資料
      initQuote: {
        sell: [],
        buy: []
      },
      currentPrice: {}, // 最新價格
      currentType: 'equal', // 最新價格趨勢：上漲、下跌、持平
      quoteWs: null, // quote Websocket
      priceWs: null // last price Websocket
    }
  },
  filters: {
    // 千分位, 小數點後兩位
    NumWithDigitsFormat (val) {
      return Number(parseFloat(val).toFixed(3)).toLocaleString('en', {
        minimumFractionDigits: 1
      })
    },
    // 千分位
    NumFormat (val) {
      return Number(parseFloat(val).toFixed(3)).toLocaleString('en', {
        minimumFractionDigits: 0
      })
    }
  },
  watch: {
    currentPrice: {
      handler (oldVal, newVal) {
        this.currentType =
          oldVal.price < newVal.price
            ? 'rise'
            : oldVal.price > newVal.price
              ? 'fall'
              : 'equal'
      },
      deep: true
    }
  },
  created () {
    // 開啓前後端的websocket通道

    // 報價表
    this.initWs('quote')

    // 最新價格
    this.initWs('price')
  },
  methods: {
    // 處理表格資料
    getQuoteData (param) {
      if (param.data.type === 'snapshot') {
        this.buy = this.setInitData('buy', param.data.bids)
        this.sell = this.setInitData('sell', param.data.asks)
      } else if (param.data.type === 'delta') {
        if (param.data.bids.length > 0) {
          this.buy = this.updateQuote('buy', param.data.bids)
          this.sell = this.updateQuote('sell', param.data.asks)
        }
      } else {
        // 取消訂閱
        this.wsSend({
          op: 'unsubscribe',
          args: ['update:BTCPFC']
        })
      }
    },
    // 初始 websocket
    initWs (val) {
      let wsURL = ''
      if (val === 'quote') {
        wsURL = 'wss://ws.btse.com/ws/oss/futures'
        this.quoteWs = new WebSocket(wsURL) // 建立連線
        this.quoteWs.onopen = this.wsOnOpenQuote
        this.quoteWs.error = this.wsOnError
        this.quoteWs.onmessage = this.wsOnMessage
        this.quoteWs.onclose = this.wsCloseQuote
      } else {
        wsURL = 'wss://ws.btse.com/ws/futures'
        this.priceWs = new WebSocket(wsURL)
        this.priceWs.onopen = this.wsOnOpenPrice
        this.priceWs.error = this.wsOnError
        this.priceWs.onmessage = this.wsOnMessage
        this.priceWs.onclose = this.wsClosePrice
      }
    },
    // 建立 websocket 連線： quote
    wsOnOpenQuote () {
      // console.log('ws 連線成功~~')
      this.wsSend('quote', {
        op: 'subscribe',
        args: ['update:BTCPFC']
      })
    },
    // 建立 websocket 連線： price
    wsOnOpenPrice () {
      // console.log('ws 連線成功~~')
      this.wsSend('price', {
        op: 'subscribe',
        args: ['tradeHistoryApi:BTCPFC']
      })
    },
    // websocket 連線失敗
    wsOnError (e) {
      console.error('ws 連線失敗', e)
    },
    // 後端通知前端，前端取得資料
    async wsOnMessage (e) {
      const _data = JSON.parse(e.data)

      if (_data.topic === 'update:BTCPFC') {
        await this.getQuoteData(_data)
      } else if (_data.topic === 'tradeHistoryApi') {
        this.currentPrice = _data.data[0]
      }
      // console.log('ws 取得資料', _data)
    },
    // 前端向 websocket 送出資料
    wsSend (type, data) {
      type === 'quote'
        ? this.quoteWs.send(JSON.stringify(data))
        : this.priceWs.send(JSON.stringify(data))
    },
    // 關閉 ws 連線：Quote
    wsCloseQuote () {
      this.wsSend({
        op: 'unsubscribe',
        args: ['update:BTCPFC']
      })
      console.log('ws 關閉 Quote 連線')
    },
    // 關閉 ws 連線：Price
    wsClosePrice () {
      this.wsSend({
        op: 'unsubscribe',
        args: ['tradeHistoryApi:BTCPFC']
      })
      console.log('ws 關閉 Price 連線')
    },
    // 設置初始買賣榜資料
    setInitData (type, param) {
      // 排序
      const temp = param.sort((a, b) => a[0] - b[0])

      // 整理資料格式為符合 UI 呈現用
      let result = []

      temp.forEach((item) => {
        result.push({
          price: Number(item[0]),
          size: Number(item[1]),
          total: 0,
          size_change: 'equal',
          isNew: false
        })

        this.initQuote[type].push({
          price: Number(item[0]),
          size: Number(item[1]),
          total: 0,
          size_change: 'equal',
          isNew: false
        })
      })

      // 取出後 8 項
      result.splice(0, 42)

      // total 數量初始值
      let count = 0
      result.forEach((item) => {
        count += Number(item.size)
        item.total = count
      })

      // 若是 sell 資料，則要反序排列
      if (type === 'sell') {
        result = result.sort((a, b) => b.price - a.price)
      }
      return result
    },
    // 設置背景百分比 CSS
    setPercent (val, type) {
      let result = 0

      result =
        type === 'sell'
          ? Math.floor((Number(val) / this.sell[0].total) * 100)
          : (result = Math.floor((Number(val) / this.buy[7].total) * 100))

      return { width: result + '%' }
    },
    // 更新買賣榜資料
    updateQuote (type, param) {
      // 將 isNew, size_change 調回預設值
      this.initQuote[type].forEach((item) => {
        item.isNew = false
        item.size_change = 'equal'
      })

      // 將新的資料 更新至 initBuy 裡
      param.forEach((item) => {
        // 原有的 initBuy 裡是否有資料
        const hasData = this.initQuote[type].findIndex(
          (old) => old.price === Number(item[0])
        )

        // 若是新的價格，則推入 initBuy
        if (hasData === -1) {
          this.initQuote[type].push({
            price: Number(item[0]),
            size: Number(item[1]),
            total: 0,
            size_change: 'equal',
            isNew: true
          })
        } else {
          // 已經存在的價格，則改寫 initBuy 的資料

          // 若 size 為 0 ，則從 this.initQuote[type] 移除價格
          if (Number(item[1]) === 0) {
            this.initQuote[type].splice(hasData, 1)
            return
          }

          // 若 新 size < 原 size
          if (this.initQuote[type][hasData].size > Number(item[1])) {
            this.initQuote[type][hasData].size_change = 'fall'
          }

          // 若 新 size > 原 size
          if (this.initQuote[type][hasData].size < Number(item[1])) {
            this.initQuote[type][hasData].size_change = 'rise'
          }

          this.initQuote[type][hasData].size = Number(item[1])
        }
      })

      this.initQuote[type] = this.initQuote[type].filter(
        (item) => item.size !== 0
      )

      // 加入新資料後再次排序
      this.initQuote[type] = this.initQuote[type].sort(
        (a, b) => a.price - b.price
      )

      // 只留最後 50 筆
      if (this.initQuote[type].length > 50) {
        this.initQuote[type].splice(0, this.initQuote[type].length - 50)
      }

      // 切出要顯示在畫面上的資料
      const result = JSON.parse(JSON.stringify(this.initQuote[type]))
      if (result.length > 8) {
        result.splice(0, result.length - 8)
      }

      // 計算要顯示在畫面上資料的 total 值
      let count = 0
      result.forEach((item) => {
        count += Number(item.size)
        item.total = count
      })

      if (type === 'sell') {
        result.sort((a, b) => b.price - a.price)
      }

      return result
    }
  },
  destroy () {
    // 關閉websocket通道
    this.wsCloseQuote()
    this.wsClosePrice()
  }
}
</script>

<style lang="scss">
$background: #131b29;
$defaultText: #f0f4f8;
$quoteTableHeadText: #8698aa;
$buyQuotePrice: #00b15d;
$sellQuotePrice: #ff5b5a;
$quoteRowHoverBackground: #1e3059;
$buyQuoteAccumulativeTotalSizeBar: rgba(16, 186, 104, 0.12);
$sellQuoteAccumulativeTotalSizeBar: rgba(255, 90, 90, 0.12);
$equalQuoteAccumulativeTotalSizeBar: rgba(134, 152, 170, 0.12);
$animationFlashGreenBackground: rgba(0, 177, 93, 0.5);
$animationFlashRedBackground: rgba(255, 91, 90, 0.5);

$p-15: 15px;

#app {
  width: 375px;
  margin: 0 auto;
  background: $background;
  text-align: center;
  color: $defaultText;
  margin-top: 60px;
  padding: 10px 0;
}

h1 {
  text-align: left;
  border-bottom: 1px solid rgba(#8698aa, 0.2);
  padding: 0 $p-15 10px;
  margin: 0;
}

.table {
  .tr {
    display: flex;
    justify-content: space-between;
    align-items: center;
    color: $quoteTableHeadText;

    > div {
      flex: 1;
      text-align: right;
    }

    div:first-child {
      text-align: left;
      padding-left: $p-15;
    }

    div:last-child {
      padding-right: $p-15;
    }
  }
  .td {
    display: flex;
    align-items: center;
    justify-content: space-between;

    > div {
      flex: 1;
      text-align: right;
    }

    div:first-child {
      text-align: left;
      padding-left: $p-15;
    }

    &:hover {
      background: $quoteRowHoverBackground;
    }

    .size {
      &.rise {
        background: $animationFlashGreenBackground;
      }

      &.fall {
        background: $animationFlashRedBackground;
      }
    }

    .total {
      position: relative;
      padding-right: $p-15;
      span {
        display: block;
        position: absolute;
        right: 0;
        height: 100%;
      }
    }
  }
}

// sell 表格
.table.sell {
  .td.new {
    background: $animationFlashRedBackground;
  }
  .td div:first-child {
    color: $sellQuotePrice;
  }
  .total {
    span {
      background: $sellQuoteAccumulativeTotalSizeBar;
    }
  }
}

// buy 表格
.table.buy {
  .td.new {
    background: $animationFlashGreenBackground;
  }

  .td div:first-child {
    color: $buyQuotePrice;
  }

  .total {
    span {
      background: $buyQuoteAccumulativeTotalSizeBar;
    }
  }
}

// 最新價格
.current {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 10px 0;
  font-weight: 900;

  svg {
    margin-left: 5px;
  }

  &.fall {
    background: $buyQuoteAccumulativeTotalSizeBar;
    color: $buyQuotePrice;

    svg {
      transform: rotate(180deg);
    }
  }

  &.rise {
    background: $sellQuoteAccumulativeTotalSizeBar;
    color: $sellQuotePrice;
  }

  &.equal {
    background: $equalQuoteAccumulativeTotalSizeBar;
    color: $defaultText;
  }
}
</style>
