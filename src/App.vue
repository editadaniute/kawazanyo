<template>
    <div class="container">
      <div class="title">
        <h1><font-awesome-icon :icon="['fab', 'bitcoin']" />仮想通貨皮算用<font-awesome-icon :icon="['fab', 'ethereum']" /></h1>
        <div class="marker"></div>
    </div>
        <h2>ー 現在の{{selectedCryptocurrency}}の価格 ー</h2>
        {{currentKasoTimestamp | currentDate}}
    <box-display>
      <div class="anotoki">
        <div class="box">
          <div class="box-title enableBuybox-current"><font-awesome-icon :icon="['fab', 'bitcoin']" />購入可能枚数（現在）</div>
          <p>{{enableKasoBuy | MathRound3}}</p>
        </div>
      </div>
      <div class="anotoki">
        <div class="box">
          <div class="box-title enableBuybox-anotoki"><font-awesome-icon :icon="['fab', 'bitcoin']" />購入可能枚数（あの時）</div>
          <p>{{bygoneEnableKasoBuy | MathRound3}}</p>
       </div>
      </div>
      <div class="anotoki">
        <div class="box">
          <div class="box-title anotokibox-notax"><font-awesome-icon :icon="['fab', 'bitcoin']" />皮算用利益（税引前）</div>
          <p>¥{{kawazanyoBefore | MathRound | moneyDelimiter}}</p>
        </div>
      </div>
      <div class="anotoki">
        <div class="box">
          <div class="box-title anotokibox-tax"><font-awesome-icon :icon="['fab', 'bitcoin']" />皮算用利益（税引後）</div>
          <p>¥{{kawazanyoAfter | MathRound | moneyDelimiter}}</p>
       </div>
      </div>
      <div class="current">
      <div class="box">
        <div class="box-title sellbox"><font-awesome-icon :icon="['fab', 'bitcoin']" />売値</div>
        <p>¥{{currentKasoSell | MathRound | moneyDelimiter}}</p>
      </div>
    </div>
    <div class="current">
      <div class="box">
        <div class="box-title buybox"><font-awesome-icon :icon="['fab', 'bitcoin']" />買値</div>
        <p>¥{{currentKasoBuy | MathRound | moneyDelimiter}}</p>
     </div>
    </div>
    <div class="current">
      <div class="box">
        <div class="box-title sagakubox"><font-awesome-icon :icon="['fab', 'bitcoin']" />差額</div>
        <p>¥{{currentKasoBuy - currentKasoSell | MathRound | moneyDelimiter}}</p>
     </div>
    </div>
    </box-display>
  
    仮想通貨：
    <select v-model="selectedCryptocurrency">
      <option v-for="kaso in optionKasos" v-bind:value="kaso.name" v-bind:key="kaso.type">
      {{ kaso.name }}
      </option>
    </select>
    <p></p>
    <div class="typeFormWrapper">
      予算：<input id="typeForm" v-model="money" type="text" class="typeForm">
      あの時の買値：<input id="typeForm" v-model="anotoki" type="text" class="typeForm">
      レバレッジ：<input id="typeForm" v-model="leverage" type="text" class="typeForm">
  </div>
  <p></p>
  <small>2021 © 仮想通貨皮算用</small>
  </div>
</template>

<script>
import axios from 'axios'
import moment from 'moment'

