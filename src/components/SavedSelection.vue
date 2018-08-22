<template>
    <div class="saved_selection"
         v-on:click="$emit('play_sound',selection.sound);show_text = true"
         v-on:mouseleave="show_text = false"
         :style="{top: selection.from_y + '%', left: selection.from_x + '%', width: selection.width + '%', height: selection.height+ '%'}">
      <h2 class="selection_text" :class="{show: show_text}">{{selection.text}}</h2>
      <div class="selection_info" :class="{show: show_saved_selections}">
      <div class="edit" v-if="edit_selections">
        <button v-on:click="edit_prompt = !edit_prompt">Edit</button>
      </div>
      <div class="edit_prompt" v-on:mouseleave="edit_prompt = false" v-bind:class="{show:edit_prompt}">
        <input v-model="selection.text" type="text" placeholder="Paste text"/>
        <input v-model="selection.sound" type="text" placeholder="Paste URL to sound"/>
        <button v-on:click="$emit('save_selection')">Save</button>
      </div>
      </div>
    </div>
</template>

<script>
  export default {
    name: "SavedSelection",
    props: ['edit_selections', 'show_saved_selections', 'selection', 'save_prompt'],
    data: function () {
      return {
        show_text: false,
        edit_prompt: false,
      }
    }
  }
</script>

<style scoped>
  #app .map .saved_selection {
    z-index: 1;
    position: absolute;
    display: flex;
    justify-content: center;
    align-content: center;
  }

  #app .map .saved_selection .selection_text {
    position: absolute;
    margin: 0;
    opacity: 0;
    padding: 0.1em 0.2em;
    border-radius: 0.1em;
    align-self: center;
    transition: opacity 1s;
    border: 0.1em solid #ffffffa3;
    background-color: rgba(98, 178, 249, 0.75);
  }

  #app .map .saved_selection .selection_text.show {
    opacity: 1;
  }

  #app .map .saved_selection .selection_info {
    height: 100%;
    width: 100%;
    opacity: 0;
    border-radius: 1.5em;
    transition: opacity 0.5s;
    background-color: rgba(240, 229, 236, 0.5);
    border: 0.21em solid rgba(160, 160, 160, 0.8);
  }
  #app .map .saved_selection .selection_info.show {
    opacity: 1;
  }

  #app .map .saved_selection .edit {
    align-self: flex-end;
  }

  #app .map .saved_selection .edit_prompt {
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

  #app .map .saved_selection .edit_prompt:before {
    content: "";
    position: absolute;
    top: -100%;
    left: 0;
    height: 100%;
    width: 100%;
  }

  #app .map .saved_selection .edit_prompt:after {
    content: "";
    position: absolute;
    top: -0.25em;
    height: 1em;
    width: 1em;
    left: 4em;
    transform: rotate(45deg);
    background-color: #e6e6e6;
  }

  #app .map .saved_selection .edit_prompt.show {
    transform: scale(1);
    opacity: 1;
    transition: opacity 1s, height 0.5s;
  }
</style>
