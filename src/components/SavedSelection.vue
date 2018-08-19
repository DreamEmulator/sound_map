<template>
  <div class="saved_selection" :class="{show: show_saved_selections}" v-on:click="$emit('play_sound',selection.sound)"
       v-bind:style="{top: selection.from_y + '%', left: selection.from_x + '%', width: selection.width + '%', height: selection.height+ '%'}">
    <div class="edit" v-if="edit_selections">
    <button v-on:click="edit_prompt = !edit_prompt">Edit</button>
    </div>
    <div class="edit_prompt" v-on:mouseleave="edit_prompt = false" v-bind:class="{show:edit_prompt}">
      <input v-model="selection.text" type="text" placeholder="Paste text"/>
      <input v-model="selection.sound" type="text" placeholder="Paste URL to sound"/>
      <button v-on:click="$emit('save_selection')">Save</button>
    </div>
  </div>
</template>

<script>
    export default {
        name: "SavedSelection",
        props: ['edit_selections','show_saved_selections','selection','save_prompt'],
        data: function () {
          return {
            edit_prompt: false,
          }
        }
    }
</script>

<style scoped>
  #app .map .saved_selection {
    z-index: 1;
    position: absolute;
    border-radius: 1.5em;
    background-color: rgba(240, 229, 236, 0.5);
    border: 0.21em solid rgba(160, 160, 160, 0.8);
    display: flex;
    justify-content: center;
    align-content: center;
    opacity: 0;
    transition: opacity 0.5s;
  }

  #app .map .saved_selection.show {
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
  #app .map .saved_selection .edit_prompt.show{
    transform: scale(1);
    opacity: 1;
    transition: opacity 1s, height 0.5s;
  }
</style>
