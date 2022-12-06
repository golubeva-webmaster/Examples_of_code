<template>
  <div>
    <v-card>
      <card-title
        icon="mdi-webcam"
        :title="$t('Camera.Camera')"
        :id="'webcam-card'"
      >
        <v-spacer></v-spacer>
        <v-btn v-if="enableLightControl" @click="zoomInImage" icon>
          <v-icon v-if="isZoomed">mdi-magnify-minus-outline</v-icon>
          <v-icon v-else>mdi-magnify-plus-outline</v-icon>
        </v-btn>
        <v-btn
          v-if="enableLightControl"
          icon
          @click="lightControl"
          :color="lightColor"
        >
          <v-icon v-if="lightIsOff">mdi-lightbulb-outline</v-icon>
          <v-icon v-else>mdi-lightbulb-off-outline</v-icon>
        </v-btn>
      </card-title>
    </v-card>

    <div v-intersect="onIntersect" style="position: relative" class="rotate">
      <div class="text-center py-5" v-if="!isLoaded">
        <v-progress-circular
          indeterminate
          color="primary"
        ></v-progress-circular>
      </div>
      <canvas
        ref="mjpegstreamerAdaptive"
        width="600"
        height="400"
        :class="
          'webcamImage ' +
          (isLoaded ? '' : 'hiddenWebcam') +
          (isZoomed ? 'zoomedWebcam' : '')
        "
      ></canvas>
    </div>
  </div>
</template>

<script lang="ts">
import { Vue, Component, mixins, Prop } from "nuxt-property-decorator";

@Component({
  name: 'WebcamCard'
})
export default class WebcamCard extends Vue {
  @Prop({ default: false }) enableLightControl!: boolean
  @Prop({ type: String, default: '' }) apiUrl!: string
  @Prop({
    type: Object, default: () => {
      return {
        boolDashboard: false,
        boolNavi: false,
        url: '/webcam/?action=stream',
        rotate: 90
      }
    }
  }) camSettings!: { boolDashboard: boolean, boolNavi: boolean, url: string, rotate: number }
  @Prop({ type: String, default: '' }) lightColor!: string

  isVisible = false;
  refresh = Math.ceil(Math.random() * Math.pow(10, 12));
  private isLoaded = true;
  private timer: any = undefined;
  private request_start_time = performance.now();
  private start_time = performance.now();
  private time = 0;
  private request_time = 0;
  private time_smoothing = 0.6;
  private request_time_smoothing = 0.1;
  private currentFPS = 0;
  showFps = true;
  isLightTurnOn = false;

  isZoomed = false;
  zoomInImage () {
    this.isZoomed = !this.isZoomed
  }

  get url () {
    const baseUrl = this.camSettings.url;
    return "http:" + this.apiUrl + baseUrl;
  }

  $refs!: {
    mjpegstreamerAdaptive: any;
  };

  onIntersect (
    entries: IntersectionObserverEntry[],
    observer: IntersectionObserver
  ) {
    const isVisible = entries[0].isIntersecting;
    this.isVisible = isVisible;
    if (isVisible) {
      this.refreshFrame();
    } else {
      clearTimeout(this.timer);
      this.timer = undefined;
    }
  }

  device: any = null;
  deviceId: any = null;

  onCameras (cameras: any) {
    this.device = cameras.length > 0 ? cameras[0] : null;
    this.deviceId = this.device.deviceId;
  }

  refreshFrame () {
    if (this.isVisible) {
      this.refresh = new Date().getTime();
      this.setFrame();
    }
  }

  async setFrame () {
    const baseUrl = this.camSettings.url;
    const url = new URL(this.url);

    url.searchParams.append("bypassCache", this.refresh.toString());
    url.searchParams.set("action", "snapshot");

    this.request_start_time = performance.now();
    this.currentFPS = Math.round(1000 / this.time);

    let canvas = this.$refs.mjpegstreamerAdaptive;
    if (canvas) {
      const ctx = canvas.getContext("2d");
      const frame: any = await this.loadImage(url.toString());

      canvas.width = canvas.clientWidth;
      canvas.height = canvas.clientWidth * (frame.height / frame.width);

      ctx?.drawImage(
        frame,
        0,
        0,
        frame.width,
        frame.height,
        0,
        0,
        canvas.width,
        canvas.height
      );
      this.isLoaded = true;
    }

    this.$nextTick(() => {
      this.onLoad();
    });
  }

  loadImage (url: string) {
    return new Promise(r => {
      let image = new Image();
      image.onload = () => r(image);
      image.onerror = () => setTimeout(this.refreshFrame, 1000);
      image.src = url;
    });
  }

  onLoad () {
    this.isLoaded = true;

    const end_time = performance.now();
    const current_time = end_time - this.start_time;
    this.time =
      this.time * this.time_smoothing +
      current_time * (1.0 - this.time_smoothing);
    this.start_time = end_time;

    const target_time = 100;

    const current_request_time = performance.now() - this.request_start_time;
    this.request_time =
      this.request_time * this.request_time_smoothing +
      current_request_time * (1.0 - this.request_time_smoothing);
    const timeout = Math.max(0, target_time - this.request_time);

    this.$nextTick(() => {
      this.timer = setTimeout(this.refreshFrame, timeout);
    });
  }

  get lightIsOff () {
    return this.lightColor === 'rgb(0, 0, 0)'
  }

  lightControl () {
    const gcode = 'TOGGLE_LIGHT'
    this.$emit('lightControl', { script: gcode }, { message: gcode, type: 'command' })
  }
}
</script>

<style scoped>
.rotate {
  transform: translateY(40%) rotate(-90deg);
  overflow: hidden;
}

.webcamImage {
  width: 100%;
  border: 1px solid #ccc;
}

.webcamFpsOutput {
  display: inline-block;
  position: absolute;
  bottom: 6px;
  right: 0;
  background: rgba(0, 0, 0, 0.8);
  padding: 3px 10px;
  border-top-left-radius: 5px;
}

.zoomedWebcam {
  -webkit-transform: scale(2.5); /* Safari and Chrome */
  -moz-transform: scale(2.5); /* Firefox */
  -ms-transform: scale(2.5); /* IE 9 */
  -o-transform: scale(2.5); /* Opera */
  transform: scale(2.5);
}
</style>
