<script lang="ts">
import { Vue, Component, Prop } from "vue-property-decorator";
import { Bar } from "vue-chartjs";

@Component({
  extends: Bar,
})
export default class BarChart extends Vue {
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
