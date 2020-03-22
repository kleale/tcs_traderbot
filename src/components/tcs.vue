<template>
  <div class="tcs">
    <ul>
      <li v-for="record in appdata.portfolioCurrencies.currencies" :key="record.currencies">
        <!-- <span class="date">{{ record.commit.author.date | formatDate }}</span>  -->
        {{ record.currency}} - {{ record.balance}}
      </li>
    </ul> 

    <table class="table table-dark table-hover table-sm">
      <!-- <thead>
        <tr>
            <th scope="col">First</th>
            <th scope="col">Last</th>
            <th scope="col">Handle</th>
        </tr>
      </thead>-->
      <tbody>
        <tr v-for="record in appdata.portfolio.positions" :key="record.positions">
          <th scope="row">{{record.ticker}}</th>
          <td>{{ record.name}}</td>
          <td>{{ record.lots}}</td>
          <td></td>
        </tr>
      </tbody>
    </table>

    <!-- <ul>
      <li v-for="record in orderbook.portfolio.positions" :key="record.positions">
        {{record.ticker}} {{ record.name}} - {{ record.lots}}
      </li>
    </ul>-->

    <trading-vue :data="this.$data"></trading-vue>
  </div>
</template>

<script>
import config from "../config/index.js";
import TradingVue from "trading-vue-js"; //https://www.npmjs.com/package/trading-vue-js/v/0.1.5
import * as moment from "moment";
import OpenAPI from "@tinkoff/invest-openapi-js-sdk";

export default {
  name: "TCS",
  props: {
    msg: String
  },
  components: { TradingVue },
  data() {
    return {
      appdata: {
        portfolioCurrencies: {
          currencies: null
        },
        portfolio: {
          positions: []
        },
        orderbook: {}
      },
      ohlcv: [
        //[timestamp, open, high, low, close, volume]
        // [ 1551128400000, 33,  37.1, 14,  14,  196 ],
        // [ 1551132000000, 13.7, 30, 6.6,  30,  206 ],
        // [ 1551135600000, 29.9, 33, 21.3, 21.8, 74 ],
        // [ 1551139200000, 21.7, 25.9, 18, 24,  140 ],
        // [ 1551142800000, 24.1, 24.1, 24, 24.1, 29 ],
      ]
    };
  },

  async mounted() {
    //const apiURL = 'https://api-invest.tinkoff.ru/openapi';
    const sandboxApiURL = "https://api-invest.tinkoff.ru/openapi/sandbox/";
    const socketURL = "wss://api-invest.tinkoff.ru/openapi/md/v1/md-openapi/ws";
    //const secretToken = process.env.TOKEN; // токен для боевого api
    //const sandboxToken = process.env.SANDBOX_TOKEN; // токен для сандбокса
    const sandboxToken = config.sandboxToken;
    const api = new OpenAPI({
      apiURL: sandboxApiURL,
      secretToken: sandboxToken,
      socketURL
    });

    //!(async function run() {
    console.info("tinkoff-api start");
    //await api.sandboxClear();
    const { figi } = await api.searchOne({ ticker: "AAPL" });

    //   console.log(
    //     await api.setCurrenciesBalance({ currency: "USD", balance: 101 })
    //   ); // 1000$ на счет
    console.log(await api.portfolioCurrencies());

    //   console.log(await api.instrumentPortfolio({ figi })); // В портфеле ничего нет
    //   console.log(
    //     await api.limitOrder({ operation: "Buy", figi, lots: 1, price: 100 })
    //   ); // Покупаем AAPL
    //  console.log(await api.instrumentPortfolio({ figi })); // Сделка прошла моментально

    //   console.log(await api.orderbookGet({ figi })); // получаем стакан по AAPL

    // console.log(
    //   await api.candlesGet({
    //     from: "2019-08-19T18:38:33.131642+03:00",
    //     to: "2019-08-19T18:48:33.131642+03:00",
    //     figi,
    //     interval: "1min"
    //   }) // Получаем свечи за конкретный промежуток времени.
    // );

    api.orderbook({ figi, depth: 10 }, x => {
      console.log(x.bids);
    });
    api.candle({ figi }, x => {
      console.log(x.h);
    });

    console.log("portfolio", await api.portfolio());
    //})();

    // portfolio
    let portfolioCurrencies = await api.portfolioCurrencies();
    let portfolio = await api.portfolio();
    // let orderbook = api.orderbook({ figi, depth: 10 }, x => {
    //     //console.log(x.bids);
    //     return x.bids
    // });

    this.appdata = {
      portfolioCurrencies: portfolioCurrencies,
      portfolio: portfolio
      //orderbook: orderbook
    };
    console.log("appdata", this.appdata);

    ///////////////////////////////
    // chart
    // candles
    //let now = new Date();
    let now = moment().valueOf();
    let minutes_ofset = 1;
    let hour_ofset = 17;
    let days_ofset = 3;

    let _to1 = now - days_ofset * 60000 * 60 * 24 - hour_ofset * 60000 * 60;
    let _to = moment(_to1)
      .utc()
      .format("YYYY-MM-DD[T]HH:mm:ss.SSSSSSZ");

    let _from1 =
      moment().valueOf() -
      minutes_ofset * 60000 -
      days_ofset * 60000 * 60 * 24 -
      hour_ofset * 60000 * 60;
    let _from = moment(_from1)
      .utc()
      .format("YYYY-MM-DD[T]HH:mm:ss.SSSSSSZ");

    console.log("_to", _to);
    console.log("_from", _from);
    let candles = await api.candlesGet({
      from: "2019-08-19T18:38:33.131642+03:00",
      to: "2019-08-19T18:48:33.131642+03:00",
      //from: _from,
      //to: _to,
      figi,
      interval: "1min"
    });

    console.log("candles", candles);

    let _ohlcv = [];
    candles.candles.forEach(function(item) {
      let candle = [];
      let time = new Date(item.time).getTime();
      candle.push(time);
      candle.push(item.o);
      candle.push(item.h);
      candle.push(item.l);
      candle.push(item.c);
      candle.push(item.v);
      _ohlcv.push(candle);
    });

    console.log("ohlcv_", _ohlcv);
    this.ohlcv = _ohlcv;
    // this.ohlcv = [[timestamp, open, high, low, close, volume]]
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.tcs {
}
</style>