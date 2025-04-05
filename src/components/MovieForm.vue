<template>
  <div>
    <div v-if="message" class="alert alert-success">{{ message }}</div>
    <div v-if="errors.length" class="alert alert-danger">
      <ul>
        <li v-for="error in errors" :key="error">{{ error }}</li>
      </ul>
    </div>
    <form id="movieForm" @submit.prevent="saveMovie">
      <div class="form-group mb-3">
        <label for="title" class="form-label">Movie Title</label>
        <input type="text" name="title" class="form-control" v-model="formData.title" />
      </div>
      <div class="form-group mb-3">
        <label for="description" class="form-label">Description</label>
        <textarea name="description" class="form-control" v-model="formData.description"></textarea>
      </div>
      <div class="form-group mb-3">
        <label for="poster" class="form-label">Poster</label>
        <input type="file" name="poster" class="form-control" @change="handleFileUpload" />
      </div>
      <button type="submit" class="btn btn-primary">Add Movie</button>
    </form>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';

let csrf_token = ref('');
let message = ref('');
let errors = ref([]);
let formData = ref({
  title: '',
  description: '',
  poster: null
});

function getCsrfToken() {
  fetch('/api/v1/csrf-token')
    .then(response => response.json())
    .then(data => {
      csrf_token.value = data.csrf_token;
    });
}

function handleFileUpload(event) {
  formData.value.poster = event.target.files[0];
}

function saveMovie() {
  let form_data = new FormData();
  form_data.append('title', formData.value.title);
  form_data.append('description', formData.value.description);
  form_data.append('poster', formData.value.poster);

  fetch('/api/v1/movies', {
    method: 'POST',
    body: form_data,
    headers: {
      'X-CSRFToken': csrf_token.value
    }
  })
    .then(response => response.json())
    .then(data => {
      if (data.message) {
        message.value = data.message;
        errors.value = [];
      } else if (data.errors) {
        message.value = '';
        errors.value = data.errors;
      }
    })
    .catch(error => {
      console.log(error);
    });
}

onMounted(() => {
  getCsrfToken();
});
</script>