export default {
  name: 'kawazanyo',
  data: function() {
        return {
            money: 1000000,
            anotoki: 50000,
            leverage: 1,
            currentKasoBuy: '',
            currentKasoSell: '',
            currentKasoTimestamp: '',
            selectedCryptocurrency: 'ビットコイン', 
            optionKasos: [ 
            { type: 'btc', name: 'ビットコイン' }, 
            { type: 'eth', name: 'イーサリアム' }, 
            { type: 'xrp', name: 'リップル' },
            { type: 'ltc', name: 'ライトコイン' },
            { type: 'bcc', name: 'ビットコインキャッシュ' },
            { type: 'mona', name: 'モナコイン' }
            ], 
          }
    },
    filters: {
      moneyDelimiter(value) {
          return value.toLocaleString()
      },
      MathRound(value) {
          return Math.round(value)
      },
      MathRound1(value) {
          return Math.round(value*10) / 10
      },
      MathRound3(value) {
          return Math.round(value*1000) / 1000
      },
      currentDate(unixtime) {
          return moment(unixtime).format('YYYY-MM-DD HH:mm:ss')
      },
    },
    methods:{
      fetchRate:function(){
        let type = this.selectedCryptocurrency
        // current bitcoin rate
        let ticker = 'https://public.bitbank.cc/btc_jpy/ticker'

        if (type === 'イーサリアム') {
          ticker = ticker.replace('btc', 'eth')
        } else if (type === 'リップル') {
          ticker = ticker.replace('btc', 'xrp')
        } else if (type === 'ライトコイン') {
          ticker = ticker.replace('btc', 'ltc')
        } else if (type === 'ビットコインキャッシュ') {
          ticker = ticker.replace('btc', 'bcc')
        } else if (type === 'モナコイン') {
          ticker = ticker.replace('btc', 'mona')
        }

        axios.get(ticker).then(function(res) {
              this.currentKasoSell = res.data.data.sell
              this.currentKasoBuy = res.data.data.buy
              this.currentKasoTimestamp = res.data.data.timestamp
          }.bind(this))
            .catch(function(error) {
            console.log(error)
          })
        }
    },
    mounted:function() {
      setInterval(this.fetchRate, 1000)
    },
    computed:{
        enableKasoBuy: function() {
          return (this.money / this.currentKasoBuy) * this.leverage
        },
        bygoneEnableKasoBuy: function() {
          return (this.money / this.anotoki) * this.leverage
        },
        kawazanyoBefore: function() {
          return (((this.money / this.anotoki) * this.currentKasoSell)) * this.leverage - (((this.money / this.currentKasoBuy) * this.currentKasoBuy)) * this.leverage
        },
        kawazanyoAfter: function() { 
          let tmpKawazanyoBefore = this.kawazanyoBefore
          if (tmpKawazanyoBefore <= 1950000) {
            return tmpKawazanyoBefore * 0.95
          } else if (tmpKawazanyoBefore <= 3300000) {
            return tmpKawazanyoBefore * 0.90
          } else if (tmpKawazanyoBefore <= 6950000) {
            return tmpKawazanyoBefore * 0.80
          } else if (tmpKawazanyoBefore <= 9000000) {
            return tmpKawazanyoBefore * 0.77
          } else if (tmpKawazanyoBefore <= 18000000) {
            return tmpKawazanyoBefore * 0.67
          } else if (tmpKawazanyoBefore <= 40000000) {
            return tmpKawazanyoBefore * 0.60
          } else {
            // tmpKawazanyoBefore >= 40000001
            return tmpKawazanyoBefore
          }
        }
    }
}
</script>

<style scorped>
.container{
  width: 640px;
  margin: 0 auto;
  text-align: center;
  font-family: 'Noto Sans JP', sans-serif;
}

.title{
  position: relative;
  font-size: 32px;
}

.marker{
  width: 100%;
  height: 35%;
  background-color: #04002c75;
  position: absolute;
  bottom: 5%;
  z-index: -1;
}

box-display {
  display: flex;
  flex-wrap: wrap-reverse;
}

box-display div.current {
  width: calc(33.3% - 10px);
  margin: 5px;
}
box-display div.anotoki {
  width: calc(50% - 10px);
  margin: 5px;
}

.box {
  margin: 1em 0;
  background: #f1f1f1;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.22);
}

.box .box-title {
  font-size: 1em;
  padding: 4px;
  text-align: center;
  color: #FFF;
  font-weight: bold;
  letter-spacing: 0.05em;
}

.box p {
  font-size: 1.5em;
  padding: 15px 20px;
  margin: 0;
}

.buybox {
  background: #0e80b9;
}

.sellbox{
  background: #c42474;
}

.sagakubox{
  background: #289b6f;
}

.anotokibox-notax{
  background: #d30fd3;
}

.anotokibox-tax{
  background: #e45d27;
}

.enableBuybox-current{
  background: red;
}

.enableBuybox-anotoki{
  background: blue;
}

.h2 {
  font-family: 'Noto Sans JP', sans-serif;
}

input {
  padding: 5px;
  width: 5em;
  font-size: 15px;
  text-align: center;
}

select {
  padding: 5px;
  font-size: 15px;
  text-align: center;
}
</style>