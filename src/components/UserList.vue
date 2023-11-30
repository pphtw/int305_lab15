<script setup>
import { RouterLink, RouterView } from "vue-router";

defineProps({
  users: {
    type: Array,
    required: true,
  },
});
</script>

<template>
  <div class="greetings ml-28">
    <h3>User List</h3>
    <ul class="list-disc marker:text-[#00BD7E]">
      <li v-for="user in users" :key="user.id">
        <RouterLink :to="`/posts/${user.id}`">{{ user.id }}</RouterLink>
      </li>
    </ul>
  </div>
</template>

<style scoped></style>
