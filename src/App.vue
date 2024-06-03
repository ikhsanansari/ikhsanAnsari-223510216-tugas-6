<template>
  <div class="container">
    <form @submit.prevent="save" class="form">
      <input type="text" v-model="form.title" placeholder="Title" class="input" /><br>
      <textarea v-model="form.content" placeholder="Content" class="textarea"></textarea><br>
      <input type="hidden" v-model="form.id" />
      <button type="submit" class="button save-button">Save</button>
    </form>
    <ul class="article-list">
      <li v-for="article in articles" :key="article.id" class="article-item">
        <strong>{{ article.title }}</strong><br>
        {{ article.content }}<br>
        <button @click="edit(article)" class="button edit-button">Edit</button>
        <button @click="deleteArticle(article.id)" class="button delete-button">Delete</button>
      </li>
    </ul>
    <button @click="load" class="button load-button">Load</button>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from 'vue';
import axios from 'axios';

export default {
  setup() {
    const form = reactive({
      id: null,
      title: '',
      content: ''
    });

    const articles = ref([]);

    async function load() {
      try {
        const response = await axios.get('http://localhost:3000/articles');
        articles.value = response.data;
      } catch (error) {
        console.error('Error loading articles: ', error);
      }
    }

    async function save() {
      try {
        const url = form.id
          ? `http://localhost:3000/articles/${form.id}`
          : 'http://localhost:3000/articles';
        const method = form.id ? 'put' : 'post';
        const response = await axios[method](url, form);

        if (form.id) {
          articles.value = articles.value.map(article =>
            article.id === response.data.id ? response.data : article
          );
        } else {
          articles.value.push(response.data);
        }

        form.id = null;
        form.title = '';
        form.content = '';
      } catch (error) {
        console.error('Error saving article: ', error);
      }
    }

    async function deleteArticle(id) {
      try {
        await axios.delete(`http://localhost:3000/articles/${id}`);
        articles.value = articles.value.filter(article => article.id !== id);
      } catch (error) {
        console.error('Error deleting article: ', error);
      }
    }

    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    onMounted(load);

    return { form, articles, save, edit, load, deleteArticle };
  },
};
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@400;500&display=swap');

.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: 'Poppins', sans-serif; /* Menggunakan font Poppins */
  background-color: #f0f9ff; /* Biru Langit */
  border-radius: 8px;
}

.form {
  margin-bottom: 20px;
}

.input,
.textarea {
  width: 100%;
  padding: 10px;
  margin-bottom: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.button {
  padding: 10px 15px;
  margin-right: 5px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  font-family: 'Poppins', sans-serif; /* Menggunakan font Poppins */
}

.save-button {
  background-color: #51c5cf; /* Hijau Stabilo */
  color: white;
}

.load-button {
  background-color: #51c5cf; /* Hijau Stabilo */
  color: white;
}

.edit-button {
  background-color: #99ccff; /* Biru Langit Terang */
  color: white;
}

.delete-button {
  background-color: #ff7373; /* Merah Muda */
  color: white;
}

.article-list {
  list-style-type: none;
  padding: 0;
}

.article-item {
  background-color: #d9d9d9; /* Abu-abu */
  padding: 10px;
  margin-bottom: 10px;
  border-radius: 12px; /* Radius Lebih Bulat */
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.article-item strong {
  font-size: 18px;
  color: #4d4d4d; /* Abu-abu Tua */
}
</style>
