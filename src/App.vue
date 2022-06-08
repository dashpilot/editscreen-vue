<template>
<Transition>
  <template v-if="curItem">
    <div class="backdrop">
      <div class="editscreen wdgt" spellcheck="false">

        <div class="wdgt-header">
          <h5>Edit</h5>
          <div class="wdgt-close" @click="curItem=false">&times;</div>
        </div>

        <div class="wdgt-body">
          <template v-for="key in Object.keys(fields)">

            <label>{{key.replace('_', ' ')}}</label>

            <template v-if="fields[key] == 'txt'">
              <input type="text" v-model="curItem[key]">
            </template>

            <template v-if="fields[key] == 'rte'">
              <Editor v-model="curItem[key]" :settings="data.ui_settings" />
            </template>

            <template v-if="fields[key] == 'gal'">
              <Gallery v-model:gallery="curItem[key]" :mykey="key" :id="curItem.id" :config="config" :image_width="data.settings.gallery_image_width" :image_title="data.ui_settings.image_title" :image_alt="data.ui_settings.image_alt"
                @update="addToGallery" />
            </template>


          </template>
        </div>

        <div class="wdgt-footer">
          <button @click="save()" class="btn btn-primary btn-save">Save</button>
        </div>
      </div>
    </div>
  </template>
</Transition>
</template>

<script>
import Editor from './widgets/Editor.vue'
import Gallery from './widgets/Gallery.vue'
export default {
  components: {
    Editor,
    Gallery
  },
  data() {
    return {
      data: {},
      curItem: false,
      fields: {},
      config: {
        save_url: "",
        image_delete_url: "",
        image_preview_url: ""
      }
    }
  },
  created: function() {


  },
  mounted() {

    this.data = JSON.parse(localStorage.getItem('data'))
    console.log(this.data)
    this.makeEditable();

  },
  methods: {
    addToGallery(filename, id, mykey) {
      console.log(mykey)
      this.curItem[mykey].push({
        id: "img-" + Math.floor(Math.random() * 999999999),
        filename: filename,
        title: ''
      });
    },
    makeEditable() {

      var myapp = this;

      setTimeout(() => {

        console.log('ok')

        const editables = document.querySelectorAll('.edit');
        const adders = document.querySelectorAll('.add');

        editables.forEach(el => el.addEventListener('click', event => {

          // console.log(event.target.getAttribute("data-el"));
          let section = event.target.closest('.section');
          let section_name = section.getAttribute('id');
          let target = event.target.closest('.edit');
          let id = target.getAttribute('id');
          console.log(id)

          let myfields = section.getAttribute('data-fields');
          var params = new URLSearchParams(myfields);
          this.fields = Object.fromEntries(params.entries())


          if (myapp.data.posts.filter(x => x.id == id)[0]) {
            myapp.curItem = myapp.data.posts.filter(x => x.id == id)[0];
          } else {
            console.log('create new');

            let curItem = {}
            curItem.id = id;
            curItem.section = section_name;
            Object.keys(this.fields).forEach((key) => {
              if (this.fields[key] == 'gal') {
                curItem[key] = [];
              } else {
                curItem[key] = "";
              }
            })

            myapp.data.posts.unshift(curItem);
            myapp.curItem = curItem;
            console.log(curItem)
          }



        }));

        adders.forEach(el => el.addEventListener('click', event => {

          // console.log(event.target.getAttribute("data-el"));
          let section = event.target.closest('.section');
          let section_name = section.getAttribute('id');
          let target = event.target.closest('.add');
          let id = "item-" + Math.floor(Math.random() * 99999999999999999);


          let myfields = section.getAttribute('data-fields');
          var params = new URLSearchParams(myfields);
          this.fields = Object.fromEntries(params.entries())

          let curItem = {}
          curItem.id = id;
          curItem.section = section_name;
          Object.keys(this.fields).forEach((key) => {
            if (this.fields[key] == 'gal') {
              curItem[key] = [];
            } else {
              curItem[key] = "";
            }
          })

          myapp.data.posts.unshift(curItem);
          myapp.curItem = curItem;

        }));


      }, 500);

    },
    save() {
      localStorage.setItem('data', JSON.stringify(this.data))
      this.curItem = false;
      window.render();

      this.makeEditable();

    }
  }
}
</script>

