<template>
  <div class="selection" :class="{active: selection_active}"
       :style="{top: selection.from_y + '%', left: selection.from_x + '%', width: selection.width + '%', height: selection.height+ '%'}">
    <div class="save_prompt" v-bind:class="{show:save_prompt}">
      <input v-model="selection.text" type="text" placeholder="Paste text"/>
      <input v-model="selection.sound" type="text" placeholder="Paste URL to sound"/>
      <button v-on:click="$emit('save_selection')">Save</button>
    </div>
  </div>
</template>

<script>
    export default {
        name: "NewSelection",
        props: ['selection','selection_active','save_prompt','selecting'],
    }
</script>

<style scoped>
  #app .map .selection  {
    z-index: 1;
    display: none;
    position: absolute;
    border-radius: 1.5em;
    background-color: rgba(240, 229, 236, 0.5);
    border: 0.21em solid rgba(219, 219, 226, 0.81);
    justify-content: center;
    align-content: center;
  }

  #app .map .selection.active {
    display: flex;
  }

  #app .map .selection .save_prompt {
    position: absolute;
    display: flex;
    justify-content: space-between;
    align-content: center;
    flex-direction: column;
    width: 9em;
    background-color: #e6e6e6;
    bottom: -5em;
    padding: 0.5em;
    transform: scale(0);
    opacity: 0;
  }
  #app .map .selection .save_prompt:after {
    content: "";
    z-index: -1;
    position: absolute;
    top: -0.25em;
    height: 1em;
    width: 1em;
    left: 4em;
    transform: rotate(45deg);
    background-color: #e6e6e6;
  }
  #app .map .selection .save_prompt.show{
    transform: scale(1);
    opacity: 1;
    transition: opacity 1s, height 0.5s;
  }
</style>
