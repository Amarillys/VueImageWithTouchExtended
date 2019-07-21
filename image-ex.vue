<template>
  <div
    class="image-ex-container"
    ref="container"
    @touchstart="touchStart"
    @touchmove="touchMove"
    @dblclick="zoomAuto"
    @mousedown="mousedownStart"
    @mousemove="mouseMove"
    @mouseup="mouseUp"
  >
    <img :src="src" class="image-ex-img" ref="imgex">
  </div>
</template>
<script>
export default {
  props: {
    src: String,
    fadeTime: {
      type: Number,
      default: () => 300
    },
    maxScale: {
      type: Number,
      default: () => 4
    },
    minScale: {
      type: Number,
      default: () => 0.25
    },
    classList: {
      type: Array,
      default: () => []
    },
    step: {
      type: Number,
      default: () => 0.05
    }
  },
  data() {
    return {
      scale: 1,
      lastX: null,
      lastY: null,
      lastZoomScale: null,
      inDrag: false
    };
  },
  mounted() {
    this.classList.forEach(className =>
      this.$refs.container.classList.add(className)
    );
    this.$refs.imgex.style.transitionDuration = `${this.fadeTime / 1000}s`;
  },
  methods: {
    mousedownStart(event) {
      this.lastX = null;
      this.lastY = null;
      this.inDrag = true;
      event.preventDefault();
    },
    mouseMove(event) {
      if (!this.inDrag) return;
      this.doMove(event.movementX, event.movementY);
      event.preventDefault();
    },
    mouseUp(event) {
      this.inDrag = false;
      event.preventDefault();
    },
    touchStart() {
      this.lastX = null;
      this.lastY = null;
      this.lastZoomScale = null;
    },
    zoom(event) {
      this.scale += event.deltaY * -this.step;
      this.scale = Math.min(Math.max(this.minScale, this.scale), this.maxScale);
      this.$refs.imgex.style.transform = `scale(${this.scale})`;
    },
    zoomAuto(event) {
      switch (this.scale) {
        case 1:
          this.scale = 2;
          break;
        case 2:
          this.scale = 4;
          break;
        default:
        case 4:
          this.scale = 1;
          break;
      }
      this.$refs.imgex.style.transform = `scale(${this.scale})`;
      event.preventDefault();
    },
    touchMove(event) {
      event.preventDefault();
      if (this.lastX === null) {
        this.lastX = event.targetTouches[0].pageX;
        this.lastY = event.targetTouches[0].pageY;
        return;
      }
      if (event.targetTouches.length === 2) {
        let zoom = this.gesturePinchZoom(event);
        this.$refs.imgex.style.transform = `scale(${zoom})`;
      } else if (event.targetTouches.length === 1) {
        let x = event.targetTouches[0].pageX - this.lastX;
        let y = event.targetTouches[0].pageY - this.lastY;
        this.lastX = event.targetTouches[0].pageX;
        this.lastY = event.targetTouches[0].pageY;
        this.doMove(x, y);
      }
    },
    doMove(x, y) {
      let style = this.$refs.imgex.style;
      style.left = `${+style.left.replace("px", "") + x}px`;
      style.top = `${+style.top.replace("px", "") + y}px`;
    },
    gesturePinchZoom: function(event) {
      let zoom = 1;
      if (event.targetTouches.length >= 2) {
        let p1 = event.targetTouches[0];
        let p2 = event.targetTouches[1];
        let zoomScale = Math.sqrt(
          Math.pow(p2.pageX - p1.pageX, 2) + Math.pow(p2.pageY - p1.pageY, 2)
        ); //euclidian distance

        if (this.lastZoomScale) {
          zoom = zoomScale - this.evtData.lastZoomScale;
        }
        this.lastZoomScale = zoomScale;
      }
      return zoom;
    }
  }
};
</script>
<style>
.image-ex-container {
  position: relative;
  margin: auto;
  overflow: hidden;
}

.image-ex-img {
  position: absolute;
  width: 100%;
  top: 0;
  left: 0;
  transition: all ease-in;
}
</style>
