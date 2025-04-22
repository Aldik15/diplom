<!-- frontend/src/views/Posts.vue -->
<template>
  <div class="posts-container">
    <h2>Посты</h2>
    <div class="create-post mb-4">
      <form @submit.prevent="createPost">
        <div class="mb-3">
          <textarea
            class="form-control"
            v-model="newPostDescription"
            placeholder="Что у вас нового?"
            rows="4"
          ></textarea>
        </div>
        <button type="submit" class="btn btn-primary">Опубликовать</button>
      </form>
    </div>
    <div v-if="errorMessage" class="alert alert-danger">
      {{ errorMessage }}
    </div>
    <div v-if="successMessage" class="alert alert-success">
      {{ successMessage }}
    </div>
    <div v-if="posts.length === 0" class="text-center">
      No posts yet.
    </div>
    <div v-else class="posts-list">
      <div v-for="post in posts" :key="post.id" class="card mb-3">
        <div class="card-body">
          <p class="card-text">{{ post.description }}</p>
          <p class="card-subtitle text-muted">
            Автор: {{ post.username }} | {{ post.created_at }} | Лайки: {{ post.likes_count }}
          </p>
          <button class="btn btn-outline-primary btn-sm" @click="likePost(post.id)">❤️ Like</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import api from '../api';

export default {
  data() {
    return {
      posts: [],
      newPostDescription: '',
      errorMessage: '',
      successMessage: '',
    };
  },
  async created() {
    await this.fetchPosts();
  },
  methods: {
    async fetchPosts() {
      this.errorMessage = '';
      try {
        console.log('Fetching posts...');
        const response = await api.get('/posts');
        console.log('Posts response:', response.data);
        this.posts = response.data;
        if (this.posts.length === 0) {
          console.log('No posts yet');
        }
      } catch (error) {
        this.errorMessage = error.response?.data?.detail || 'Failed to load posts';
        console.error('Fetch posts error:', error.response?.data || error.message);
      }
    },
    async createPost() {
      this.errorMessage = '';
      this.successMessage = '';
      if (!this.newPostDescription.trim()) {
        this.errorMessage = 'Post description cannot be empty';
        return;
      }
      try {
        console.log('Creating post:', { description: this.newPostDescription });
        const response = await api.post('/posts', {
          description: this.newPostDescription,
        });
        console.log('Create post response:', response.data);
        this.successMessage = 'Post created successfully!';
        this.newPostDescription = '';
        await this.fetchPosts();
      } catch (error) {
        this.errorMessage = error.response?.data?.detail || 'Failed to create post';
        console.error('Create post error:', error.response?.data || error.message);
      }
    },
    async likePost(postId) {
      this.errorMessage = '';
      try {
        console.log('Liking post:', postId);
        const response = await api.post(`/posts/${postId}/like`);
        console.log('Post liked:', response.data);
        await this.fetchPosts();
      } catch (error) {
        this.errorMessage = error.response?.data?.detail || 'Failed to like post';
        console.error('Like post error:', error.response?.data || error.message);
      }
    },
  },
};
</script>

<style scoped>
.posts-container {
  max-width: 800px;
  margin: 0 auto;
  padding: 2rem;
}
.create-post {
  background: #fff;
  padding: 1rem;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}
.card {
  background: #fff;
  border-radius: 8px;
}
.btn-sm {
  margin-top: 10px;
}
</style>