<style scoped>
* {
  box-sizing: border-box;
}

.backdrop {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
}

.editscreen {
  width: 940px;
  height: 80vh;
  max-height: 700px;
  background-color: white;
  margin: 0 auto;
  margin-top: 5%;
  margin-bottom: 5%;
  border-radius: 8px;
  padding: 20px;
  padding-right: 0 !important;
  position: relative;
  overflow: hidden;
}

.wdgt-header {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  padding: 10px 20px 0px;
  background-color: white;
}

.wdgt-header h5 {
  float: left;
  margin-top: 10px;
}

.wdgt-close {
  float: right;
  font-size: 25px;
  color: #777;
}

.wdgt-close:hover {
  color: black;
  cursor: pointer;
}

.wdgt-body {
  height: 100%;
  padding-right: 20px;
  margin-top: 25px;
  padding-top: 5px;
  padding-bottom: 90px;
  overflow: hidden;
  overflow-y: auto;
}

.wdgt-footer {
  padding: 15px 20px;
  border-top: 1px solid #DDD;
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  background-color: #F8F8F8;
}

.btn-save {
  width: 100px;
}

input,
textarea {
  width: 100%;
}

textarea {
  min-height: 150px;
  height: 150px;
}

label {
  display: block;
  font-size: 14px;
  text-transform: uppercase;
  letter-spacing: 0.02em;
  padding-top: 15px;
  padding-bottom: 3px;
}

input,
textarea {
  display: block;
  width: 100%;
  height: calc(1.5em + .75rem + 2px);
  padding: .375rem .75rem;
  font-size: 1rem;
  font-weight: 400;
  line-height: 1.5;
  color: #495057;
  background-color: #fff;
  background-clip: padding-box;
  border: 1px solid #ced4da;
  border-radius: .25rem;
  transition: border-color .15s ease-in-out, box-shadow .15s ease-in-out;
  resize: none;
}
</style>

<style>
#app,
textarea {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}

.edit {
  border: 1px dashed transparent;
  display: block;
  min-height: 20px;
}

.edit:hover {
  border: 1px dashed #CCC;
  cursor: pointer;
}

.v-enter-active,
.v-leave-active {
  transition: opacity 0.5s ease;
}

.v-enter-from,
.v-leave-to {
  opacity: 0;
}

.wdgt .btn {
  display: inline-block;
  font-weight: 400;
  color: #212529;
  text-align: center;
  vertical-align: middle;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
  background-color: transparent;
  border: 1px solid transparent;
  border-top-color: transparent;
  border-right-color: transparent;
  border-bottom-color: transparent;
  border-left-color: transparent;
  padding: .375rem .75rem;
  font-size: 1rem;
  line-height: 1.5;
  border-radius: .25rem;
  transition: color .15s ease-in-out, background-color .15s ease-in-out, border-color .15s ease-in-out, box-shadow .15s ease-in-out;
}

[type="button"],
[type="reset"],
[type="submit"],
button {
  -webkit-appearance: button;
}

.wdgt button,
.wdgt select {
  text-transform: none;
}

.wdgt button,
.wdgt input {
  overflow: visible;
}

.wdgt button,
.wdgt input,
.wdgt optgroup,
.wdgt select,
.wdgt textarea {
  margin: 0;
  font-family: inherit;
  font-size: inherit;
  line-height: inherit;
}

.wdgt a.btn {
  display: inline-block;
  text-decoration: none;
}

.wdgt .btn-primary {
  color: #fff;
  background-color: #007bff;
  border-color: #007bff;
}

.wdgt .btn-primary:hover {
  color: #fff;
  background-color: #0069d9;
  border-color: #0062cc;
}

.wdgt .btn-outline-secondary {
  color: #777;
  border: 1px solid #CCC;
}

.wdgt .btn-outline-secondary:hover {
  color: black;
  border: 1px solid #AAA;
}

.wdgt .btn-outline-dark {
  color: black;
  border: 1px solid black;
}


.rte {
  height: 200px;
}

.ProseMirror {
  min-height: 190px !important;
  border: 1px solid #DDD;
  padding: 15px;
  background-color: white;
  border-radius: 6px;
  outline: none;
  border-top-left-radius: 0;
  border-top-right-radius: 0;
}
</style>
