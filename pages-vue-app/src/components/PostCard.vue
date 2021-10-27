<template>
    <div class="w-96 rounded flex items-center text-center flex-col p-6 border-blue-400 border-8 bg-gray-100 shadow-xl">
        <div class="mb-4">
          <h2 class="text-2xl">{{ title }}</h2>
          <span>By: {{ userName }}</span>
        </div>
        <img class="w-3/4 rounded mb-6" :src="postUrl">
        <p>{{ text }}</p>
    </div>
</template>

<script>
export default {
    props: {
        title: String,
        userName: String,
        postUrl: String,
        text: String
    }
}
</script>