<template>
  <div class="flex flex-col justify-center items-center pt-10 mb-10">
    <div v-if="showModal" @click="clearErrors" class="w-full h-full absolute flex justify-center items-center inset-0 bg-gray-500 bg-opacity-60">
      <div class="w-96 py-10 rounded-lg text-center bg-gray-100 border-2 border-red-500">
        <span v-for="error in errorMessage" :key="error.ID">
          <p class="text-red-500 text-lg">{{error}}</p>
        </span>
      </div>
    </div>
    <div class="text-center mb-10">
      <h3 class="text-xl">Hello World!</h3>
      <h1 class="text-3xl">Welcome to my Cloudflare page</h1>
      <p>Feel free to drop in and add a post about yourself. As of October 26st 2021, this should be populated by a handful of post by me (Luis A Rodriguez).</p>
      <button class="bg-blue-600 text-gray-100 px-5 py-1 rounded-md text-lg font-medium mt-4" @click="getPosts">Refresh Posts</button>
    </div>
    <div v-if="startLoading" class="flex flex-wrap justify-center gap-10">

      <PostCard v-for="post in posts" :key="post.id"
        :title="post.postTitle"
        :userName="post.postUser"
        :postUrl="post.postUrl"
        :text="post.postText"
      />

    </div>

    <div class="w-96 my-6 border-2 border-gray-500 rounded-md p-4 shadow-xl">
      <h3 class="text-center text-2xl">Post Away!</h3>
      <form class="space-y-3" @submit.prevent="postIt">
        <div class="border-b-2">
          <input v-model="postTitle" class="w-full text-center" type="text" placeholder="Title">
        </div>
        <div class="border-b-2">
          <input v-model="postUser" class="w-full text-center" type="text" placeholder="User Name">
        </div>
        <div class="border-b-2">
          <input v-model="postUrl" class="w-full text-center" type="text" placeholder="Image Url">
        </div>
        <div class="border-2">
          <textarea v-model="postText" class="block w-full" placeholder="Post text goes here"></textarea>
        </div>
        <div class="flex justify-center">
          <button class="bg-blue-600 text-gray-100 px-5 py-1 rounded-md text-lg font-medium">Submit</button>
        </div>
      </form>
    </div>

  </div>
</template>
 
<script>
import PostCard from './components/PostCard';

export default {
  beforeMount() {
    this.getPosts();
    this.startLoading = true;
  },
  data() {
    return{
      startLoading: false,
      showModal: false,
      errorMessage: [],
      postTitle: '',
      postUser: '',
      postUrl: '',
      postText: '',
      posts: []
    };
  },
  components: {
    PostCard
  },
  methods: {
    async getPosts() {
      this.posts = [];
      const res = await fetch('https://my-worker.luis-rodriguez.workers.dev/');
      let data = await res.json();
      const keys = [];
      for (let i = 0; i < data.keys.length; i++) {
        keys[i] = data.keys[i].name;
        const obj = {
          postTitle: decodeURI(data.keys[i].name)
        }
        this.posts.push(obj);
      }
      for (let i = 0; i < keys.length; i++) {
        const res = await fetch(`https://my-worker.luis-rodriguez.workers.dev/${keys[i]}`);
        const data = await res.json();
        const post = JSON.parse(data);
        this.posts[i].postUser = post.user;
        this.posts[i].postUrl = post.url;
        this.posts[i].postText = post.text;
      }
    },
    async postIt() {
      if (this.postTitle === '') {
        this.errorMessage.push('Please enter a title');
      }
      if (this.postUser === '') {
        this.errorMessage.push('Please enter a user name');
      }
      if (this.postUrl === '') {
        this.errorMessage.push('Please enter an image Url');
      }
      if (this.postText === '') {
        this.errorMessage.push("Don't forget the text for the post!");
      }
      if (this.errorMessage.length !== 0) {
        this.showModal = true;
        return;
      }

      const post = {
        user: this.postUser,
        url: this.postUrl,
        text: this.postText
      }

      await fetch(`https://my-worker.luis-rodriguez.workers.dev/${this.postTitle}`, {
        method: 'POST',
        body: JSON.stringify(post)
      });

      this.postTitle = '';
      this.postUser = '';
      this.postUrl = '';
      this.postText = '';
      this.getPosts();
    },
    clearErrors() {
      this.errorMessage = [];
      this.showModal = false;
    }
  }
}
</script>

