<script setup>
import { ref, onMounted, watch } from "vue";
import { useRoute } from "vue-router";
import {
  collection,
  query,
  where,
  getDocs,
  onSnapshot,
  doc,
  getDoc,
  updateDoc,
} from "firebase/firestore";
import db from "../firebase/init.js";
import PostItem from "../components/PostItem.vue";

const user = ref({});
const posts = ref([]);
const route = useRoute();

async function getPosts() {
  let userId = route.params.user;
  const snap = await getDoc(doc(db, "users", userId));
  user.value = snap.data();

  const postsRef = collection(db, "posts");

  let qry = null;
  qry = query(postsRef, where("user", "==", userId));

  let data;
  onSnapshot(
    qry,
    async (snapshot) => {
      //docChanges => catch only doc that change
      posts.value = [];
      for (const change of snapshot.docChanges()) {
        switch (change.type) {
          case "added":
            data = change.doc.data(); // add all data to ref
            data.id = change.doc.id;
            data.user = user.value;
            data.comments = [];
            posts.value.push(data);
            break;
          case "modified":
            // Handle modified document
            break;
          case "removed":
            let delPostIndex = posts.value.findIndex(
              (e) => e.id == change.doc.id
            );
            posts.value.splice(delPostIndex, 1);
            break;
        }
      }
    },
    (err) => {
      console.log(`Encountered error: ${err}`);
    }
  );

  //computed pattern
  setInterval(async () => {
    await updateDoc(doc(db, "users", userId), {
      amountpost: posts.value.length,
    });
    getPosts();
  }, 1000 * 60); // computed every 1 min
}

watch(() => route.params.user, getPosts);

onMounted(() => {
  getPosts();
});
</script>

<template>
  <h3>
    Posts : {{ user.firstname }} {{ user.lastname }} ({{ user.amountpost }})
  </h3>
  <PostItem v-for="post in posts" :post="post" :key="post.id" />
</template>

<style scoped></style>
