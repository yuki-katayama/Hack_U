// GraphDoughnut.vue
<script>
// ここでこのコンポーネントで使用するグラフの種類を定義する。今回はドーナツグラフなのでDoughnutとなる。
import { Doughnut, mixins } from "vue-chartjs";

export default {
  extends: Doughnut,
  props: {
    chartdata: {
      type: Object,
      default: null,
    },
    options: {
      type: Object,
      default: null,
    },
  },
  mounted() {},
  watch: {
    chartdata() {
      this.renderChart(this.chartdata, this.options);
    },
  },
};
</script>

