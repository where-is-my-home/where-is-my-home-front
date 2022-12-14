<template>
  <div>
    <Bar
      :chart-options="chartOptions"
      :chart-data="chartData"
      :chart-id="chartId"
      :dataset-id-key="datasetIdKey"
      :plugins="plugins"
      :css-classes="cssClasses"
      :styles="styles"
      :width="width"
      :height="height"
    />

    <div v-show="!flag" id="exist"></div>
    <div v-show="flag" id="noexist">1년간 거래내역이 존재하지 않습니다.</div>
  </div>
</template>

<script>
// vue-chartjs 참고
// https://vue-chartjs.org/guide/#creating-your-first-chart
import { mapState } from "vuex";

import { Bar } from "vue-chartjs";
import {
  Chart as ChartJS,
  Title,
  Tooltip,
  Legend,
  BarElement,
  CategoryScale,
  LinearScale,
} from "chart.js";

ChartJS.register(
  Title,
  Tooltip,
  Legend,
  BarElement,
  CategoryScale,
  LinearScale
);

export default {
  name: "BarChart",
  components: { Bar },
  props: {
    chartId: {
      type: String,
      default: "bar-chart",
    },
    datasetIdKey: {
      type: String,
      default: "label",
    },
    width: {
      type: Number,
      default: 400,
    },
    height: {
      type: Number,
      default: 400,
    },
    cssClasses: {
      default: "",
      type: String,
    },
    styles: {
      type: Object,
      default: () => {},
    },
    plugins: {
      type: Object,
      default: () => {},
    },
  },
  data() {
    return {
      flag: false,
    };
  },
  computed: {
    ...mapState("houseStore", ["houseDeal"]),
    chartData() {
      let ret = {
        labels: [
          "1월",
          "2월",
          "3월",
          "4월",
          "5월",
          "6월",
          "7월",
          "8월",
          "9월",
          "10월",
          "11월",
          "12월",
        ],
        datasets: [{}, {}],
      };
      // chart의 데이터셋을 houseDeal의 dealAmount 최대 최솟값으로 초기화해주기
      var arrMax = Array.from({ length: 12 }, () => 0);
      var arrMin = Array.from({ length: 12 }, () => Number.MAX_VALUE);

      this.changeFlag();
      for (let i = 0; i < this.houseDeal.length; i++) {
        const data = this.houseDeal[i];
        const dealMonth = data.dealMonth;
        const dealAmount = parseInt(data.dealAmount.replace(",", "")) / 1000;
        arrMax[dealMonth - 1] = Math.max(arrMax[dealMonth - 1], dealAmount);
        arrMin[dealMonth - 1] = Math.min(arrMin[dealMonth - 1], dealAmount);
      }

      for (let i = 0; i < 12; i++) {
        if (arrMin[i] == Number.MAX_VALUE) arrMin[i] = 0;
      }

      // datasets 속성 종류
      // https://www.chartjs.org/docs/latest/charts/bar.html

      // 색상 코드 찾는 사이트
      // https://www.color-hex.com/
      ret.datasets[0] = {
        label: "최고 매매가",
        data: arrMax,
        backgroundColor: "#ffb5b5",
      };
      ret.datasets[1] = {
        label: "최저 매매가",
        data: arrMin,
        backgroundColor: "#9fc5e8",
      };

      return ret;
    },
    chartOptions() {
      return {
        responsive: true,
        aspectRatio: 1.2,
        scales: {
          x: {
            ticks: {
              font: {
                size: 11,
              },
            },
          },
        },
        plugins: {
          title: {
            display: true,
            text: "2021년 거래내역 (단위 : 천만)",
            font: {
              size: 15,
            },
          },
          legend: {
            labels: {
              font: {
                size: 12,
              },
            },
          },
        },
      };
    },
  },
  created() {
    console.log("차트 그려지기 시작");
    console.log(this.houseDeal);
  },
  methods: {
    changeFlag() {
      if (this.houseDeal.length == 0) {
        this.flag = true;
      } else {
        this.flag = false;
      }
    },
  },
};
</script>

<style scoped>
#bar {
  height: 25vh;
  position: absolute;
}
/* #exist {
  height: 25vh;
  width: 100%;
  position: absolute;
  z-index: 5;
} */
#noexist {
  font-size: 12px;
  height: 178px;
  width: 311px;
  margin-left: 30px;
  margin-top: 30px;
  background-color: rgba(241, 241, 241, 0.65);
  align-items: center;
  display: flex;
  justify-content: center;
  position: fixed;
  bottom: 71px;
}
</style>
