<template lang="pug">
.bg
  h1 Sentiment Analysis
  v-row
    v-col(cols="4")
      v-row(style="padding:10px; background-color:#222222;")
        v-col(cols="6")
          v-img(:src="thumbnail" contain height="300")
        v-col(cols="6")
          v-img(:src="linedimage" contain height="300")
      v-row(style="padding:10px; background-color:#D81B60;")
        v-col(cols="12")
          span.font-weight-bold.text-h2 {{`時給`}}
        v-col(cols="12")
          span.font-weight-bold.white.black--text.text-h1 {{money}}
          span.font-weight-bold.text-h1 {{`円`}}


    v-col(cols="8")
      v-row
        v-card.space-all(
          v-for="(chartdata, i) in emotionChartData" :key="i"
          :height="chartHeight", :width="chartWidth"
        )
          template(v-if="chartdata.component==='face-chart'")
            h4 {{chartdata.key}}
            face-chart(
              :chartdata="makeEmotionChartData(chartdata.key, chartdata.options)"
              :options="options",
              :height="chartHeight",
              :width="chartWidth"
            )
          template(v-else-if="chartdata.component==='bar-chart'")
            h4 {{chartdata.key}}
            bar-chart(
              :chartdata="makeEmotionChartData(chartdata.key, chartdata.options)"
              :options="options",
              :height="chartHeight",
              :width="chartWidth"
            )
        v-card.space-all()
          doughnut-chart(
            :chartdata="emotionsDoughnutChart1"
            :options="options",
            :height="chartHeight",
            :width="chartWidth"
          )
        v-card.space-all()
          doughnut-chart(
            :chartdata="emotionsDoughnutChart2"
            :options="options",
            :height="chartHeight",
            :width="chartWidth"
          )
        v-col
          v-simple-table(height="180px" fixed-header)
            template(v-slot:default)
              thead
                tr
                  th(v-for="key in Object.keys(tableData[0])" :key="key") {{key}}
              tbody
                tr(v-for="(item, i) in tableData" :key="i")
                  td(v-for="(key) in Object.keys(tableData[0])" :key="key") {{ item[key] }}

