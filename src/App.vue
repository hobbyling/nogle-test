<template>
  <div id="app">
    <h1>Order Book</h1>

    <div class="table sell">
      <div class="tr">
        <div>Price(USD)</div>
        <div>Size</div>
        <div>Total</div>
      </div>
      <div class="td" v-for="(item, key) in sell" :key="`sell-${key}`">
        <div class="price">{{ item.price | NumFormat }}</div>
        <div class="size">{{ item.size | NumFormat }}</div>
        <div class="total">{{ item.total | NumFormat }}</div>
      </div>
    </div>
    <div class="diff rise">
      {{ diff | NumFormat }}
      <svg
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
    <div class="table buy">
      <div class="td" v-for="(item, key) in sell" :key="`sell-${key}`">
        <div class="price">{{ item.price | NumFormat }}</div>
        <div class="size">{{ item.size | NumFormat }}</div>
        <div class="total">{{ item.total | NumFormat }}</div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data () {
    return {
      diff: 21657.5,
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
      wsNotify: {},
      ws: null
    }
  },
  filters: {
    NumFormat (val) {
      return Number(parseFloat(val).toFixed(3)).toLocaleString('en', {
        minimumFractionDigits: 1
      })
    }
  },
  watch: {
    wsNotify: {
      handler (val) {
        // const { data } = val
        // 根據data決定要do something
      },
      deep: true
    }
  },
  methods: {
    setWsNotify (param) {
      this.wsNotify = param
    },
    // 初始websocket
    initWebsocket () {
      const wsURL = 'wss://ws.btse.com/ws/oss/futures'
      this.ws = new WebSocket(wsURL) // 建立連線
      this.ws.onopen = this.websocketonopen
      this.ws.error = this.websocketonerror
      this.ws.onmessage = this.websocketonmessage
      this.ws.onclose = this.websocketclose
    },
    websocketonopen () {
      console.log('ws 連線成功~~')
    },
    websocketonerror (e) {
      console.error('ws 連線失敗', e)
    },
    websocketonmessage (e) {
      // 後端通知前端，前端取得資料
      const _data = e.data
      // 當有後端資料送到前端，利用vuex存到共用的state
      this.setWsNotify({
        data: JSON.parse(_data)
      })
      console.log('ws 取得資料', _data)
    },
    websocketsend (data) {
      // 前端丟資料
      console.log('send data', data)
      this.ws.send({
        op: 'subscribe',
        args: ['update:BTCPFC']
      })
    },
    websocketclose () {
      console.log('ws 關閉連線')
    }
  },
  created () {
    this.initWebsocket() // 開啓前後端的websocket通道
  },
  destroy () {
    this.websocketclose() // 關閉websocket通道
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

$p-x: 15px;

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
  padding: 0 $p-x 10px;
  margin: 0;
}

.table {
  padding: 0 $p-x;
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
  }
}

.table.sell {
  .td div:first-child {
    color: $sellQuotePrice;
  }
}

.table.buy {
  .td div:first-child {
    color: $buyQuotePrice;
  }
}

.diff {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 10px 0;
  font-weight: 900;

  svg {
    margin-left: 5px;
  }

  &.rise {
    background: $buyQuoteAccumulativeTotalSizeBar;
    color: $buyQuotePrice;

    svg {
      transform: rotate(180deg);
    }
  }

  &.fall {
    background: $sellQuoteAccumulativeTotalSizeBar;
    color: $sellQuotePrice;
  }
}
</style>
