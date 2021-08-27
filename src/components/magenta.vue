<script></script>
<template>
  <div>
    <textarea
      v-model="message"
      name=""
      rows="4"
      cols="40"
    ></textarea>
    <p></p>
    <button @click="musicGenerate()">generate</button>
    <p>{{music}}</p>
    <template v-if="ismusic">
      <button @click="playRef()">musicStart</button>
      <button @click="playRNN()">runRNN</button>
    </template>
  </div>
</template>
<script>
export default {
  data() {
    return {
      message: "",
      music: {
        notes: [],
        tempos: [
          {
            times: 0,
            qpm: 120
          }
        ],
        totalTime: 0
      },
      music_rnn: Object,
      rnnPlayer: Object,
      refPlayer: Object,
      ismusic: false
    };
  },
  methods: {
    musicGenerate() {
      const str = this.message;
      if (str) {
        const chars = str.split("");
        this.music.notes = chars.map((char, index) => {
          switch (char) {
            case "a":
              return { pitch: 60, startTime: index, endTime: index + 1 };
            case "b":
              return { pitch: 62, startTime: index, endTime: index + 1 };
            case "c":
              return { pitch: 64, startTime: index, endTime: index + 1 };
            case "d":
              return { pitch: 65, startTime: index, endTime: index + 1 };
            case "e":
              return { pitch: 57, startTime: index, endTime: index + 1 };
            default:
              return { pitch: 60, startTime: index, endTime: index + 1 };
          }
        });
        this.music.totalTime = str.length;
        if (str.length) {
          this.ismusic = true;
        }
      }
    },
    playRNN() {
      const RNN_STEPS = 50;
      const RNN_TEMPERATURE = 1.5;
      // The model expects a quantized sequence, and ours was unquantized:
      const qns = mm.sequences.quantizeNoteSequence(this.music, 4);
      this.music_rnn
        .continueSequence(qns, RNN_STEPS, RNN_TEMPERATURE)
        .then(sample => this.rnnPlayer.start(sample));
    },
    playRef() {
      this.refPlayer.start(this.music);
    }
  },
  mounted: function() {
    console.log("mounted");
    // Initialize model
    this.refPlayer = new mm.Player();
    this.music_rnn = new mm.MusicRNN(
      "https://storage.googleapis.com/magentadata/js/checkpoints/music_rnn/basic_rnn"
    );
    this.music_rnn.initialize();
    this.rnnPlayer = new mm.Player();
  }
};
</script>