</template>
<script>
import FaceChart from "~/components/FaceChart";
import BarChart from "~/components/BarChart";
import DoughnutChart from "~/components/DoughnutChart";
import firebase from "~/plugins/firebase.js";
export default {
  data: function() {
    return {
      data: [],
      chartWidth: 180,
      chartHeight: 180,
      emotionChartData: [
        {
          component: "face-chart",
          key: "HAPPY",
          options: { borderColor: "#79f879" },
        },
        {
          component: "face-chart",
          key: "SAD",
          options: { borderColor: "#79f879" },
        },
        {
          component: "face-chart",
          key: "ANGRY",
          options: { borderColor: "#79f879" },
        },
        {
          component: "face-chart",
          key: "SURPRISED",
          options: { borderColor: "#79f879" },
        },
        {
          component: "bar-chart",
          key: "CONFUSED",
          options: {
            backgroundColor: "#FF0000",
          },
        },
        {
          component: "bar-chart",
          key: "CALM",
          options: {
            backgroundColor: "#FF0000",
          },
        },
        {
          component: "bar-chart",
          key: "DISGUSTED",
          options: {
            backgroundColor: "#FF0000",
          },
        },
        {
          component: "bar-chart",
          key: "FEAR",
          options: {
            backgroundColor: "#FF0000",
          },
        },
      ],
      wage: 0,
      thumbnail: "",
      linedimage: "",
      options: {
        animation: false,
        legend: {
          display: false, //ラベル非表示
        },
      },
    };
  },
  componets: { FaceChart, BarChart, DoughnutChart },
  async mounted() {
    await firebase
      .firestore()
      .collection("faces")
      .orderBy("timestamp", "desc")
      .limit(20)
      .onSnapshot((doc) => {
        const chartdata = {
          labels: [],
          datasets: [
            {
              label: "喜び",
              data: [],
            },
          ],
        };
        this.data = [];
        doc.forEach((item) => {
          const d = item.data();
          this.data.push(d);
        });
        const lastData = this.data[0];
        this.thumbnail =
          "https://hacku2020.s3-ap-northeast-1.amazonaws.com/" +
          lastData.thumbnailImage;
        this.linedimage =
          "https://hacku2020.s3-ap-northeast-1.amazonaws.com/" +
          lastData.linedImage;
      });
  },
  methods: {
    makeEmotionChartData(key, options) {
      const chartdata = {};
      chartdata.datasets = [];
      chartdata.datasets[0] = {};
      chartdata.datasets[0].data = [];
      chartdata.labels = [];
      this.data.forEach((d) => {
        const happy = d.FaceDetails[0].Emotions.find((e) => e.Type === key);
        chartdata.datasets[0].data.push(happy.Confidence);
        chartdata.labels.push("");
      });
      chartdata.datasets[0] = Object.assign(chartdata.datasets[0], options);
      return chartdata;
    },
    makeEmotionsDoughnutData(keys, backgroundColors) {
      const chartdata = {};
      chartdata.datasets = [];
      chartdata.datasets[0] = {};
      chartdata.datasets[0].data = [];
      chartdata.labels = [];
      chartdata.datasets[0].backgroundColor = [];
      if (this.data.length <= 0) {
        return chartdata;
      }
      for (let i = 0; i < keys.length; i++) {
        const d = this.data[0].FaceDetails[0].Emotions.find(
          (e) => e.Type === keys[i]
        );
        if (d) {
          chartdata.datasets[0].data.push(d.Confidence);
          chartdata.labels.push("");
          chartdata.datasets[0].backgroundColor.push(backgroundColors[i]);
        }
      }
      return chartdata;
    },
  },
  computed: {
    money() {
      let sum = 0;
      this.data.forEach((d) => {
        console.log(d);
        const emo = d.FaceDetails[0].Emotions.find((e) => e.Type === "HAPPY");
        sum += Number(emo.Confidence);
      });
      return Math.floor((sum / this.data.length) * 20);
    },
    myStyles() {
      return {
        height: `300px`,
        position: "relative",
      };
    },
    emotionsDoughnutChart1() {
      return this.makeEmotionsDoughnutData(
        ["HAPPY", "SAD", "ANGRY", "SURPRISED"],
        ["#800000", "#800080", "#ff9e3d", "#FFFF00"]
      );
    },
    emotionsDoughnutChart2() {
      return this.makeEmotionsDoughnutData(
        ["CONFUSED", "CALM", "DISGUSTED", "FEAR"],
        ["#0000cd", "#ff00ff", "#800080", "#ff9e3d"]
      );
    },
    tableData() {
      const ret = [];
      this.data.forEach((d) => {
        const faceDetails = d.FaceDetails[0];
        ret.push({
          timestamp: `${d.timestamp}`,
          AgeRange: `${faceDetails.AgeRange.Low}-${faceDetails.AgeRange.High}`,
          Eyeglasses: `${faceDetails.Eyeglasses.Value}`,
          EyesOpen: `${faceDetails.EyesOpen.Value}`,
          MouthOpen: `${faceDetails.MouthOpen.Value}`,
          Gender: `${faceDetails.Gender.Value}`,
        });
      });
      if (ret.length === 0) {
        return [{}];
      }
      return ret;
    },
  },
};
</script>
<style>
.bg {
  background-image: url("AuroraDark.png");
}
.space-all {
  margin: 4px;
}
.space-top {
  margin-top: 20px;
}
.space-right {
  margin-right: 20px;
}
.space-bottom {
  margin-bottom: 20px;
}
.space-left {
  margin-left: 20px;
}
.floot-rigth {
  text-align: right;
}
.padding-top {
  padding-top: 50px;
}
.padding-rigth {
  padding-right: 50px;
}
.padding-bottom {
  padding-bottom: 50px;
}
</style>
