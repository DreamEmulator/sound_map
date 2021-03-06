<template>
  <div id="app">
    <img class="logo" src="./assets/logo.png">
    <div class="dev_notes"
         style="float: right; margin-right: 2em; text-align: left; background-color: aliceblue; padding: 0.25em 1em; max-width: 400px; word-wrap: break-word;">
      <h3>Copy/Paste</h3>
      <ul style="text-align: left">
        <h3>Sounds:</h3>
        <li>static/sounds/bell.mp3</li>
        <li>static/sounds/positive.mp3</li>
      </ul>
      <ul style="text-align: left">
        <h3>Images:</h3>
        <li>static/images/world.jpg</li>
        <li>static/images/the_cove.jpeg</li>
        <li>static/images/fishy.jpeg</li>
      </ul>
      <ul>
        <h3>JSON:</h3>
        <li>
          <input id="json_sound_map" v-bind:value="JSON.stringify(saved_selections)">
          <button v-on:click="copy_json">Copy JSON</button>
        </li>
      </ul>
    </div>
    <div id="sound_map" v-on:click="cancel_select">
      <h1>Geluids Kaart {{name}}</h1>

      <div class="controls">
        <ul>
          <li>
            <button v-on:click="edit_mode = !edit_mode">
              <span v-if="!edit_mode">Edit</span>
              <span v-if="edit_mode">Play</span>
            </button>
          </li>
        </ul>

        <div v-if="edit_mode">
          <ul class="image">
            <li>
              <button v-if="!image_stored" v-on:click="save_image">Choose image</button>
              <button v-if="image_stored" v-on:click="change_image">Change image</button>
              <input v-if="!image_stored" v-model="image" placeholder="Enter URL for image">
            </li>
            <li>
              <button v-on:click="load_json">Load JSON</button>
            </li>
            <li>
              <textarea v-model="json_input"></textarea>
            </li>
          </ul>
          <ul class="selections" v-if="saved_selections.length > 0">
            <li>
              <button v-on:click="show_saved_selections = !show_saved_selections">
                Show selections
              </button>
            </li>
            <li>
              <button v-if="saved_selections.length > 0 && show_saved_selections"
                      v-on:click="edit_selections = !edit_selections">Edit selections
              </button>
            </li>
            <li>
              <button v-if="saved_selections.length > 0" v-on:click="reset_saved_selections">Reset selections</button>
            </li>
          </ul>
        </div>
      </div>

      <div class="map"
           v-on:click="stop_select($event)"
           :class="{edit_mode: edit_mode}">
        <img id="map_image"
             v-on:mousedown="start_select($event)"
             :class="{show_image:image_stored}"
             :src="`${image}`"
             v-on:load="image_loaded"
             v-on:error="image_error"
             v-on:mousemove="select($event)">
        <saved-selection v-for="saved_selection in saved_selections"
                         :key="saved_selection.id"
                         :selection="JSON.parse(saved_selection)" :save_prompt="save_prompt"
                         :show_saved_selections="show_saved_selections"
                         :edit_selections="edit_selections"
                         :edit_mode="edit_mode"
                         v-on:play_sound="play_sound"></saved-selection>
        <new-selection v-if="edit_mode" :selection="selection"
                       :save_prompt="save_prompt" :selection_active="selection_active"
                       v-on:save_selection="save_selection"></new-selection>
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
        edit_mode: true,

        show_controls: false,
        image: "",
        image_shown: false,
        image_stored: false,
        image_dimensions: {
          width: 0,
          height: 0,
        },

        json_input: "",

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
        show_saved_selections: false,
        edit_selections: false,
      }
    },
    methods: {
      e_stop: function (e) {
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
        this.reset_saved_selections();
        this.show_saved_selections = true;
        this.check_image();
      },
      change_image: function () {
        this.show_saved_selections = false;
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

      //JSON:
      copy_json: function () {
        var json = document.getElementById('json_sound_map');
        json.select();
        document.execCommand("copy");
      },
      load_json: function () {
        this.saved_selections = (JSON.parse(this.json_input));
      },

      //Selection methods
      start_select: function () {
        this.e_stop();
        this.set_image_dimensions();
        this.selection.width = this.selection.height = 0;
        this.selecting = false;
        this.selecting = true;
        this.selection_active = true;
        this.selection.from_x = ((event.clientX - this.image_dimensions.left) / this.image_dimensions.width) * 100;
        this.selection.from_y = ((event.clientY - this.image_dimensions.top) / this.image_dimensions.height) * 100;

        this.save_prompt = false;
      },
      select: function () {
        this.e_stop();
        if (this.selecting) {

          this.selection.to_x = ((event.clientX - this.image_dimensions.left) / this.image_dimensions.width) * 100;
          this.selection.to_y = ((event.clientY - this.image_dimensions.top) / this.image_dimensions.height) * 100;

          this.selection.height = this.selection.to_y - this.selection.from_y;
          this.selection.width = this.selection.to_x - this.selection.from_x;
        }
      },
      cancel_select: function () {
        this.e_stop();
        this.selection_active = false;
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
        localStorage.saved_selections = [];
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
    color: #2c3e50;
    padding: 2em;
  }

  #app img.logo {
    position: fixed;
    top: 1em;
    height: 2em;
  }

  #app .controls ul {
    opacity: 1;
    display: inline-block;
    margin: 0.25em;
    list-style: none;
    padding: 1em 2.5em;
    display: inline-block;
    border-radius: 0.5em;
    transition: opacity 0.5s;
    background-color: #e6e6e6;
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
    opacity: 0.5;
    background-color: aliceblue;
    transition: opacity 1s;
    cursor: pointer;
  }

  #app .map.edit_mode {
    cursor: crosshair;
  }

  #app .map img.show_image {
    opacity: 1;
    transform: scale(1);
  }
</style>
