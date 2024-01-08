<script setup>
import { ref } from 'vue'
import Swal from "sweetalert2";
const emit = defineEmits(['remove'])
const props = defineProps(['item'])

let displayTitle = ref('Zmien nazwe rundy')
function remove() {
  emit('remove', props.item.id)
}

async function rename() {
  let { value: title } = await Swal.fire({
    title: 'Enter the new title',
    input: 'text',
    inputValue: displayTitle.value,
    showCancelButton: true
  })
  if (title !== undefined) displayTitle.value = title
}

</script>

<template>
  <tr>
    <td>{{ props.item.id + 1}}</td>
    <td><div v-if='displayTitle'><span @click='rename'>{{ displayTitle }}</span></div></td>
    <td>{{ props.item.roundTime }}</td>
    <td>{{ props.item.roundStop }}</td>
    <td><button type='button' @click='remove'>Remove</button></td>
  </tr>

</template>

<style scoped>
  tr  {
    margin:10px 0;
    display: table;
  }
</style>