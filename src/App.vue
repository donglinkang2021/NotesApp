<script setup>
import { marked } from 'marked'
import { debounce } from 'lodash-es'
import { ref, computed } from 'vue'

const showNoteModal = ref(false);
const reEdit = ref(false);
const newNote = ref("");
const curNoteID = ref(null);
const renderMarkdown = computed(() => marked(newNote.value))
const update = debounce((e) => {
  newNote.value = e.target.value
}, 100)

const notes = ref([]);
const errorMessage = ref("");
const draggedIndex = ref(null);

function getRandomColor() {
  return "hsl(" + Math.random() * 360 + ", 100%, 75%)";
}

const addNote = () => {
  if (newNote.value.length < 10) {
    errorMessage.value = "Note must be at least 10 characters long";
    return;
  }
  curNoteID.value = Math.floor(Math.random() * 1000_000);
  notes.value.push({ 
    id: curNoteID.value,
    text: newNote.value, 
    date: new Date().toLocaleString(), 
    backgroundColor: getRandomColor()
  });
  showNoteModal.value = false;
  newNote.value = "";
  errorMessage.value = "";
  isFullscreen.value = false;
}

const saveNote = () => {
  if (newNote.value.length < 10) {
    errorMessage.value = "Note must be at least 10 characters long";
    return;
  }
  const note = notes.value.find(note => note.id === curNoteID.value);
  note.text = newNote.value;
  note.date = new Date().toLocaleString();
  showNoteModal.value = false;
  newNote.value = "";
  errorMessage.value = "";
  reEdit.value = false;
  isFullscreen.value = false;
}

const deleteNote = (noteID) => {
  notes.value.splice(notes.value.findIndex(note => note.id === noteID), 1);
}

const reEditnNote = (noteID) => {
  const note = notes.value.find(note => note.id === noteID);
  newNote.value = note.text;
  curNoteID.value = noteID;
  showNoteModal.value = true;
  reEdit.value = true;
}

const isFullscreen = ref(false);

const toggleFullscreen = () => {
  isFullscreen.value = !isFullscreen.value;
}

const dragStart = (index) => {
  draggedIndex.value = index;
}

const drop = (index) => {
  const draggedNote = notes.value.splice(draggedIndex.value, 1)[0];
  notes.value.splice(index, 0, draggedNote);
}
</script>

<template>
  <main>
    <div v-show="showNoteModal" class="overlay">
      <div :class="['modal', { fullscreen: isFullscreen }]">
        <div class="header">
          <p @click="toggleFullscreen"> {{ isFullscreen ? '🌐' : '🌏' }}</p>
          <p @click="showNoteModal = false" >❌</p>
        </div>

        <div class="editor">
          <textarea class="origin" v-model.trim="newNote" @input="update"/>
          <div class="preview" v-html="renderMarkdown"></div>
        </div>
        <p v-if="errorMessage">{{ errorMessage }}</p>
        <button v-show="!reEdit" @click="addNote">Add Note</button>
        <button v-show="reEdit" @click="saveNote">Save Note</button>
      </div>
    </div>
    <div class="container">
      <header>
        <h1>Notes📝</h1>
        <button @click="showNoteModal = true">+</button>
      </header>
      <div class="cards-container">
        <div 
          v-for="(note, index) in notes" 
          class="card" 
          :key="note.id"
          :style="{ 
            backgroundColor: note.backgroundColor 
          }"
          draggable="true"
          @dragstart="dragStart(index)"
          @dragover.prevent
          @drop="drop(index)"
        >
          <div class="edit">
            <!-- edit the note -->
            <p @click="reEditnNote(note.id)">✏️</p>
            <!-- delete the note -->
            <p @click="deleteNote(note.id)">❌</p>
          </div>
          <div class="main-text" v-html="marked(note.text)"></div>
          <p class="date">{{ note.date }}</p>
        </div>
      </div>
    </div>
  </main>
</template>

<style scoped>
main {
  height: 100vh;
  width: 100vw;
}

.container {
  max-width: 1000px;
  padding: 10px;
  margin: 0 auto;
}

header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

h1 {
  font-weight: bold;
  margin-bottom: 25px;
  font-size: 75px;
}

header button {
  border: none;
  border-radius: 100%;
  padding: 10px;
  width: 50px;
  height: 50px;
  cursor: pointer;
  background-color: rgb(21, 20, 20);
  color: white;
  font-size: 25px;
}

.card {
  width: 225px;
  height: 225px;
  background-color: rgb(237, 182, 44);
  padding: 10px;
  border-radius: 15px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  margin-right: 20px;
  margin-bottom: 20px;
  cursor: grab;
}

.card:active {
  cursor: grabbing;
}

.card .edit {
  display: flex;
  justify-content: flex-end;
}

.card .edit p {
  cursor: pointer;
}

.main-text {
  line-height: 1.25;
  font-size: 17.5px;
  font-weight: bold;
  overflow: hidden;
  text-overflow: ellipsis;
  /* white-space: nowrap; */
}

.date {
  font-size: 12.5px;
  margin-top: auto;
  margin-left: auto;
}

.cards-container {
  display: flex;
  flex-wrap: wrap;
}

.overlay {
  position: absolute;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.77);
  z-index: 10;
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal {
  width: 750px;
  height: 480px;
  background-color: white;
  border-radius: 10px;
  padding: 10px;
  position: relative;
  display: flex;
  flex-direction: column;
  transition: width 0.3s, height 0.5s;
}

.modal.fullscreen {
  width: 100%;
  height: 100%;
  border-radius: 0;
}

.modal button {
  padding: 8px;
  font-size: 15px;
  background-color: blueviolet;
  border: none;
  border-radius: 5px;
  color: white;
  cursor: pointer;
  width: 15%;
  margin-top: 15px;
  margin-left: auto;
}


.modal .header {
  margin-left: auto;
  display: flex;
  align-items: center;
  padding: 10px;
  font-size: 20px;
  cursor: pointer;
}


.modal p {
  color: red;
}
  
.modal .editor {
  width: 100%;
  height: 350px;
  padding: 5px;
  display: flex;
  transition: width 0.3s, height 0.5s;
}

.modal.fullscreen .editor {
  height: 100%;
}

.modal .origin,
.modal .preview {
  overflow: auto;
  width: 50%;
  height: 100%;
  box-sizing: border-box;
  padding: 0 10px;
}

.modal .origin {
  border: none;
  border-right: 1px solid #ccc;
  resize: none;
  outline: none;
  background-color: #f6f6f6;
  font-size: 14px;
  font-family: 'Monaco', courier, monospace;
  padding: 20px;
}

</style>