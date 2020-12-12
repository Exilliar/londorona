<script lang="ts">
import { Vue, Component, Prop } from "vue-property-decorator";
import { Line } from "vue-chartjs";

@Component({
  extends: Line,
})
export default class LineChart extends Vue {
  @Prop() data!: any;
  options = {
    type: Object,
    default: null,
  };
  mounted() {
    console.log("data from component:", this.data);
    this.renderChart(this.data, this.options);
  }
}
</script>
