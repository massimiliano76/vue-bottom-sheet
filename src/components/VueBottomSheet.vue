<template>
  <div
    :class="{
      opened: opened,
      closed: opened === false,
      moving: moving
    }"
    @click="close"
    class="bottom-sheet"
    ref="bottomSheet"
  >
    <div v-if="overlay" ref="backdrop" class="bottom-sheet__backdrop"></div>
    <div
      :style="{ bottom: cardP, maxWidth: `${maxWidth}px` }"
      :class="{ stripe: stripe }"
      ref="bottomSheetCard"
      class="bottom-sheet__card"
    >
      <div class="bottom-sheet__pan" ref="pan">
        <div class="bottom-sheet__bar" ref="bar"></div>
      </div>
      <div
        :style="{ height: contentH }"
        ref="bottomSheetCardContent"
        class="bottom-sheet__content"
        @click="close"
      >
        <slot />
      </div>
    </div>
  </div>
</template>

<script>
import Hammer from "hammerjs";

export default {
  name: "VueBottomSheet",
  data() {
    return {
      opened: null,
      contentH: "auto",
      mc: null,
      cardP: null,
      cardH: null,
      moving: false,
      stripe: 0
    };
  },
  props: {
    overlay: {
      type: Boolean,
      default: true
    },
    maxWidth: {
      type: Number,
      default: 640
    }
  },
  mounted() {
    let iPhone = /iPhone/.test(navigator.userAgent) && !window.MSStream;
    let aspect = window.screen.width / window.screen.height;
    if (iPhone && aspect.toFixed(3) === "0.462") {
      this.stripe = 20;
    }

    this.cardH = this.$refs.bottomSheetCard.clientHeight;
    this.contentH = `${this.cardH - this.$refs.pan.clientHeight}px`;
    this.cardP = `-${this.cardH + this.stripe}px`;

    this.mc = new Hammer(this.$refs.pan);
    this.mc.get("pan").set({ direction: Hammer.DIRECTION_ALL });
    this.mc.on("panstart", () => {
      this.moving = true;
    });
    this.mc.on("panup pandown", evt => {
      const panPosition =
        this.$refs.backdrop.clientHeight - this.cardH - evt.center.y;
      if (panPosition < 0) {
        this.cardP = `${panPosition}px`;
      }
    });
    this.mc.on("panend", evt => {
      this.moving = false;
      const panP = this.$refs.bottomSheet.clientHeight - this.cardH - evt.center.y;
      if (panP < -30) {
        this.opened = false;
        this.cardP = `-${this.cardH + this.stripe}px`;
      }
    });
  },
  methods: {
    open() {
      this.opened = true;
      this.cardP = 0;
    },
    close() {
      this.opened = false;
      this.cardP = `-${this.cardH + this.stripe}px`;
    }
  },
  beforeDestroy() {
    this.mc.destroy();
  }
};
</script>

<style lang="scss" scoped>
.bottom-sheet {
  z-index: 99999;

  &__content {
    overflow-y: scroll;
  }

  &__backdrop {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: rgba(0, 0, 0, 0.3);
    z-index: 9999;
    opacity: 0;
    visibility: hidden;
  }

  &__card {
    box-sizing: border-box;
    width: 100%;
    position: fixed;
    background: white;
    border-radius: 14px 14px 0 0;
    left: 50%;
    transform: translate(-50%, 0);
    z-index: 9999;
    max-height: 95%;
    transition: bottom 0.3s ease;
    margin: 0 auto;

    &.stripe {
      padding-bottom: 20px;
    }
  }

  &__pan {
    padding-bottom: 20px;
    padding-top: 15px;
    height: 38px;
  }

  &__bar {
    display: block;
    width: 50px;
    height: 3px;
    border-radius: 14px;
    margin: 0 auto;
    cursor: pointer;
    background: rgba(0, 0, 0, 0.3);
  }

  &.closed {
    .bottom-sheet__backdrop {
      animation: hide 0.3s ease;
    }
  }

  &.moving {
    .bottom-sheet__card {
      transition: none;
    }
  }

  &.opened {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;

    .bottom-sheet__backdrop {
      animation: show 0.3s ease;
      opacity: 1;
      visibility: visible;
    }
  }
}

@keyframes show {
  0% {
    opacity: 0;
    visibility: hidden;
  }

  100% {
    opacity: 1;
    visibility: visible;
  }
}

@keyframes hide {
  0% {
    opacity: 1;
    visibility: visible;
  }

  100% {
    opacity: 0;
    visibility: hidden;
  }
}
</style>
