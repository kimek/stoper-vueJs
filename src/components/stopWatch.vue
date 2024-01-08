<script setup>
import Timer from './TimeComponent.vue'
import Round from './Round.vue'
import helper from './timeHelper.js'
import Swal from "sweetalert2"
import { ref } from 'vue'

let roundsQuantity = 0, removedId = 0

const fields = ref([]),
  currentTime = ref(0),
  time = ref(0);

function addRound() {
  fields.value.push({ id: roundsQuantity++, roundTime:null, roundStop: null })

  let prevRound = fields._value[roundsQuantity-2]

  if(roundsQuantity === 2 && prevRound) { //set first roundStop
    prevRound.roundTime = currentTime.value
    prevRound.roundStop = currentTime.value
  }

  if(roundsQuantity > 2) {
    if(prevRound) {
      prevRound.roundTime = time.value;
      prevRound.roundStop = currentTime.value;
    }
  }
}

function getDifference(time, prev) {
  if(prev) {
    let previous = new Date('2011/02/07 ' + prev),
      finish = new Date('2011/02/07 ' + time.value),
      diff = Math.abs(finish - previous);
    return formatTime(helper.methods.timeUnits(diff))
  }
  return false;
}

function formatTime(diffObject) {
  for(const par in diffObject) {
    if(diffObject[par] < 10) {
      diffObject[par] = '0' + diffObject[par];
    }
  }
  return diffObject['hours'] + ':' + diffObject['minutes'] + ':' + diffObject['seconds'];
}

function removeRound(id) {
  //TODO implement removing rounds
    Swal.fire('To implement');
    return
  // const index = fields.value.findIndex(f => f.id === id)
  // fields.value.splice(index,1)
  // removedId++
}

function removeAll() {
  fields.value = []
  roundsQuantity = 0;
}

const handleSendMessage = (timerPushNotice, value1, value2) => {
  if(timerPushNotice === 'removeRounds'){
    removeAll()
  }
  //TODO Dont use fields._value as this is terrible
  if(timerPushNotice === 'time') {
    time.value = value1

    if(roundsQuantity >= 2) {
      let prevRound = fields._value[roundsQuantity-2]
      if(prevRound) {
        time.value = getDifference(time, prevRound.roundStop )
      }

    }
    currentTime.value = value1

    if(roundsQuantity > 0) {

      let prevRound = fields._value[roundsQuantity-1]

      if(!prevRound) {
        prevRound = fields._value[roundsQuantity-removedId-1]
        console.log('BUG: TODO - Fix Handling IDs in more reactive way');
      }

      prevRound.roundStop = value1
      prevRound.roundTime = time.value

    }
  }
};

</script>
<template>
  <div class='wrapper'>
    <div>
      <Timer @send-message="handleSendMessage" />
      <button @click="addRound()">Dodaj runde</button>
      <table v-if='fields.length'>
        <thead>
          <tr>
            <td></td>
            <td>Runda</td>
            <td>Czas</td>
            <td>Łączny czas</td>
            <td>Akcja</td>
          </tr>
        </thead>
        <tbody>
          <Round v-for="item in fields" :key="item.id"
                 :item="item"
                 @remove='removeRound(item.id)' />
        </tbody>
      </table>
    </div>
  </div>
</template>

<style>
button {
  background: #2c2c2c;
  color: #fff;
  font-size: 0.8em;
  padding: 9px 16px;
  margin: 5px;
  border: 1px solid #302d2d;
  border-radius: 3px;
  letter-spacing: 1px;
  box-shadow: 0 0 2px 2px #000;
  font-weight: bold;
}
table button {
  padding: 11px 0;
  width:100%;
  margin: 0;
}
td  {
  width: 23%;
}
td:nth-child(1)  {
  width: 8%;
}
table {
  width:100%;
  display: flex;
  flex-direction: column-reverse;
  margin: 2em 0;
}

thead {
  order:1;
  display: table;
}
tbody {
  display: flex;
  flex-direction: column-reverse;
}

</style>