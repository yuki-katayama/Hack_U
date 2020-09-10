<script>
import { Line, mixins } from "vue-chartjs";
export default {
  extends: Line,
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
