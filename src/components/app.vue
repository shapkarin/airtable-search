<template>
  <div class="hello">
    <div class="wrapper">
      <input v-model="code" class="code" placeholer="Введите номер отслеживания"/>
      <button @click="search" class="button">Найти</button>
      <div class="result" v-if="output.length">
        <div class="order_number">Номер заказа {{ order_number }}</div>
        <div v-for="item in output" :key="item.date">
            <div class="date">{{ item.date }}</div>
            <div class="stage">{{ item.stage }}</div>
        </div>
      </div>
    </div>
    <div v-if="none">
      Не найдено
    </div>
  </div>
</template>

<script>
import Airtable from "airtable";
import _ from "lodash";

export default {
  props: {
    msg: String,
  },
  data: () => ({
    code: '',
    output: [],
    order_number: '',
    none: false
  }),
  methods: {
    search() {
      const table = new Airtable({ apiKey: "keyzJ1Et2GNfslLSL" });
      var base = table.base("app9DZOX12KLlGanS");

      base("Table 1")
        .select({
          view: "Grid view",
        })
        .firstPage((err, records) => {
          if (err) {
            console.error(err);
            return;
          }
          records.forEach((record) => {
            if (record.get("Tracking_number") === this.code) {
              // this.none = false;
              this.order_number = record.get('order_number');
              const keys = _.keys(record.fields).sort();

              const dates = keys.filter(key => {
                return key.match(/Date \d/g);
              });

              const stages = keys.filter(key => {
                return key.match(/Stage \d/g);
              });

              const result = dates.map((date, i) => {
                return {
                  date: record.fields[date],
                  stage: record.fields[stages[i]]
                }
              });

              this.output = result;
            }
          });
        });
    },
  },
};
</script>

<style scoped>
.wrapper {
  max-width: 414px;
}
.code {
  width: 75%;
  height: 48px;
  background: #FFFFFF;
  border: 1px solid #B7B7B7;
  box-sizing: border-box;
  border-top-left-radius: 6px;
  border-bottom-left-radius: 6px;
  padding-left: 10px;
  outline: none;
  margin-bottom: 52px;
}
.button {
  width: 25%;
  height: 48px;
  background: #64DB8D;
  border-top-right-radius: 6px;
  border-bottom-right-radius: 6px;
  border: none;
  outline: none;
  cursor: pointer;
}
.result {
  background: #FFFFFF;
  border: 1px solid #F1F1F1;
  box-sizing: border-box;
  border-radius: 8px;
  width: 100%;
  margin: 0 auto;
  padding: 20px;
  text-align: left;
}
.date {
  font-family: Roboto;
  font-size: 16px;
  line-height: 19px;
  color: #919191;
  margin-bottom: 4px;
}
.stage {
  font-family: Roboto;
  font-size: 16px;
  line-height: 19px;
  color: #000000;
  margin-bottom: 24px;
}
.order_number {
  font-family: Roboto;
  font-weight: 500;
  font-size: 20px;
  line-height: 23px;
  color: #000000;
  margin-bottom: 32px;
}
</style>
