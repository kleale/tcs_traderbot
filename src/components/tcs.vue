<template>
  <div class="tcs">
    <ul>
      <li v-for="record in appdata.portfolioCurrencies.currencies" :key="record.currencies">
        <!-- <span class="date">{{ record.commit.author.date | formatDate }}</span>  -->
        {{ record.currency}} - {{ record.balance}}
      </li>
    </ul>

    <ul>
      <li v-for="record in appdata.portfolio.positions" :key="record.positions">
        {{record.ticker}} {{ record.name}} - {{ record.lots}}
      </li>
    </ul>

    <!-- <ul>
      <li v-for="record in orderbook.portfolio.positions" :key="record.positions">
        {{record.ticker}} {{ record.name}} - {{ record.lots}}
      </li>
    </ul> -->
  </div>
</template>

<script>
import OpenAPI from "@tinkoff/invest-openapi-js-sdk";

export default {
  name: "TCS",
  props: {
    msg: String
  },
  data() {
    return {
      appdata: {
        portfolioCurrencies: {
          currencies: null
        },
        portfolio: {
          positions:[]
        },
        orderbook: {}
      }
    };
  },

  async mounted() {
    //const apiURL = 'https://api-invest.tinkoff.ru/openapi';
    const sandboxApiURL = "https://api-invest.tinkoff.ru/openapi/sandbox/";
    const socketURL = "wss://api-invest.tinkoff.ru/openapi/md/v1/md-openapi/ws";
    //const secretToken = process.env.TOKEN; // токен для боевого api
    //const sandboxToken = process.env.SANDBOX_TOKEN; // токен для сандбокса
    const sandboxToken =
      "t.ugJHWRvTGjmhrayoDkP4lNtaM5B6gKi2CTweEplDnacYUc4rk4kx8iRFzVkCCb3MuUwPmVfIAMVfGTzbFCDveg";
    const api = new OpenAPI({
      apiURL: sandboxApiURL,
      secretToken: sandboxToken,
      socketURL
    });

    //!(async function run() {
      console.info("tinkoff-api start");
      //await api.sandboxClear();
      const { figi } = await api.searchOne({ ticker: "AAPL" });

      console.log(
        await api.setCurrenciesBalance({ currency: "USD", balance: 101 })
      ); // 1000$ на счет
      console.log(await api.portfolioCurrencies());

      console.log(await api.instrumentPortfolio({ figi })); // В портфеле ничего нет
      console.log(
        await api.limitOrder({ operation: "Buy", figi, lots: 1, price: 100 })
      ); // Покупаем AAPL
      console.log(await api.instrumentPortfolio({ figi })); // Сделка прошла моментально

      //   console.log(await api.orderbookGet({ figi })); // получаем стакан по AAPL

      //   console.log(
      //     await api.candlesGet({
      //       from: "2019-08-19T18:38:33.131642+03:00",
      //       to: "2019-08-19T18:48:33.131642+03:00",
      //       figi,
      //       interval: "1min"
      //     }) // Получаем свечи за конкретный промежуток времени.
      //   );

      api.orderbook({ figi, depth: 10 }, x => {
        console.log(x.bids);
      });
      api.candle({ figi }, x => {
        console.log(x.h);
      });

      console.log("portfolio", await api.portfolio());
    //})();


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
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.tcs {
  color: #333;
}
</style>