<script setup>
import { onMounted, ref } from "vue";
import CommentItem from "../components/CommentItem.vue";
import NewComment from "./NewComment.vue";
import {
  collection,
  getDocs,
  query,
  addDoc,
  updateDoc,
  doc,
} from "firebase/firestore";
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
  let qry = query(commentRef);
  let snap = await getDocs(qry);
  post.value.comments = snap.docs.map((doc) => doc.data());

  setInterval(async () => {
    await updateDoc(doc(db, "posts", post.value.id), {
      amountcmt: post.value.comments.length,
    });
  }, 1000 * 60);
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
    <h4 class="title">comments ({{ post.amountcmt }})</h4>
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
