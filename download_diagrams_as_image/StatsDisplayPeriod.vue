<template>
  <div>
    <v-card>
      <card-title
        icon="mdi-filter-outline"
        :title="$t('Statistics.DisplayPeriod.FilterHeader')"
      >
        <v-spacer />

        <template v-if="!isPanel">
          <v-btn
            v-if="$vuetify.breakpoint.smAndUp"
            @click="downloadStatisticty"
            outlined
            color="primary"
            ><v-icon left>mdi-download</v-icon
            >{{ $t("Statistics.DownloadBtnText") }}
          </v-btn>
          <v-btn v-else icon color="primary" @click="downloadStatisticty"
            ><v-icon>mdi-download</v-icon></v-btn
          >
        </template>
        <v-btn
          :title="$t('Statistics.History.TitleRefreshHistory')"
          color="primary"
          @click="refreshHistory"
          icon
          ><v-icon>mdi-refresh</v-icon></v-btn
        >
      </card-title>
      <v-card-text>
        <stats-filter
          :dates.sync="dates"
          :items="items"
          :value="displayPeriod"
          @change="change"
          :pickerLabel="$t('Statistics.DisplayPeriod.SelectDiapazon')"
          :pickerOk="$t('Statistics.DisplayPeriod.Select')"
        />

        <stats-oee />

        <!-- Diagrams -->
        <v-container>
          <v-row align="center">
            <v-col cols="12" md="6" sm="6">
              <pie-chart
                :data="statusQuantityArray"
                :dark="dark"
                :title="$t('Statistics.Quantity')"
                :formatter="statusQuantityFormatter"
                ref="statusQuantity"
              />
            </v-col>
            <v-col cols="12" md="6" sm="6">
              <pie-chart
                :data="statusTimeArray"
                :dark="dark"
                :title="$t('Statistics.Time')"
                :formatter="statusTimeFormatter"
                ref="statusTime"
              />
            </v-col>
            <v-col cols="12" md="6" sm="6">
              <pie-chart
                :data="statusMaterialArray"
                :dark="dark"
                :title="$t('Statistics.HistoryFilamentUsage')"
                :formatter="statusMaterialFormatter"
                ref="statusMaterial"
              />
            </v-col>
          </v-row>

          <!-- Hidden container for images and canvas elements. It`s needs for "Download statistic" -->
          <v-row class="hidden">
            <img ref="imgForStatusQuantity" class="square-img" />
            <img ref="imgForStatusTime" class="square-img" />
            <img ref="imgForStatusMaterial" class="square-img" />
            <br />canvas:<br />
            <canvas
              ref="allDiadramsAsImage"
              style="border: 1px solid #000"
              :class="`background-color: ${
                $vuetify.theme.dark
                  ? $vuetify.theme.currentTheme.accent
                  : $vuetify.theme.currentTheme.secondary
              }`"
            ></canvas>
            <a style="display: none" ref="linkForDownload">download</a>
          </v-row>
        </v-container>
      </v-card-text>
    </v-card>
  </div>
</template>

<script lang="ts">
import { Component } from "nuxt-property-decorator";
import StatsFilter from '@stereotech/nuxt-common-module/dist/runtime/components/stats/StatsFilter.vue'
import PieChart from '@stereotech/nuxt-common-module/dist/runtime/components/stats/PieChart.vue'
import BaseMixin from '~/components/mixins/base'

@Component({
  components: {
    StatsFilter,
    PieChart
  }
})
export default class StatsDisplayPeriod extends (BaseMixin) {

  $refs!: {
    statusQuantity: any;
    statusTime: any;
    statusMaterial: any;
    allDiadramsAsImage: HTMLCanvasElement;
    imgForStatusQuantity: HTMLImageElement;
    imgForStatusTime: HTMLImageElement;
    imgForStatusMaterial: HTMLImageElement;
    linkForDownload: HTMLLinkElement;
  };

  get dark () {
    return this.$vuetify.theme.dark
  }

  get showPicker (): boolean {
    return this.displayPeriod === "period";
  }

  // data for select
  get items (): { text: string; value: string }[] {
    return [
      {
        text: this.$t("Statistics.DisplayPeriod.Day").toString(),
        value: "day"
      },
      {
        text: this.$t("Statistics.DisplayPeriod.Month").toString(),
        value: "month"
      },
      {
        text: this.$t("Statistics.DisplayPeriod.Year").toString(),
        value: "year"
      },
      {
        text: this.$t("Statistics.DisplayPeriod.Period").toString(),
        value: "period"
      },
      {
        text: this.$t("Statistics.DisplayPeriod.All").toString(),
        value: "all"
      }
    ];
  }

  // Период, например ['2023-01-12', '2023-01-13']
  get dates (): string[] {
    return this.$store.state.gui.printjobs.arrPeriodRange;
  }

  set dates (val: string[]) {
    val.sort();
    this.$store.dispatch("gui/saveSetting", {
      name: "printjobs.arrPeriodRange",
      value: val
    });
  }

  nowDate () {
    let now = new Date();
    return `${now.getFullYear()}-${now.getMonth() + 1}-${now.getDate()}`;
  }

  get dateRangeText (): string {
    return this.dates.join(" ~ ");
  }

  // За какой период отображать статистику. Cтартовое значение.
  get displayPeriod () {
    return this.$store.state.gui.printjobs.displayPeriod;
  }

  // В дочернем произошло изменение displayPeriod
  change (val: any) {
    this.$store.dispatch("gui/saveSetting", {
      name: "printjobs.displayPeriod",
      value: val
    });
  }

  get statusQuantityArray () {
    let arr = this.$store.getters["server/printjobs/getAllPrintStatusArray"].map(
      (i: any) => {
        return {
          ...i,
          name: this.$t(i.name)
        };
      }
    );
    return arr;
  }

  statusQuantityFormatter = (params: any) => {
    return params.marker + ' ' + params.data.name + ' ' + params.data.value
  }

  get statusTimeArray () {
    return this.$store.getters["server/printjobs/getAllPrintStatusTime"].map(
      (i: any) => {
        return {
          ...i,
          name: this.$t(i.name)
        };
      }
    );
  }

  statusTimeFormatter = (params: any) => {
    return params.marker + ' ' + params.data.name + '<br/>' + this.$helpers.formatPrintTime(params.data.value) + '<br/>' + params.percent + '%'
  }

  get statusMaterialArray () {
    return this.$store.getters["server/printjobs/getMaterialStatusArray"].map(
      (i: any) => {
        return {
          ...i,
          name: this.$t(i.name)
        };
      }
    );
  }

  statusMaterialFormatter = (params: any) => {
    return params.marker + ' ' + params.data.name + '<br/>' + (params.data.value / 1000).toFixed(2) + 'm<br/>' + params.percent + '%'
  }

  refreshHistory () {
    this.$socket.emit(
      "server.printjobs.list",
      { start: 0, limit: 50 },
      { action: "server/printjobs/getPrintjobs" }
    );
  }

  downloadStatisticty () {
    let padding: number = 10;
    let paddingTop: number = 30;
    this.drawImages(
      this.$refs.imgForStatusQuantity,
      this.$refs.statusQuantity?.getImage(),
      padding,
      paddingTop
    );
    this.drawImages(
      this.$refs.imgForStatusTime,
      this.$refs.statusTime?.getImage(),
      this.imgGeometry.width,
      paddingTop
    );
    this.drawImages(
      this.$refs.imgForStatusMaterial,
      this.$refs.statusMaterial?.getImage(),
      padding,
      this.imgGeometry.height + paddingTop
    );
    if (this.canvasCTX) {
      let width = this.imgGeometry.width * 2 + 10;
      let height = this.imgGeometry.height * 2 + 20;
      this.$refs.allDiadramsAsImage.width = width;
      this.$refs.allDiadramsAsImage.height = height;
      this.canvasCTX.fillStyle = this.$vuetify.theme.dark ? "black" : "white"
      this.canvasCTX.fillRect(0, 0, width, height);
      this.canvasCTX.font = "18px Roboto";
      this.canvasCTX.fillStyle = this.$vuetify.theme.dark ? "white" : "black";
      this.canvasCTX?.fillText(
        this.$t("Statistics.PrinterName") + ": " + this.printerName,
        10,
        20
      );
    }
    setTimeout(() => {
      // Поместим ссылку на канвас в <a> и кликнем по ней
      this.$refs.linkForDownload.setAttribute(
        "href",
        this.$refs.allDiadramsAsImage.toDataURL()
      );
      const d = new Date();
      const fileName =
        this.printerName +
        "-" +
        d.getDate() +
        "-" +
        (d.getMonth() + 1) +
        "-" +
        d.getFullYear() +
        "-" +
        d.toTimeString().split(" ")[0];
      this.$refs.linkForDownload.setAttribute("download", fileName);
      this.$refs.linkForDownload.click();
    }, 20);
  }
  drawImages (img: HTMLImageElement, src: string, x: number = 0, y: number = 0) {
    img.src = src;
    img.width = this.imgGeometry.width;
    img.height = this.imgGeometry.height;
    img.onload = () => {
      this.canvasCTX?.drawImage(img, x, y, img.width, img.height);
    };
  }

  get imgGeometry () {
    // Find out the size of each of the square diagrams - it will be equal to the size of the FIRST diagram
    return this.$refs.statusQuantity?.getGeometry();
  }
  get canvasCTX () {
    return this.$refs.allDiadramsAsImage.getContext("2d");
  }
}
</script>

<style>
.hidden {
  display: none !important;
}
</style>
