<template>
  <div class="tcs">
    <div class="container-responsive">
      <div class="row">
        <div class="col-lg-4" id="column1">
          <ul>
            <li v-for="record in appdata.portfolioCurrencies.currencies" :key="record.currencies">
              <!-- <span class="date">{{ record.commit.author.date | formatDate }}</span>  -->
              {{ record.currency}} - {{ record.balance}}
            </li>
          </ul>
          <table class="table table-dark table-hover table-sm">
            <thead>
              <tr>
                <th scope="col"></th>
                <th scope="col">lots</th>
                <th scope="col">bal</th>
                <th scope="col">block</th>
                <th>cur</th>
                <th class="text-right" scope="col" title="expectedYield">Profit</th>
                <th class="text-right" scope="col" title="averagePositionPrice">Now</th>
                <th class="text-right">Buy</th>
                <th>
                  <!--<button @click="save" class="btn btn-primary btn-sm">Save</button>-->
                  <button @click="load" class="btn btn-primary btn-sm">Load</button>
                </th>
                <th></th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="record in appdata.portfolio.positions" :key="record.positions">
                <th scope="row">
                  {{record.ticker}}
                  <span class="name">{{ record.name}}</span>
                </th>
                <td>{{ record.lots}}</td>
                <td>{{ record.balance}}</td>
                <td>{{ record.blocked}}</td>
                <td>{{ record.expectedYield.currency}}</td>
                <td class="text-right">{{ record.expectedYield.value}}</td>
                <td class="text-right">{{ record.averagePositionPrice.value}}</td>
                <td class="text-right">{{record.currentPrice}}</td>
                <td width="70">
                  <input
                    :id="record.ticker"
                    v-model="record.interestPrice"
                    v-on:change="save"
                    class="form-control form-control-sm"
                    aria-label="Text input with checkbox"
                  />
                </td>
                <td>
                  <input type="radio" name="exampleRadios" value="chart_tiker" unchecked />
                </td>
              </tr>
            </tbody>
          </table>
        </div>
        <div class="col-lg-6">
          <!-- <trading-vue :data="this.$data"></trading-vue> -->
          <trading-vue
            :data="chart"
            :width="this.width"
            :height="this.height"
            title-txt="The King"
            :toolbar="true"
            :color-back="colors.colorBack"
            :color-grid="colors.colorGrid"
            :color-text="colors.colorText"
          ></trading-vue>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import config from "../config/index.js";
import TradingVue from "trading-vue-js"; //https://www.npmjs.com/package/trading-vue-js/v/0.1.5
//import Data from "../config/data_tools.json";
import "trading-vue-js/src/stuff/utils.js";
import DataCube from "trading-vue-js/src/helpers/datacube.js";

import * as moment from "moment";
import OpenAPI from "@tinkoff/invest-openapi-js-sdk";

export default {
  name: "TCS",
  props: {
    msg: String
  },
  components: { TradingVue },
  methods: {
    onResize() {
      // for TradingVue chart
      this.width = window.innerWidth - 700;
      this.height = window.innerHeight - 43;
    },
    save() {
      let save = [];
      this.appdata.portfolio.positions.forEach(function(item) {
        if (item.interestPrice) {
          save.push({
            ticker: item.ticker,
            interestPrice: item.interestPrice
          });
        }
      });
      let parsed = JSON.stringify(save);
      localStorage.setItem("positions", parsed);
      console.log("сохранили positions", save);
    },
    load() {
      let positionsNow = this.appdata.portfolio.positions;
      let positionsLoaded = JSON.parse(localStorage.getItem("positions"));

      positionsLoaded.forEach(function(item) {
        let position = positionsNow.find(x => x.ticker === item.ticker);
        console.log("find position", position);
        position.interestPrice = item.interestPrice;
      });

      //this.appdata.portfolio.positions = positionsNow;
      console.log("загрузили appdata", this.appdata.portfolio.positions);
    }
  },
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
      // cahrt
      width: window.innerWidth,
      height: window.innerHeight,
      colors: {
        colorBack: "#121826",
        colorGrid: "#2e3a57",
        colorText: "#ccc"
      },
      chart: {}
    };
  },

  created() {
    window.addEventListener("resize", this.onResize); // for TradingVue chart
    this.onResize();
    window.DataCube = this.chart;
  },
  beforeDestroy() {
    window.removeEventListener("resize", this.onResize);
  },

  async mounted() {
    //const socketURL = config.socketURL;
    const api = new OpenAPI({
      // БОЕВОЙ API
      apiURL: config.apiURL,
      secretToken: config.secretToken

      // apiURL: config.sandboxApiURL,
      // secretToken: config.sandboxToken;
      //,socketURL
    });

    //!(async function run() {

    //await api.sandboxClear();

    const { figi } = await api.searchOne({ ticker: "AAPL" });

    //   console.log(
    //     await api.setCurrenciesBalance({ currency: "USD", balance: 101 })
    //   ); // 1000$ на счет
    console.log(await api.portfolioCurrencies());

    //console.log("instrumentPortfolio", await api.instrumentPortfolio({ figi })); // В портфеле ничего нет
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

    // api.orderbook({ figi, depth: 10 }, x => {
    //   console.log(x.bids);
    // });
    // api.candle({ figi }, x => {
    //   console.log(x.h);
    // });

    //})();

    // portfolio
    let portfolioCurrencies = await api.portfolioCurrencies();
    let portfolio = await api.portfolio();

    portfolio.positions.forEach(function(item) {
      item.currentPrice = (
        (item.averagePositionPrice.value * item.lots +
          item.expectedYield.value) /
        item.lots
      ).toFixed(2);
      item.interestPrice = "";
      try {
        let positionsLoaded = JSON.parse(localStorage.getItem("positions"));
        let position = positionsLoaded.find(x => x.ticker === item.ticker);
        console.log("find position", position);
        item.interestPrice = position.interestPrice;
      } catch (e) {
        //localStorage.removeItem('cats');
      }
    });

    console.log("portfolio", portfolio);

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
    let minutes_ofset = 30;
    let hour_ofset = 3;
    let days_ofset = 5;

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
      //from: "2019-08-19T18:38:33.131642+03:00",
      //to: "2019-08-19T18:48:33.131642+03:00",
      from: _from,
      to: _to,
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

    let tools = [
      {
        type: "LineTool",
        settings: {
          color: "#35c460"
        }
      },
      {
        type: "LineTool:Extended",
        settings: {
          color: "#3186c4"
        }
      }
    ];

    let Data = {
      ohlcv: _ohlcv,
      tools: tools,
      onchart: [
        {
          name: "EMA, 25",
          type: "EMA",
          data: [],
          settings: {}
        }
      ],
      offchart: [
        {
          name: "RSI, 20",
          type: "RSI",
          data: [],
          settings: {
            upper: 40,
            lower: 30,
            backColor: "#9b9ba316",
            bandColor: "#666"
          }
        }
      ]
    };

    console.log("this.chart", Data);
    this.chart = new DataCube(Data);

    // setTimeout(() => {
    //             // Async data setup
    //             this.$set(this, 'chart', Data)
    //         }, 0)

    // this.ohlcv = [[timestamp, open, high, low, close, volume]]

    // сохраняем интересные мне значения
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.tcs {
  overflow: hidden;
}
.form-control-sm {
  padding: 0.1rem 0.6rem;
  height: 1.7rem;
  margin: -0.1rem 0;
  background: rgba(255, 255, 255, 0.07);
  border: transparent;
  color: #c1c1c1;
}
.name {
  font-size: 0.75rem;
  opacity: 0.5;
}
</style>