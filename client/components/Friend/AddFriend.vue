<script setup lang="ts">
import { useUserStore } from "@/stores/user";
import { storeToRefs } from "pinia";
import { ref } from "vue";
import { fetchy } from "../../utils/fetchy";

import Modal from "../UtilComponents/Modal.vue";
import SearchFriend from "./SearchFriend.vue";

const { logoutUser, updateSession } = useUserStore();
const { currentUsername, isLoggedIn } = storeToRefs(useUserStore());

const isModalVisible = ref(false);
const isValidNewFriend = ref(false);
const messageType = ref("");
const message = ref("");
const friendToAdd = ref("");
const errorMessage = ref("");

async function showModal() {
  isModalVisible.value = true;
}

async function closeModal() {
  isModalVisible.value = false;
  isValidNewFriend.value = false;
  messageType.value = "";
  message.value = "";
  friendToAdd.value = "";
}

async function checkValidNewFriend(username: string) {
  // user must not be self
  if (username == currentUsername.value) {
    errorMessage.value = "Cannot add yourself as a friend!";
    isValidNewFriend.value = false;
    return;
  }

  const query: Record<string, string> = { username };
  let user;

  try {
    user = await fetchy(`/api/users/${username}`, "GET", { query });
  } catch {
    errorMessage.value = `User ${friendToAdd.value} not found!`;
    isValidNewFriend.value = false;
    return;
  }

  // user must exist
  if (user) {
    // must not already be friends
    // must not already have sent a request
    const friends = await fetchy("/api/friends", "GET", {});

    if (friends.includes(username)) {
      errorMessage.value = `You are already friends with ${username}`;
      isValidNewFriend.value = false;
      friendToAdd.value = "";
    } else {
      isValidNewFriend.value = true;
      friendToAdd.value = username;
      errorMessage.value = "";
    }
  }
}

function handleMessageUploaded(messageLink: string, type: "Video" | "Audio") {
  messageType.value = type;
  message.value = messageLink;
}

async function addFriend() {
  try {
    await fetchy(`/api/friend/requests/${friendToAdd.value}`, "POST", {
      body: { to: friendToAdd.value, message: message.value, messageType: messageType.value },
    });
    closeModal();
  } catch (_) {
    return;
  }
}
</script>

<template>
  <button type="button" class="btn" v-on:click="showModal" id="showModalButton">
    <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-person-fill-add" viewBox="0 0 16 16">
      <path d="M12.5 16a3.5 3.5 0 1 0 0-7 3.5 3.5 0 0 0 0 7Zm.5-5v1h1a.5.5 0 0 1 0 1h-1v1a.5.5 0 0 1-1 0v-1h-1a.5.5 0 0 1 0-1h1v-1a.5.5 0 0 1 1 0Zm-2-6a3 3 0 1 1-6 0 3 3 0 0 1 6 0Z" />
      <path d="M2 13c0 1 1 1 1 1h5.256A4.493 4.493 0 0 1 8 12.5a4.49 4.49 0 0 1 1.544-3.393C9.077 9.038 8.564 9 8 9c-5 0-6 3-6 4Z" />
    </svg>
    Add Friend
  </button>

  <Modal v-show="isModalVisible" v-on:closed-modal="closeModal">
    <template v-slot:header> Add Friend </template>

    <template v-slot:body>
      <main>
        <section class="inputGroup">
          <SearchFriend @search-user="checkValidNewFriend" />
          <p v-if="errorMessage" id="errorMessage">{{ errorMessage }}</p>
        </section>

        <!-- submit message -->
        <button v-if="isValidNewFriend" type="submit" class="pure-button-primary pure-button" @click="addFriend" id="confirmButton">
          <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="currentColor" class="bi bi-plus" viewBox="-1 1 16 16">
            <path d="M8 4a.5.5 0 0 1 .5.5v3h3a.5.5 0 0 1 0 1h-3v3a.5.5 0 0 1-1 0v-3h-3a.5.5 0 0 1 0-1h3v-3A.5.5 0 0 1 8 4z" />
          </svg>
          Add friend
        </button>
        <!-- </section> -->
      </main>
    </template>
  </Modal>
</template>

<style scoped>
main {
  height: 100%;
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;
}

#showModalButton {
  height: 30px;
  width: 125px;
}

button svg {
  margin-right: 4px;
  margin-left: 0px;
}

#errorMessage {
  font-weight: bold;
  margin-top: 2px;
}

.inputGroup {
  display: flex;
  flex-direction: column;
  justify-content: center;
  width: 100%;
  align-items: center;
}

.inputGroup input {
  width: 50%;
}

.hello {
  display: flex;
  justify-content: center;
  margin: 16px;
}

.confirm {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  height: auto;
  margin: 16px;
}

#confirmButton {
  height: 40px;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: row;
  margin: 12px;
}
</style>
