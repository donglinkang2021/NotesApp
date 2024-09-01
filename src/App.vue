<script setup>
  import { ref } from 'vue';

  const showNoteModal = ref(false);
  const newNote = ref("");
  const notes = ref([]);
  const errorMessage = ref("");

  function getRandomColor() {
    return "hsl(" + Math.random() * 360 + ", 100%, 75%)";
  }

  const addNote = () => {
    if (newNote.value.length < 10) {
      errorMessage.value = "Note must be at least 10 characters long";
      return;
    }
    notes.value.push({ 
      id: Math.floor(Math.random() * 1000_000),
      text: newNote.value, 
      date: new Date().toLocaleString(), 
      backgroundColor: getRandomColor()
    });
    showNoteModal.value = false;
    newNote.value = "";
    errorMessage.value = "";
  }

</script>

<template>
  <main>
    <div v-show="showNoteModal" class="overlay">
      <div class="modal">
        <p @click="showNoteModal = false" class="close">❌</p>
        <textarea v-model.trim="newNote"/>
        <p v-if="errorMessage">{{ errorMessage }}</p>
        <button @click="addNote">Add Note</button>
      </div>
    </div>
    <div class="container">
      <header>
        <h1>Notes📝</h1>
        <button @click="showNoteModal = true">+</button>
      </header>
      <div class="cards-container">
        <div 
          v-for="note in notes" 
          class="card" 
          :key="note.id"
          :style="{ 
            backgroundColor: note.backgroundColor 
          }"
        >
          <p class="main-text">{{ note.text }}</p>
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
  }

  .main-text {
    line-height: 1.25;
    font-size: 17.5px;
    font-weight: bold;
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
    background-color: white;
    border-radius: 10px;
    padding: 10px;
    position: relative;
    display: flex;
    flex-direction: column;
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

  .modal .close {
    margin-left: auto;
    font-size: 20px;
    z-index: 100000;
    cursor: pointer;
  }

  .modal p {
    color: red;
  }

  .modal textarea {
    width: 100%;
    height: 250px;
    padding: 10px;
    font-size: 20px;
  }

</style>