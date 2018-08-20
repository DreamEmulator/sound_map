<template>
  <div id="app">
    <img class="logo" src="./assets/logo.png">
    <ul style="text-align: left">
      <h3>Sounds:</h3>
      <li>static/sounds/bell.mp3 </li>
      <li>static/sounds/positive.mp3</li>
    </ul>
    <ul style="text-align: left">
      <h3>Images:</h3>
      <li>static/images/the_cove.jpeg</li>
      <li>static/images/fishy.jpeg</li>
    </ul>
    <div id="sound_map">
      <h1>Sound Map {{name}}</h1>
      <div class="image_control">
        <button v-if="!image_stored" v-on:click="save_image">Save image</button>
        <button v-if="image_stored" v-on:click="change_image">Change image</button>
        <input v-if="!image_stored" v-model="image" placeholder="Enter URL for image">
      </div>
      <div class="selection_control">
        <button v-if="saved_selections.length > 0" v-on:click="show_saved_selections = !show_saved_selections">Show selections</button>
        <button v-if="saved_selections.length > 0" v-on:click="edit_selections = !edit_selections">Edit selections
        </button>
        <button v-if="saved_selections.length > 0" v-on:click="reset_saved_selections">Reset selections</button>
      </div>
        <div class="map"  v-on:mouseup="stop_select">
          <img id="map_image"v-on:mousedown="start_select" :class="{show_image:image_stored}" :src="`${image}`" v-on:load="image_loaded" v-on:error="image_error" v-on:mousemove="select">
          <saved-selection v-for="saved_selection in saved_selections" :key="saved_selection.id"
                           :selection="JSON.parse(saved_selection)" :save_prompt="save_prompt" :show_saved_selections="show_saved_selections"
                           :edit_selections="edit_selections" v-on:play_sound="play_sound"></saved-selection>
          <new-selection :selection="selection" :save_prompt="save_prompt" :selection_active="selection_active" v-on:save_selection="save_selection"></new-selection>
        </div>
    </div>
    <audio ref="play_sound" autoplay v-bind:src="`${audio}`" type="audio/mp3">
    </audio>
  </div>
</template>

<script>
  import NewSelection from './components/NewSelection'
  import SavedSelection from './components/SavedSelection'

  export default {
    name: 'App',
    components: {
      NewSelection,
      SavedSelection,
    },
    data: function () {
      return {
        name: "",

        show_controls: false,
        image: "",
        image_shown: false,
        image_stored: false,
        image_dimensions: {
          width: 0,
          height: 0,
        },

        audio: "",

        selecting: false,
        selection: {
          from_x: 0,
          from_y: 0,
          to_x: 0,
          to_y: 0,
          width: 0,
          height: 0,
          text: "",
          sound: "",
        },
        selection_active: false,
        save_prompt: false,
        saved_selections: [],
        show_saved_selections: true,
        edit_selections: false,
      }
    },
    methods: {
      e_stop: function () {
        event.preventDefault();
        event.stopPropagation();
      },

      //Image CRUD
      image_loaded: function () {
        this.image_shown = true;
      },
      image_error: function () {
        this.image.shown = false;
      },
      save_image: function () {
        localStorage.SoundMapImage = this.image;
        this.check_image();
      },
      change_image: function () {
        this.image_stored = false;
        this.save_prompt = false;
      },
      check_image: function () {
        this.image_stored = localStorage.SoundMapImage.length > 0;
      },
      set_image_dimensions: function () {
        let image_dimensions = document.getElementById('map_image').getBoundingClientRect();
        this.image_dimensions.height = image_dimensions.height;
        this.image_dimensions.width = image_dimensions.width;
        this.image_dimensions.top = image_dimensions.top;
        this.image_dimensions.left = image_dimensions.left;
      },

      //Selection methods
      start_select: function () {
        this.e_stop();
        this.set_image_dimensions();
        this.selection.width = this.selection.height = 0;
        this.selecting = false;
        this.selecting = true;
        this.selection_active = true;
        this.selection.from_x = ((event.clientX - this.image_dimensions.left) / this.image_dimensions.width)*100;
        this.selection.from_y = ((event.clientY - this.image_dimensions.top) / this.image_dimensions.height)*100;

        this.save_prompt = false;
      },
      select: function () {
        this.e_stop();
        if (this.selecting) {

          this.selection.to_x = ((event.clientX - this.image_dimensions.left) / this.image_dimensions.width)*100;
          this.selection.to_y = ((event.clientY - this.image_dimensions.top) / this.image_dimensions.height)*100;

          this.selection.height = this.selection.to_y - this.selection.from_y;
          this.selection.width = this.selection.to_x - this.selection.from_x;
        }
      },
      cancel_select: function () {
        this.e_stop();
        this.selecting = false;
        this.save_prompt = false;
      },
      stop_select: function () {
        this.e_stop();
        this.selecting = false;
        this.save_prompt = true;
      },

      //Selection CRUD
      save_selection: function () {
        this.e_stop();
        this.selection_active = false;
        this.save_prompt = false;
        this.saved_selections.push(JSON.stringify(this.selection));
        localStorage.saved_selections = JSON.stringify(this.saved_selections);
      },
      load_saved_selections: function () {
        if (localStorage.saved_selections) {
          this.saved_selections = JSON.parse(localStorage.saved_selections);
        }
      },
      reset_saved_selections: function () {
        this.saved_selections = [];
        this.selecting = false;
      },

      //Sounds
      play_sound: function (sound) {
        this.audio = sound;
        this.$refs.play_sound.play();
      }
    },
    watch: {
      image_stored: function () {
        this.image_stored ? this.image = localStorage.SoundMapImage : false;
      },
    },
    mounted: function () {
      console.log("Vue is ready");
      this.check_image();
      this.load_saved_selections();
    }
  }
</script>

<style>
  * {
    box-sizing: border-box;
  }

  body {
    margin: 0;
  }

  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
  }

  #app img.logo {
    position: fixed;
    top: 1em;
    left: 6em;
    height: 2em;
  }

  #app .controls {
    display: inline-block;
    margin-top: 2.5em;
    border-radius: 0.5em;
    left: 1em;
    padding: 1em 3.5em;
    background-color: #e6e6e6;
    transition: opacity 0.5s;
    opacity: 1;
  }

  #app .controls.show {
    opacity: 1;
  }

  #app .buffer_frame {
    padding: 1em;
    display: inline-block;
    background-color: aquamarine;
  }

  #app .map {
    position: relative;
    display: inline-block;
    margin: 1em;
  }

  #app .map img {
    max-width: 80vw;
    max-height: 80vh;
    opacity: 0;
    background-color: aliceblue;
    transition: opacity 1s, transform 0.5s;
    transform: scale(0);
    cursor: pointer;
  }

  #app .map img.show_image {
    opacity: 1;
    transform: scale(1);
  }
</style>
