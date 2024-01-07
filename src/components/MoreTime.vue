<script setup>
import Timer from './TimeComponent.vue'
import Runda from './Runda.vue'
import {ref} from 'vue'

let rundQty = 0,
  timeStart;

const fields = ref([]),
  currentTime = ref(0),
  time = ref(0);

function addRound() {
  fields.value.push({ id: rundQty++, time:time, roundTime:null, roundStop: null, currentTime:currentTime })

  if(rundQty === 2) {
    fields._value[rundQty - 2].roundStop = time.value //set first roundStop
    fields._value[rundQty - 2].roundTime = time.value
  }

  if(rundQty > 2) {
    let prevRound = fields._value[rundQty-2]
    prevRound.roundTime = time.value;
    prevRound.roundStop = currentTime.value;
  }
}

function getDifference(time, prev) {
  if(prev) {
    let previous = new Date('2011/02/07 ' + prev),
      finish = new Date('2011/02/07 ' + time.value),
      diff = Math.abs(finish - previous);
    return formatTime(timeUnits(diff))
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

// Helper
function timeUnits( ms ) {
  if ( !Number.isInteger(ms) ) {
    return null
  }
  /**
   * Takes as many whole units from the time pool (ms) as possible
   * @param {int} msUnit - Size of a single unit in milliseconds
   * @return {int} Number of units taken from the time pool
   */
  const allocate = msUnit => {
    const units = Math.trunc(ms / msUnit)
    ms -= units * msUnit
    return units
  }
  // Property order is important here.
  // These arguments are the respective units in ms.
  return {
    // weeks: allocate(604800000), // Uncomment for weeks
    days: allocate(86400000),
    hours: allocate(3600000),
    minutes: allocate(60000),
    seconds: allocate(1000),
    // ms: ms // remainder
  }
}

// End Helpers
function removeRound(id) {
  const index = fields.value.findIndex(f => f.id === id)
  fields.value.splice(index,1)
}

function removeAll() {
  fields.value = []
  rundQty = 0;
}

const handleSendMessage = (timerPushNotice, value1, value2) => {
  if(timerPushNotice === 'removeRounds'){
    removeAll()
  }
  if(timerPushNotice === 'time') {
    time.value = value1
    if(rundQty >= 2) {
      let prev = fields._value[rundQty-2].roundStop;
      time.value = getDifference(time, prev )
    }
    currentTime.value = value1
  }
};

</script>
<template>
  <div class='wrapper'>
    <div>
      <Timer @send-message="handleSendMessage" />
      <button @click="addRound(time)">Dodaj runde</button>
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
          <Runda v-for="field in fields" :id="field.id"
                 :time="field.time"
                 :roundStop="field.roundStop"
                 :timeStart="field.currentTime"
                 :roundTime="field.roundTime"
                 @remove='removeRound(field.id)' />
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