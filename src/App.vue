<template>
  <div class="container__app">
    <div v-for="(phrase, index) in fiveMostPopularPhrases" v-if="fiveMostPopularPhrases.length > 0" :key="index">
      <div class="row__phrase">
        <div class="item__number" :class="{big: index===0}" :style="{color: getColor(index)}">
          {{ phrase[1] }}
        </div>
        <div class="item__phrase" :class="{big: index===0}">
          {{ phrase[0] }}
        </div>
      </div>
    </div>
    <div v-else>
      Waiting for messages...
    </div>
  </div>
</template>

<script>
import tmi from 'tmi.js';
import config from '../config.js';

export default {
  name: 'App',
  data() {
    return {
      freq: {
        1: {},
        2: {},
        3: {}
      }
    }
  },
  created() {
    const BOT_USERNAME = config.BOT_USERNAME;
    const BOT_TOKEN = config.BOT_TOKEN;
    const CHANNEL_NAME = config.CHANNEL_NAME;

    const opts = {
      identity: {
        username: BOT_USERNAME,
        password: BOT_TOKEN
      },
      channels: [
        CHANNEL_NAME
      ],
      connection: {
        reconnect: true
      },
      options: {
        debug: true
      }
    };

    const client = new tmi.client(opts);

    client.on('message', this.onMessageHandler);
    client.on('connected', this.onConnectedHandler);

    client.connect();
  },
  computed: {
    fiveMostPopularPhrases() {
      const phrases = []
      for (const key in this.freq[1]) {
        phrases.push([key, this.freq[1][key]])
      }
      for (const key in this.freq[2]) {
        phrases.push([key, this.freq[2][key]])
      }
      for (const key in this.freq[3]) {
        phrases.push([key, this.freq[3][key]])
      }

      phrases.sort((a, b) => {
        return b[1] - a[1]
      })

      // for 2 word phrases in the top 10, if it is also contained in a 3 word phrase in the top 10, subtract the 3 word phrase's frequency from the 2 word phrase's frequency
      for (let i = 0; i < Math.min(phrases.length, 5); i++) {
        if (phrases[i][0].split(' ').length === 2) {
          for (let j = 0; j < Math.min(phrases.length, 5); j++) {
            if (phrases[j][0].split(' ').length === 3) {
              if (phrases[j][0].includes(phrases[i][0])) {
                phrases[i][1] -= phrases[j][1]
              }
            }
          }
        }
      }

      // for 1 word phrases in the top 5, if it is also contained in a 2 word phrase in the top 5, subtract the 2 word phrase's frequency from the 1 word phrase's frequency
      for (let i = 0; i < Math.min(phrases.length, 5); i++) {
        if (phrases[i][0].split(' ').length === 1) {
          for (let j = 0; j < Math.min(phrases.length, 5); j++) {
            if (phrases[j][0].split(' ').length === 2) {
              if (phrases[j][0].includes(phrases[i][0])) {
                phrases[i][1] -= phrases[j][1]
              }
            }
          }
        }
      }

      // if a phrase has a frequency of 0, remove it
      for (let i = 0; i < phrases.length; i++) {
        if (phrases[i][1] <= 0) {
          phrases.splice(i, 1)
          i--
        }
      }

      phrases.sort((a, b) => {
        // if the frequencies are the same, sort by length of phrase, prioritizing longer phrases
        if (a[1] === b[1]) {
          return b[0].split(' ').length - a[0].split(' ').length
        }
        return b[1] - a[1]
      })

      return phrases.slice(0, 5)
    }
  },
  methods: {
    getColor(index) {
      switch (index){
        case 0:
          return '#edfd0e'
        case 1:
          return '#fe0fe7'
        case 2:
          return '#16ffe2'
        default:
          return '#ffffff'
      }
    },
    onMessageHandler(target, context, msg, self) {
      if (self) {
        return;
      }

      // remove any non-alphanumeric characters
      let trimmedMsg = msg.trim()
      trimmedMsg = trimmedMsg.replace(/[^a-zA-Z0-9 ]/g, '')
      const words = trimmedMsg.split(' ')

      // if there are any blank spaces in words, remove them
      for (let i = 0; i < words.length; i++) {
        if (words[i].trim() === '') {
          words.splice(i, 1)
        }
      }

      // support 1, 2, and 3 word phrases
      for (let i = 1; i <= 3; i++) {
        // if there are too many words in the frequency table, remove all that appear the least
        if (Object.keys(this.freq[i]).length > 1000) {
          let min = Infinity
          let minKeys = []
          for (const key in this.freq[i]) {
            if (this.freq[i][key] < min) {
              min = this.freq[i][key]
              minKeys = [key]
            } else if (this.freq[i][key] === min) {
              minKeys.push(key)
            }
          }
          for (const key of minKeys) {
            delete this.freq[i][key]
          }
        }

        for (let j = 0; j < words.length - i + 1; j++) {
          const phrase = words.slice(j, j + i).join(' ')
          if (this.freq[i][phrase]) {
            this.freq[i][phrase] += 1
          } else {
            this.freq[i][phrase] = 1
          }
        }
      }

      const mostFrequentOneWordPhrase = this.findMostFrequentPhrase(1)
      const mostFrequentTwoWordPhrase = this.findMostFrequentPhrase(2)
      const mostFrequentThreeWordPhrase = this.findMostFrequentPhrase(3)
    },
    onConnectedHandler(addr, port) {
      console.log(`* Connected to ${addr}:${port}`);
    },
    findMostFrequentPhrase(n) {
      let max = 0
      let maxPhrase = ''
      for (const key in this.freq[n]) {
        if (this.freq[n][key] > max) {
          max = this.freq[n][key]
          maxPhrase = key
        }
      }
      return maxPhrase
    }
  }
}
</script>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}

.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}

.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}

.row__phrase {
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
  white-space: nowrap;
}

.item__number {
  font-size: 2em;
  font-weight: bold;
  margin-right: 0.5em;
  color: #16ffe2;
}

.item__phrase {
  font-size: 1.5em;
}

.item__phrase.big {
  font-size: 2.5em;
}

.item__number.big {
  font-size: 3.5em;
  color: #edfd0e;
}


.container__app {
  display: flex;
  flex-direction: column;
  justify-content: start;
  align-items: start;
  white-space: nowrap;
  padding: 1.5em;
  width: 100vw;
  overflow: hidden;
  height: 100vh;
  max-height: 100vh;
}
</style>
