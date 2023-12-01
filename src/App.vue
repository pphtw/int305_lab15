<script setup>
import { RouterView } from "vue-router";
import { ref, onMounted } from "vue";
import { collection, onSnapshot } from "firebase/firestore";
import db from "./firebase/init.js";
import UserList from "./components/UserList.vue";

const users = ref([]);

async function getUsers() {
  const usersRef = collection(db, "users");

  onSnapshot(
    usersRef,
    (snapshot) => {
      //docChanges => catch only doc that change
      snapshot.docChanges().forEach((change) => {
        switch (change.type) {
          case "added":
            let data = change.doc.data();
            data.id = change.doc.id;
            users.value.push(data);
            break;
          case "modified":
            // Handle modified document
            break;
          case "removed":
            let delUserIndex = users.value.findIndex(
              (e) => e.id === change.doc.id
            );
            users.value.splice(delUserIndex, 1);
            break;
        }
      });
    },
    (err) => {
      console.log(`Encountered error: ${err}`);
    }
  );
}

onMounted(() => {
  getUsers();
});
</script>

<template>
  <div>
    <div>
      <UserList :users="users" />
    </div>
  </div>
  <div class="content">
    <RouterView />
  </div>
</template>

<style scoped></style>
