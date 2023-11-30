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
} from "firebase/firestore";
import db from "../firebase/init.js";
import PostItem from "../components/PostItem.vue";

const user = ref({});
const posts = ref([]);
const route = useRoute();

// async function getComments(postId) {
//   const commentRef = collection(db, "posts", postId, "comments");
//   let comments = await getDocs(query(commentRef));
//   comments = comments.docs.map((e) => e.data());
//   return comments;

// console.log(comments);
// onSnapshot(
//   commentRef,
//   async (commentSnap) => {
//     // comments = commentSnap.docChanges();
//     // comments = comments.map((e) => e.doc.data());
//     // console.log(comments);

//     let comments = [];
//     commentSnap.docChanges().forEach((changeComment) => {
//       switch (changeComment.type) {
//         case "added":
//           let comment = changeComment.doc.data();
//           comment.id = changeComment.doc.id;
//           comments.push(comment);
//           break;
//         case "modified":
//           // Handle modified document
//           break;
//         case "removed":
//           let delCommentIndex = data.comments.findIndex(
//             (e) => e.id === change.doc.id
//           );
//           data.comments.splice(delCommentIndex, 1);
//           break;
//       }
//     });
//     return comments;
//   },
//   (err) => {
//     console.log(`Encountered error: ${err}`);
//   }
// );
// }

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
            // let comments = await getComments(data.id);
            // data.comments = comments;

            // const commentRef = collection(db,"posts", change.doc.id, "comments")
            // onSnapshot(commentRef, (commentSnap) => {
            //   // let comments = commentSnap.docChanges()
            //   // comments = comments.map(e=> e.doc.data())
            //   // data.comments = comments

            //   commentSnap.docChanges().forEach((changeComment) => {
            //     switch (changeComment.type) {
            //     case 'added':
            //       let comment = changeComment.doc.data();
            //       comment.id = changeComment.doc.id;
            //       data.comments.push(comment);
            //       break;
            //     case 'modified':
            //       // Handle modified document
            //       break;
            //     case 'removed':
            //       let delCommentIndex = data.comments.findIndex(e => e.id === change.doc.id)
            //       data.comments.splice(delCommentIndex,1)
            //       break;
            // }})
            // }, err => {
            //     console.log(`Encountered error: ${err}`);
            // })
            // console.log(posts.value);
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
}

watch(() => route.params.user, getPosts);

onMounted(() => {
  getPosts();
});
</script>

<template>
  <h3>Posts : {{ user.firstname }} {{ user.lastname }} ({{ posts.length }})</h3>
  <PostItem v-for="post in posts" :post="post" :key="post.id" />
</template>

<style scoped></style>
