<template>
  <div class="bg-gray-100 h-screen bg-center bg-cover bg-[url('/bg.avif')]">
    <div
      class="flex items-center justify-center h-full w-full backdrop-blur-md"
    >
      <div
        class="relative p-4 rounded-lg shadow-lg bg-[#e0e1d8] border-2 border-gray-300"
      >
        <lcdRGB ref="lcdRGB" :options="options" />
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, onMounted, ref } from "vue";
import type { Options } from "./types";
import lcdRGB from "./components/lcdRGB.vue";

export default defineComponent({
  name: "App",
  components: {
    lcdRGB,
  },
  setup() {
    const lcdRGB: any = ref(null);

    /**
     * Options for the LCD screen
     */
    const options: Options = {
      backgroundColor: "#87ad34",
      borderColor: "#303030",
      borderSize: "10px",
      columns: 20,
      pixelOffColor: "#7b9f31",
      pixelOnColor: "#0b4233",
      pixelSize: 2,
      rows: 4,
    };

    onMounted(() => {
      if (lcdRGB.value) {
        lcdRGB.value.writeText("vue-lcdRGB", 0, true);
        lcdRGB.value.writeText("by Tony Myers", 2);
        lcdRGB.value.blink(true, 2, 13);
        lcdRGB.value.rotateRow("https://github.com/tony-nz ", 3, 500);
      }
    });

    return {
      lcdRGB,
      options,
    };
  },
});
</script>
