<template>
  <div id="app">
    <div class="subheading">Elections</div>
    <div class="cont" v-if="!voted">
      <div class="heading">VOTING</div>
      <div class="cards">
        <candidate v-for="candidate in candidateList"
          v-bind:candidate="candidate" v-bind:key="candidate.id">
        </candidate>
      </div>
      <div class="copyright">
        <a href="https://unsplash.com/s/photos/portrait" target="_blank" rel="noreferrer noopener">
          Images taken from unsplash.com
        </a>
      </div>
    </div>
    <div class="cont" v-if="voted">
      <div class="heading">RESULTS</div>
      <div class="chart-container">
        <canvas id="canvas"></canvas>
      </div>
    </div>
  </div>
</template>

<script>
import candidate from './components/candidate.vue'
import { Chart } from 'chart.js';
import { Realtime } from 'ably';
let ably = new Realtime('DdYVnQ.eruigg:PPG34TX7Pm24EdOK');
let channel = ably.channels.get('vote-channel');

export default {
  name: 'App',
  components: { candidate },
  data: function() {
    return {
        candidateList: [
          { votes: 0, img: require('./assets/johndoe.jpg'), name: 'John Doe', desc: 'Lorem ipsum dolor sit amet, consectetur adipiscing elit. Donec id tristique ipsum.'},
          { votes: 0, img: require('./assets/janesmith.jpg'), name: 'Jane Smith', desc: 'Lorem ipsum dolor sit amet, consectetur adipiscing elit. Donec id tristique ipsum.'},
          { votes: 0, img: require('./assets/richardroe.jpg'), name: 'Richard Roe', desc: 'Lorem ipsum dolor sit amet, consectetur adipiscing elit. Donec id tristique ipsum.'}
        ],
        voted: false
    }
  },
  methods: {
    markVoted: function () {
      if (!this.voted) this.voted = true;
    },
    addVote: function (candidateName) {
        const index = this.candidateList.map(e => e.name).indexOf(candidateName);
        if (index !== -1) {
          this.candidateList[index].votes++;
        }
    }
  },
  mounted: function() {
    channel.history((err, results) => {
      for (let item of results.items) this.addVote(item.data);
    });
  },
  created: function() {
      channel.subscribe('vote', (message) => {
        this.addVote(message.data);
        this.chart = new Chart('canvas', {
          type: 'bar',
          data: {
            labels: this.candidateList.map(({ name }) => name),
            datasets: [{
              data: this.candidateList.map(({ votes }) => votes),
              backgroundColor: [ '#1B998B', '#0496FF', '#FFBC42' ],
            }]
          },
          options: {
            tooltips: { enabled: false },
            responsive: true,
            responsiveAnimationDuration: 1000,
            animation: { duration: 2000 },
            maintainAspectRatio: false,
            scales: { yAxes: [{ ticks: { precision: 0, beginAtZero: true } }] },
            layout: { padding: 25 },
            legend: { display: false }
          }
        });
      });
  }
}
</script>

<style>
html {
  height: 100%;
}
body {
  min-height: 100%;
}
body {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif, "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol";
  color: #333;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  margin: 0;
  text-align: center;
}
#app {
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
}
.cont {
  height: calc(100% - 40px);
}
.subheading {
  font-weight: 300;
  font-size: 20px;
  line-height: 40px;
}
.heading {
  font-weight: 700;
  font-size: 50px;
  line-height: 70px;
}
.cards {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  margin-top: 20px;
}
.copyright {
  color: grey;
  margin: 10px 0;
}
.chart-container {
  height: calc(100% - 70px);
}
</style>
