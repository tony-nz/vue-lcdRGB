<template>
  <div ref="lcdScreen" />
</template>

<script lang="ts">
import { defineComponent, onMounted, ref } from "vue";
import lcdLetters from "@/helpers/letters";
import type { Options } from "@/types";

export default defineComponent({
  name: "lcdRGB",
  props: {
    id: {
      type: String,
      required: false,
      default: "lcdScreen",
    },
    options: {
      type: Object as () => Options,
      required: false,
    },
  },
  setup(_props) {
    const lcdScreen = ref<HTMLElement | null>(null);
    const options: Options = _props.options || {
      backgroundColor: "#87ad34",
      borderColor: "#303030",
      borderSize: "10px",
      columns: 20,
      pixelOffColor: "#7b9f31",
      pixelOnColor: "#0b4233",
      pixelSize: 2,
      rows: 4,
    };

    /**
     * Blink the LCD screen
     * @param state boolean
     * @param row number
     * @param letterIndex number
     */
    const blink = (state: boolean, row: number, letterIndex: number) => {
      if (!lcdScreen.value) return;

      const lcdElement = lcdScreen.value.querySelector(
        `#${_props.id}-letter${row.toString().padStart(2, "0")}-${letterIndex
          .toString()
          .padStart(2, "0")}`
      );
      if (!lcdElement) {
        console.warn(
          `LCD element with ID ${_props.id}-letter${row
            .toString()
            .padStart(2, "0")}-${letterIndex
            .toString()
            .padStart(2, "0")} not found.`
        );
        return;
      }

      if (state) {
        lcdElement.classList.add("blink");
      } else {
        lcdElement.classList.remove("blink");
      }
    };

    /**
     * Rotate the text in a row
     * @param text string
     * @param row number
     */
    const rotateRow = (text: string, row: number, speed: number = 200) => {
      // Update the row content cyclically
      const rotatedText = text.substring(1) + text.substring(0, 1);
      writeText(rotatedText, row);

      // Schedule the next rotation after speed (default 200ms)
      setTimeout(() => {
        rotateRow(rotatedText, row);
      }, speed);
    };

    /**
     * Create a new LCD screen
     * @param name string
     * @param place string
     */
    const lcdNew = (name: string, place: HTMLElement | null) => {
      if (!place || !lcdScreen.value) return; // Check if lcdScreen and place are initialized

      lcdScreen.value.innerHTML = ""; // Clear previous content if any
      const lcdElement = document.createElement("div");

      lcdElement.id = name;
      lcdElement.classList.add(
        "mx-auto",
        "p-2",
        "space-y-1",
        "bg-[" + options.backgroundColor + "]",
        "text-[0px]",
        "border-[" + options.borderSize + "]",
        "border-solid",
        "border-[" + options.borderColor + "]",
        "rounded-md",
        "pixelated"
      );

      // Append the LCD element to the place element
      place.appendChild(lcdElement);

      for (let g = 0; g < options.rows; g++) {
        const LCDRowIndex = g < 10 ? `0${g}` : `${g}`;
        const LCDRowElement = document.createElement("div");

        LCDRowElement.id = `${name}-row${LCDRowIndex}`; // Example: "lcdName-row00"
        LCDRowElement.classList.add("block", "border-none", "p-0", "mx-auto");
        lcdElement.appendChild(LCDRowElement);

        for (let h = 0; h < options.columns; h++) {
          const letterIndex = h < 10 ? `0${h}` : `${h}`;
          const letterElement = document.createElement("div");

          letterElement.id = `${name}-letter${LCDRowIndex}-${letterIndex}`; // Example: "lcdName-letter00-00"
          letterElement.classList.add(
            "text-[0px]",
            "inline-block",
            "ml-0.5",
            "w-[15px]"
          );
          LCDRowElement.appendChild(letterElement);

          for (let j = 0; j < 8; j++) {
            const rowIndex = j < 10 ? `0${j}` : `${j}`;
            const rowElement = document.createElement("div");

            rowElement.id = `${name}-row${LCDRowIndex}-letter${letterIndex}-row${rowIndex}`; // Example: "lcdName-row00-letter00-row00"
            rowElement.classList.add("letter-row");
            letterElement.appendChild(rowElement);

            for (let i = 0; i < 5; i++) {
              const columnIndex = i < 10 ? `0${i}` : `${i}`;
              const pixel = document.createElement("div");
              pixel.id = `${name}-row${LCDRowIndex}-letter${letterIndex}-row${rowIndex}-column${columnIndex}`; // Example: "lcdName-row00-letter00-row00-column00"
              pixel.classList.add(
                "pixel",
                "inline-block",
                "bg-[" + options.pixelOffColor + "]",
                "h-[" + options.pixelSize + "px]",
                "w-[" + options.pixelSize + "px]",
                "border-0",
                "p-0",
                "m-0",
                "ml-[1px]",
                "mt-[1px]"
              );
              rowElement.appendChild(pixel);
            }
          }
        }
      }
    };

    /**
     * Write a letter to the LCD screen
     * @param letter string
     * @param row number
     * @param index number
     * @param lcdID string
     */
    const writeLetter = (
      letter: string,
      row: number,
      index: number,
      lcdID: string
    ) => {
      if (!lcdLetters[letter]) return; // Check if letter exists in dictionary
      if (!lcdScreen.value) return; // Ensure lcdScreen is initialized

      const lcdElement = document.getElementById(lcdID);
      if (!lcdElement) {
        console.warn(`LCD element with ID ${lcdID} not found.`);
        return;
      }

      const LCDRowIndex = row < 10 ? `0${row}` : `${row}`;
      const letterIndex = index < 10 ? `0${index}` : `${index}`;

      const letterElement = lcdElement.querySelector(
        `#${lcdID}-letter${LCDRowIndex}-${letterIndex}`
      );
      if (!letterElement) {
        // console.warn(
        //   `Letter element with ID ${lcdID}-letter${LCDRowIndex}-${letterIndex} not found.`
        // );
        return;
      }

      const pixelArray = lcdLetters[letter];
      const pixelElements = letterElement.querySelectorAll(".pixel");

      pixelElements.forEach((pixelElement, idx) => {
        const rowIndex = Math.floor(idx / 5); // Calculate the row index within the letter
        const columnIndex = idx % 5; // Calculate the column index within the letter
        if (pixelArray[rowIndex].charAt(columnIndex) === "1") {
          pixelElement.classList.add("bg-[" + options.pixelOnColor + "]");
        } else {
          pixelElement.classList.remove("bg-[" + options.pixelOnColor + "]");
        }
      });
    };

    /**
     * Write text to the LCD screen
     * @param text The text to write
     * @param rowIndex The row index to write the text to
     * @param center Center the text on the screen
     */
    const writeText = (
      text: string,
      rowIndex: number = 0,
      center: boolean = false
    ) => {
      // center text if required
      if (center) {
        const textLength = text.length;
        const padding = Math.floor((options.columns - textLength) / 2);
        text = " ".repeat(padding) + text + " ".repeat(padding);
      }
      for (let i = 0; i < text.length; i++) {
        const letterIndex = i;
        writeLetter(text.charAt(i), rowIndex, letterIndex, _props.id);
      }
    };

    onMounted(() => {
      lcdNew(_props.id, lcdScreen.value);
    });

    return {
      blink,
      lcdScreen,
      lcdNew,
      rotateRow,
      writeLetter,
      writeText,
    };
  },
});
</script>

<style>
.blink .pixel {
  animation: blink 3s steps(1, start) infinite;
  -webkit-animation: blink 3s steps(1, start) infinite;
}

@keyframes blink {
  50% {
    background-color: #0b4233;
  }
  100% {
  }
}

@-webkit-keyframes blink {
  50% {
    background-color: #0b4233;
  }
  100% {
  }
}

.pixelated {
  image-rendering: -moz-crisp-edges;
  image-rendering: -o-crisp-edges;
  image-rendering: -webkit-optimize-contrast;
  -ms-interpolation-mode: nearest-neighbor;
}
</style>
