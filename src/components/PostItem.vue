<script setup>
import { onMounted, ref } from "vue";
import CommentItem from "../components/CommentItem.vue";
import NewComment from "./NewComment.vue";
import { collection, getDocs, query, addDoc } from "firebase/firestore";
import db from "../firebase/init.js";

const emits = defineEmits(["addComment"]);

const props = defineProps({
  post: {
    type: Object,
    required: true,
  },
});

const isComment = ref(false);
const post = ref(props.post);

const commentRef = collection(db, "posts", post.value.id, "comments");

async function getComments() {
  // onSnapshot(
  //   commentRef,
  //   async (commentSnap) => {
  //     commentSnap.docChanges().forEach((changeComment) => {
  //       switch (changeComment.type) {
  //         case "added":
  //           let comment = changeComment.doc.data();
  //           comment.id = changeComment.doc.id;
  //           post.value.comments.push(comment);
  //           break;
  //         case "modified":
  //           // Handle modified document
  //           break;
  //         case "removed":
  //           let delCommentIndex = post.value.comments.findIndex(
  //             (e) => e.id === change.doc.id
  //           );
  //           post.value.comments.splice(delCommentIndex, 1);
  //           break;
  //       }
  //     });
  //   },
  //   (err) => {
  //     console.log(`Encountered error: ${err}`);
  //   }
  // );

  let qry = query(commentRef);
  let snap = await getDocs(qry);
  post.value.comments = snap.docs.map((doc) => doc.data());
  // console.log(post.value);
}

onMounted(async () => {
  getComments();
});

const addNewComment = async (commentDetail) => {
  if (
    commentDetail.user !== "" &&
    commentDetail.email !== "" &&
    commentDetail.comment !== "" &&
    commentDetail.stars !== null
  ) {
    await addDoc(commentRef, commentDetail);
    getComments();
  } else console.error("Cannot add doc");
};
</script>

<template>
  <div class="box">
    {{ post.body }}
    <h4 class="title">comments ({{ post.comments.length }})</h4>
    <CommentItem
      v-for="comment in post.comments"
      :comment="comment"
      :key="comment.id"
    />
    <button
      class="bg-[#00BD7E] text-black rounded px-3"
      @click="() => (isComment = !isComment)"
    >
      Add Comment
    </button>
    <NewComment
      v-if="isComment"
      :postId="post.id"
      @addComment="addNewComment"
    />
  </div>
</template>

<style scoped>
.box {
  border: 1px solid grey;
  padding: 5px;
  margin: 5px;
  border-radius: 5px;
}
.title {
  font-style: italic;
  color: hsla(160, 100%, 37%, 1);
}
</style>
