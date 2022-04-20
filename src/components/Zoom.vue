<template>
  <div>
    <p v-if="error">{{ error }}</p>
    <select class="dd" v-model="languages">
      <option value="en-US">English</option>
      <option value="ru">Russian</option>
    </select>
    <button v-if="!recRunning" @click="startSpeechRecognition()">Start</button>
    <button v-if="recRunning" @click="endSpeechRecognition()">End</button>
    <br />
    <br />
    <textarea
      :class="['area', { active: speaking }]"
      :value="runtimeTranscription"
    ></textarea>
    <br />
    <div v-for="item in sentences.slice().reverse()" :key="item.id">
      {{ item.date.getHours() }}:{{ item.date.getMinutes() }}:{{
        item.date.getSeconds()
      }}
      - {{ item.text }}
    </div>
  </div>
</template>

<script>
export default {
  name: "SpeechRec",
  data: function () {
    return {
      languages: "en-US",
      error: false,
      speaking: false,
      recRunning: false,
      runtimeTranscription: "",
      sentences: [],
      recognition: null,
    };
  },
  methods: {
    endSpeechRecognition() {
      this.recognition.stop();
      this.recRunning = false;
    },

    startSpeechRecognition() {
      let SpeechRecognition =
        window.SpeechRecognition || window.webkitSpeechRecognition;
      this.recognition = SpeechRecognition ? new SpeechRecognition() : false;

      if (!this.recognition) {
        this.error =
          "Speech recognition is not available on this browser. Please use Chrome or Firefox";
        this.recRunning = false;
      }

      this.recRunning = true;
      this.recognition.lang = this.languages;
      this.recognition.interimResults = true;

      this.recognition.addEventListener("speechstart", () => {
        console.log("speechstart");
        this.speaking = true;
      });

      this.recognition.addEventListener("speechend", () => {
        console.log("speechend");
        this.speaking = false;
        this.recRunning = false;

        this.startSpeechRecognition();
      });

      this.recognition.addEventListener("result", (event) => {
        const text = Array.from(event.results)
          .map((result) => result[0])
          .map((result) => result.transcript)
          .join("");
        this.runtimeTranscription = text;
      });

      this.recognition.addEventListener("end", () => {
        if (this.runtimeTranscription) {
          this.sentences.push({
            date: new Date(),
            text: this.capitalizeFirstLetter(this.runtimeTranscription),
            speaker: "",
          });
          this.runtimeTranscription = "";
          this.recRunning = false;
          this.startSpeechRecognition();
        }
      });

      this.recognition.addEventListener("start", () => {
        console.log("start");
        this.recRunning = true;
      });

      this.recognition.start();
    },
    capitalizeFirstLetter(string) {
      return string.charAt(0).toUpperCase() + string.slice(1);
    },
  },
};
</script>

<style scoped lang="scss">
.area {
  width: 400px;
  height: 150px;
  &.active {
    border-color: red;
  }
}
.dd {
  width: 120px;
  margin-right: 20px;
}
button {
  margin-right: 20px;
}
</style>
