<template>
  <div class="card">
    <div class="img"
      v-bind:style="{ 'background-image': 'url(' + candidate.img + ')' }">
    </div>
    <div class="card-cont">
      <div class="name">{{ candidate.name }}</div>
      <div class="desc">{{ candidate.desc }}</div>
      <div class="button"
        v-on:click="vote(candidate.name)">Vote</div>
    </div>
  </div>
</template>

<script>
import { Realtime } from 'ably';
let ably = new Realtime('DdYVnQ.eruigg:PPG34TX7Pm24EdOK');
let channel = ably.channels.get('vote-channel');

export default {
  name: 'candidate',
  props: ['candidate'],
  methods: {
    vote: function (name) {
      channel.publish('vote', name);
      this.$parent.markVoted();
    }
  }
}
</script>

<style scoped>
.card {
  border: 1px solid #eee;
  background-color: #fafafa;
  margin: 0 8px 16px;
  display: flex;
  flex-basis: 450px;
  flex-grow: 9999;
  transition: all 0.2s ease-in-out;
}
.card-cont {
  width: 60%;
  padding: 20px;
  box-sizing: border-box;
}
.card .name {
  font-size: 30px;
  margin-bottom: 10px;
}
.card .desc {
  font-size: 15px;
  margin-bottom: 10px;
}
.card .img {
  width: 40%;
  background-position: center;
  background-size: cover;
}
.card .button {
  background-color: #333;
  border: 1px solid #333;
  color: white;
  padding: 10px;
  cursor: pointer;
}
</style>
