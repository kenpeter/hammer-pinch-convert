<template>
    <div class="product-images-popup">
        <div class="product-images-popup-modal">
            <div class="product-images-popup-header">

            </div>

            <div class="product-images-popup-body">
                <div class="product-images-popup-carousel_wrap">
                    <h1>hi</h1>
                    <div id="slider">
                        <transition-group tag="div" :name="transitionName" class="slides-group">
                            <div v-if="show" :key="current" class="slide">

                                <div
                                        id="zoomImageContainer"
                                        @load="imageContainerLoad"
                                        @wheel="imageContainerWheel"

                                        v-hammer:pan="panStart"
                                        v-hammer:panend="panEnd"
                                        v-hammer:pancancel="panEnd"

                                        v-hammer:pinch="pinchStart"
                                        v-hammer:pinchmove="pinchStart"

                                        v-hammer:pinchcancel="pinchEnd"
                                        v-hammer:pinchend="pinchEnd"

                                >
                                    <img
                                            id="theZoomImage"
                                            sizes="100vw"
                                            :srcset="images[current]"
                                            :src="images[current]"
                                            @load="displayImageLoad"
                                            @mousedown="displayImageMouseDown"
                                    />
                                </div>

                            </div>
                        </transition-group>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
  import Vue from 'vue'
  import Vue2TouchEvents from 'vue2-touch-events'
  import { VueHammer } from 'vue2-hammer'

  VueHammer.config.pinch = {
    enable: true
  };

  VueHammer.config.pan = {
    direction: ((2|4) | (8|16))
  };

  Vue.use(Vue2TouchEvents)
  Vue.use(VueHammer)

  export default {
    name: 'PopUpCarousel',

    data() {
      return {
        current: 0,
        direction: 1,
        transitionName: "fade",
        show: false,
        images: [],

        // zoom var global
        displayImage: false,
        imageContainer: false,

        // zoom var
        minScale: 1,
        maxScale: 4,

        imageWidth: 0,
        imageHeight: 0,

        containerWidth: 0,
        containerHeight: 0,

        displayImageX: 0,
        displayImageY: 0,
        displayImageScale: 1, // origin scale is 1

        displayDefaultWidth: false,
        displayDefaultHeight: false,

        rangeX: 0,
        rangeMaxX: 0,
        rangeMinX: 0,

        rangeY: 0,
        rangeMaxY: 0,
        rangeMinY: 0,

        displayImageRangeY: 0,

        displayImageCurrentX: 0,
        displayImageCurrentY: 0,
        displayImageCurrentScale: 1,
      }
    },

    // compute
    computed: {

    },

    methods: {
      slide(dir) {
        this.direction = dir

        dir === 1
          ? (this.transitionName = "slide-next")
          : (this.transitionName = "slide-prev")

        // images
        let len = this.images.length

        this.current = (this.current + dir % len + len) % len
      },

      resizeContainer() {
        // test
        console.log('-- resizeContainer --')

        // img container outer width
        this.containerWidth = this.imageContainer.offsetWidth;
        // img container outer height
        this.containerHeight = this.imageContainer.offsetHeight;

        // img outer width
        // img outer height
        if (this.displayDefaultWidth !== false && this.displayDefaultHeight !== false) {
          // img outer width
          this.displayDefaultWidth = this.displayImage.offsetWidth;
          // img outer height
          this.displayDefaultHeight = this.displayImage.offsetHeight;

          // update range
          this.updateRange();

          // display img curr x
          // img x, min_x, max_x
          this.displayImageCurrentX = this.clamp( this.displayImageX, this.rangeMinX, this.rangeMaxX );

          // display img curr y
          // img y, min_y, max_y
          this.displayImageCurrentY = this.clamp( this.displayImageY, this.rangeMinY, this.rangeMaxY );

          // update display img
          this.updateDisplayImage(
            // img x
            this.displayImageCurrentX,
            // img y
            this.displayImageCurrentY,
            // img scale
            this.displayImageCurrentScale );
        }
      },

      clamp(value, min, max) {
        // min: max(min vs val) VS max
        return Math.min(Math.max(min, value), max);
      },

      clampScale(newScale) {
        return this.clamp(newScale, this.minScale, this.maxScale);
      },

      updateRange() {
        this.rangeX = Math.max(0, Math.round(this.displayDefaultWidth * this.displayImageCurrentScale) - this.containerWidth);
        this.rangeY = Math.max(0, Math.round(this.displayDefaultHeight * this.displayImageCurrentScale) - this.containerHeight);

        this.rangeMaxX = Math.round(this.rangeX / 2);
        this.rangeMinX = 0 - this.rangeMaxX;

        this.rangeMaxY = Math.round(this.rangeY / 2);
        this.rangeMinY = 0 - this.rangeMaxY;
      },

      updateDisplayImage(x, y, scale) {
        const transform = 'translateX(' + x + 'px) translateY(' + y + 'px) translateZ(0px) scale(' + scale + ',' + scale + ')';
        this.displayImage.style.transform = transform;
        this.displayImage.style.WebkitTransform = transform;
        this.displayImage.style.msTransform = transform;
      },

      imageContainerLoad() {
        // test
        console.log('-- imageContainerLoad --');

        this.imageContainer = document.querySelector('#zoomImageContainer');
        this.containerWidth = this.imageContainer.offsetWidth;
        this.containerHeight = this.imageContainer.offsetHeight;
      },

      imageContainerWheel(e) {
        // test
        console.log('-- imageContainerWheel --');

        this.displayImageScale = this.displayImageCurrentScale = this.clampScale(this.displayImageScale + (e.wheelDelta / 800));
        this.updateRange();
        this.displayImageCurrentX = this.clamp(this.displayImageCurrentX, this.rangeMinX, this.rangeMaxX)
        this.displayImageCurrentY = this.clamp(this.displayImageCurrentY, this.rangeMinY, this.rangeMaxY)
        this.updateDisplayImage(this.displayImageCurrentX, this.displayImageCurrentY, this.displayImageScale);
      },

      displayImageLoad(event) {
        // test
        console.log('-- displayImageLoad --');

        // not the best, but it works
        this.displayImage = document.querySelector('#theZoomImage');

        // global img w
        this.imageWidth = this.displayImage.width;
        // global img h
        this.imageHeight = this.displayImage.height;

        // outer width
        this.displayDefaultWidth = this.displayImage.offsetWidth;
        // outer height
        this.displayDefaultHeight = this.displayImage.offsetHeight;

        // the container w????
        this.rangeX = Math.max(0, this.displayDefaultWidth - this.containerWidth);

        this.rangeY = Math.max(0, this.displayDefaultHeight - this.containerHeight);
      },

      displayImageMouseDown() {
        e => e.preventDefault()
      },

      panStart(ev) {
        this.displayImageCurrentX = this.clamp(this.displayImageX + ev.deltaX, this.rangeMinX, this.rangeMaxX);
        this.displayImageCurrentY = this.clamp(displayImageY + ev.deltaY, this.rangeMinY, this.rangeMaxY);
        this.updateDisplayImage(this.displayImageCurrentX, this.displayImageCurrentY, this.displayImageScale);
      },

      panEnd(ev) {
        this.displayImageScale = this.displayImageCurrentScale;
        this.displayImageX = this.displayImageCurrentX;
        this.displayImageY = this.displayImageCurrentY;
      },

      pinchStart(ev) {
        this.displayImageCurrentScale = this.clampScale(ev.scale * this.displayImageScale);

        this.updateRange();

        this.displayImageCurrentX = this.clamp(this.displayImageX + ev.deltaX, this.rangeMinX, this.rangeMaxX);
        this.displayImageCurrentY = this.clamp(this.displayImageY + ev.deltaY, this.rangeMinY, this.rangeMaxY);

        this.updateDisplayImage(this.displayImageCurrentX, this.displayImageCurrentY, this.displayImageCurrentScale);

      },

      pinchEnd() {
        this.displayImageScale = this.displayImageCurrentScale;
        this.displayImageX = this.displayImageCurrentX;
        this.displayImageY = this.displayImageCurrentY;
      }
    },

    mounted() {
      //test
      console.log('-- mounted --');

      this.show = true
      this.images = [
        'http://www.orseu-concours.com/452-616-thickbox/selor-test-de-raisonnement-abstrait-niveau-d.jpg'
      ]

      this.resizeContainer()
      window.addEventListener('resize', this.resizeContainer, true)
    },

    destroyed: function() {
      window.removeEventListener('resize', this.resizeContainer, true)
    }
  };
</script>



