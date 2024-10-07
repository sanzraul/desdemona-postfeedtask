<template>
  <div>
    <div v-for="post in posts" :key="post.id" class="post-item">
      <div class="post-image-container">

        <img :src="imageUrl" alt="Post Image" class="post-image">
      </div>

      <h2 class="post-title">{{ post.title }}</h2>

      <p class="post-body">{{ post.body }}</p>

      <small>Posted by user: {{ post.userId }}</small>

      <button @click="loadComments(post.id)" class="comments-toggle">
        {{ comments[post.id] ? 'Hide comments' : 'Show comments' }}
      </button>

      <div v-if="comments[post.id]" class="comments-section">
        <h3>Comments</h3>
        <ul>
          <li v-for="comment in comments[post.id]" :key="comment.id">
            <strong>{{ comment.name }}:</strong> {{ comment.body }}
          </li>
        </ul>
      </div>

      <button @click="toggleAddComment(post.id)" class="add-comment-btn">
        Add Comment
      </button>

      <form v-if="showCommentForm[post.id]" @submit.prevent="addComment(post.id)" class="add-comment-form">
        <textarea v-model="newComment" placeholder="Write your comment"></textarea>
        <button type="submit" class="submit-btn">Submit</button>
      </form>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      posts: [],
      comments: {},
      showCommentForm: {},
      newComment: '',
      imageUrl: 'https://fakeimg.pl/300x150',
    };
  },
  async mounted() {
    const response = await this.$axios.get('/posts');
    this.posts = response.data;
  },
  methods: {
    async loadComments(postId) {
      if (!this.comments[postId]) {
        const response = await this.$axios.get(`/posts/${postId}/comments`);
        this.comments = { ...this.comments, [postId]: response.data };
      } else {
        this.$set(this.comments, postId, null);
      }
    },
    toggleAddComment(postId) {
      this.$set(this.showCommentForm, postId, !this.showCommentForm[postId]);
    },
    async addComment(postId) {
      if (this.newComment.trim() === '') return;

      const comment = {
        postId,
        body: this.newComment,
        name: 'Default User',
        email: 'user1@example.com',
        userId: 1,
      };

      await this.$axios.post(`/posts/${postId}/comments`, comment);
      this.comments[postId] = [...(this.comments[postId] || []), comment];
      this.newComment = '';
      this.toggleAddComment(postId);
    },
  },
};
</script>

<style scoped>
.post-item {
  border-bottom: 1px solid #ccc;
  padding: 20px;
  margin-bottom: 20px;
}
.post-image-container{
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}
.post-image {
  width: 20%;
  height: auto;
  align-self: center;
}

.post-title {
  font-size: 1.5rem;
  margin-top: 10px;
}

.post-body {
  font-size: 1rem;
  margin: 10px 0;
}

.comments-toggle,
.add-comment-btn,
.submit-btn {
  margin-top: 10px;
  padding: 8px 15px;
  background-color: #007bff;
  color: white;
  border: none;
  cursor: pointer;
}

.comments-toggle:hover,
.add-comment-btn:hover,
.submit-btn:hover {
  background-color: #0056b3;
}

.comments-section {
  margin-top: 15px;
}

.add-comment-form {
  margin-top: 10px;
}

.add-comment-form textarea {
  width: 100%;
  padding: 10px;
  margin-bottom: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
}
</style>
