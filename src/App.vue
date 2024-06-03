<template>
  <div class="container" :style="containerStyle">
    <form @submit.prevent="save" class="form">
      <input 
        type="text" 
        v-model="form.title" 
        placeholder="Title" 
        class="input title-input"
      /><br />
      <textarea 
        v-model="form.content" 
        placeholder="Content" 
        class="textarea content-textarea"
      ></textarea><br />
      <button type="submit" class="button save-button">Save</button>  
    </form>
    <ul class="article-list">
      <li v-for="article in articles" :key="article.id" class="article-item">
        <strong class="article-title">{{ article.title }}</strong><br />
        <p class="article-content">{{ article.content }}</p>
        <button @click="edit(article)" class="button edit-button">Edit</button>
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

    const containerStyle = {
      backgroundImage: 'url("http://2.bp.blogspot.com/-gX4gQFmiBgs/VBuGufeo_DI/AAAAAAAAAtY/t9jES_C9Yp0/s1600/Nature+Background+twitter.jpg")',
      backgroundSize: 'auto',
      backgroundPosition: 'center',
      minHeight: '100vh',
      padding: '20px'
    };

    async function load() {
      try {
        const response = await axios.get('http://localhost:3000/articles');
        articles.value = response.data;
      } catch (error) {
        console.error('Error loading articles:', error);
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
          // Update existing article
          articles.value = articles.value.map((article) =>
            article.id === response.data.id ? response.data : article
          );
        } else {
          // Add new article
          articles.value.push(response.data);
        }

        // Reset form
        form.id = null;
        form.title = '';
        form.content = '';
      } catch (error) {
        console.error('Error saving article:', error);
      }
    }

    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    onMounted(load);

    return { form, articles, save, edit, load, containerStyle };
  }
}
</script>

<style>
.container {
  padding: 20px;
}
.form {
  margin-bottom: 20px;
}
.input, .textarea {
  width: 100%;
  margin-bottom: 10px;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
}
.button {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}
.save-button {
  background-color: #28a745;
  color: white;
}
.load-button {
  background-color: #007bff;
  color: white;
}
.edit-button {
  background-color: #ffc107;
  color: white;
}
.article-list {
  list-style: none;
  padding: 0;
}
.article-item {
  margin-bottom: 20px;
}
.article-title {
  font-size: 1.2em;
  margin-bottom: 5px;
}
.article-content {
  margin-bottom: 10px;
}
</style>