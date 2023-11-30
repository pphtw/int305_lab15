<script setup>
import { reactive } from "vue";
import { serverTimestamp } from "firebase/firestore";

const emits = defineEmits(["addComment"]);

const props = defineProps({
  postId: {
    type: String,
    require: true,
  },
});

const commentDetail = reactive({
  comment: "",
  email: "",
  name: "",
  stars: 0,
  cmtdate: serverTimestamp(),
});

const clearData = () => {
  commentDetail.comment = "";
  commentDetail.email = "";
  commentDetail.name = "";
  commentDetail.stars = 0;
};

const addNewCommentHandler = () => {
  // if (
  //   commentDetail.user !== "" &&
  //   commentDetail.email !== "" &&
  //   commentDetail.comment !== "" &&
  //   commentDetail.stars !== null
  // ) {
  //   const commentRef = collection(db, "posts", props.postId, "comments");
  //   await addDoc(commentRef, commentDetail);
  //   clearData();
  // } else console.error("Cannot add doc");
  emits("addComment", commentDetail);
  clearData();
};
</script>

<template>
  <div class="my-2 flex flex-col gap-y-2 w-full">
    <div class="flex flex-row gap-x-3 w-full">
      <label>stars:</label>
      <input
        type="number"
        class="rounded bg-[#363636] px-2 w-12"
        min="0"
        max="5"
        placeholder="star"
        v-model.trim="commentDetail.stars"
      />
      <label>username: </label>
      <input
        type="text"
        class="rounded bg-[#363636] px-2 w-1/5"
        placeholder="john"
        v-model.trim="commentDetail.name"
      />
      <label>email: </label>
      <input
        type="email"
        class="rounded bg-[#363636] px-2 w-1/5"
        placeholder="example@email.com"
        v-model.trim="commentDetail.email"
      />
    </div>

    <div class="flex flex-row gap-x-3 w=full">
      <label>comment: </label>
      <textarea
        class="rounded bg-[#363636] px-2 w-1/2 h-20"
        placeholder="Comments something on this post!"
        v-model.trim="commentDetail.comment"
      />
      <button
        class="bg-[#00BD7E] w-1/12 h-10 text-black rounded px-3"
        @click="addNewCommentHandler"
      >
        submit
      </button>
    </div>
  </div>
</template>

<style scoped></style>
