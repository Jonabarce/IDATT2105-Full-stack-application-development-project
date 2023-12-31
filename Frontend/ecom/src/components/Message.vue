<script setup>
import { ref, computed, onUpdated, nextTick, onMounted } from 'vue'
import axios from 'axios';
import { useTokenStore } from "../stores/userToken";
import { useI18n } from 'vue-i18n';

const { t } = useI18n();

let contacts = ref([])
let currentChat = ref(0)
let chat = ref([])
let chatInput = ref("")
let currentScrollLength = 0;
const tokenStore = useTokenStore()
let windowWidth = ref(window.innerWidth)
let showSide = ref(false)


function openChat(contactValue){
  const index = contacts.value.findIndex(c => c === contactValue)
  currentScrollLength = 0
  currentChat.value = index
	chatInput.value = ""
  if (windowWidth.value < 768) {
    switchSide()
  }
}

let searchText = ref('')

const filteredContacts = computed(() => {
  if (!searchText.value) {
    return contacts.value
  }

  return contacts.value.filter((contact) => {
    return (contact.firstName + contact.lastName).toLowerCase().includes(searchText.value.toLowerCase())
  })
})

function getLastMessage(contact){
  let contactIndex = contacts.value.indexOf(contact)
  if (chat.value[contactIndex]) {
    let thisChat = chat.value[contactIndex]
    let lastMessage = thisChat[thisChat.length-1][0]
    return (lastMessage.length > 35) ? lastMessage.substring(0, 35) + "..." : lastMessage;
  }
}

async function addChat(input) {

  const config = {
        headers: {
            "Content-type": "application/json",
            "Authorization" : "Bearer " + tokenStore.jwtToken
        },
    };

	if (input != "") {
    let message = {
      "toEmail": contacts.value[currentChat.value].email,
      "fromEmail": tokenStore.loggedInUser.email,
      "messageContent": input
    } 
    await axios.post("http://localhost:9090/api/messages/sendMessage", message,config)

		chat.value[currentChat.value].push([input, 0])
		chatInput.value = ""
		scrollToBottom()
	}
}

function scrollToBottom() {
  nextTick(() => {
      if (chat.value[currentChat.value]) {
        if (currentScrollLength < chat.value[currentChat.value].length) {
          let chatInstances = document.querySelector(".chatInstances");
            
          chatInstances.scrollTop = chatInstances.scrollHeight;
          currentScrollLength = chat.value[currentChat.value].length
        }
      }
    });
}

async function getContacts() {
  const config = {
        headers: {
            "Content-type": "application/json",
            "Authorization" : "Bearer " + tokenStore.jwtToken
        },
    };

  contacts.value = (await axios.get("http://localhost:9090/api/messages/" + tokenStore.loggedInUser.email + "/contacts",config)).data
  contacts.value.reverse();
}

async function getMessages() {
  
  const config = {
        headers: {
            "Content-type": "application/json",
            "Authorization" : "Bearer " + tokenStore.jwtToken
        },
    };
  

  let newChat = ref([])
  for (let i = 0; i < contacts.value.length; i++) {


    newChat.value.push([])
    const response = await axios.get('http://localhost:9090/api/messages/' + tokenStore.loggedInUser.email + "/" + contacts.value[i].email,config);


    let tempMessages = response.data
    
    for (const message in tempMessages) {
  
      let content = tempMessages[message].messageContent
      let sender = 1
      if (tempMessages[message].fromEmail === tokenStore.loggedInUser.email) {
        sender = 0
      }
  
      newChat.value[i].push([content, sender])
    }
  }
  

  chat.value = newChat.value
  
}

function switchSide() {
  showSide.value = (!showSide.value)
}

onUpdated(() => {
	scrollToBottom()
})

onMounted(() => {
  initialize()

  window.addEventListener('resize', () => {
    windowWidth.value = window.innerWidth
  })

  setInterval(() => {
    getContacts()
    getMessages()
    scrollToBottom()
  }, 1000);
})

async function initialize() {
    await getContacts()
    await getMessages()
}

</script>

<template>
  <div class="container">
    <div class="chats" v-if="!showSide">
      <div class="header">
        <img src="../assets/chat-dots-fill.svg" id="chatLogo" alt="Chat" />
        <h1>Chats</h1>
        <img src="../assets/arrow-bar-right.svg" id="arrowRight" alt="Arrow Right" v-if="windowWidth < 768" @click="switchSide()">
      </div>
      <hr />
      <input type="text" id="search" :placeholder="t('search')" v-model="searchText" />
      <div class="contacts">
        <div class="contact" v-for="contact in filteredContacts" @click="openChat(contact)">
            <img src="../assets/person-fill.svg" alt="Person img">
            <div>
                <h2> {{ contact.firstName + " " + contact.lastName }} </h2>
                <h3 v-if="windowWidth > 768">{{ getLastMessage(contact) }}</h3>
            </div>
        </div>
    </div>
    </div>

    <div class="chat" v-if="showSide || windowWidth > 768">
      <div class="header">
        <img src="../assets/person-fill.svg" alt="Person img" :style="{'margin-left': (windowWidth < 768) ? '3em' : 'none'}"/>
        <h1 v-if="contacts[currentChat]"> {{ contacts[currentChat].firstName + " " + contacts[currentChat].lastName }} </h1>
        <img src="../assets/arrow-bar-left.svg" id="arrowLeft" alt="Arrow Left" v-if="windowWidth < 768" @click="switchSide()">
      </div>
      <hr />
      <div class="chatInstances">
        <div
          id="chatInstance"
          v-for="chatInstance in chat[currentChat]"
          :class="{ chatSend: chatInstance[1] === 0, chatReceive: chatInstance[1] === 1 }"
        >
          {{ chatInstance[0] }}
        </div>
      </div>
      <div class="chatInputContainer">
        <input type="text" id="chatInput" :placeholder="t('sendAChat')" autocomplete="off" v-model="chatInput" @keydown.enter="addChat(chatInput)" />
        <img src="../assets/send.svg" alt="Send" class="send-icon" @click="addChat(chatInput)" />
      </div>
    </div>
  </div>
</template>

<style>
.container {
  display: flex;
  flex-flow: row wrap;
  justify-content: center;
  margin: 2em auto;
  align-items: center;
  height: 85vh;
  width: 90%;
  padding: 1em;
}

.chats {
  height: 100%;
  width: 50%;
  padding: 2em;
  border-radius: 15px 0 0 15px;
  box-shadow: 2px 5px 15px 4px rgba(0, 0, 0, 0.2);
}

.header {
  display: flex;
  justify-content: start;
  align-items: center;
}

#search {
  background: url('../assets/search.svg') no-repeat 1.5% 50%;
  background-size: 20px;
  color: var(--color-blue);
  padding-left: 2.5em;
  width: 93%;
  margin: 1em;
  transition: all 0.3s ease;
}

#search::-webkit-input-placeholder {
  color: var(--color-blue);
}

#chatLogo {
  height: 2em;
}

#arrowRight{
  position: absolute;
  height: 2em;
  top: 0;
  right: 0;
}

#arrowLeft{
  position: absolute;
  height: 2em;
  top: 0;
  left: 0;
  background-color: var(--color-background);
}

.contacts {
  overflow-y: scroll;
  height: 75%;
}

.contacts::-webkit-scrollbar {
  display: none;
}

.contact {
  display: flex;
  flex-flow: row wrap;
  gap: 1em;
  width: 99%;
  align-items: center;
  margin-top: 1em;
  border-radius: 15px;
  border: 2px solid transparent;
  box-shadow: 2px 3px 4px 2px rgba(0, 0, 0, 0.2);
}

.contact:hover {
  border: 2px solid var(--color-blue-light);
  cursor: pointer;
}

.contact img {
  height: 4em;
  margin: 0.5em 0 0.5em 0.5em;
  border-radius: 50px;
  background-color: var(--color-blue-light);
}

.chat {
	display: flex;
	flex-direction: column;
	width: 100%;
  height: 100%;
  flex: 1;
  justify-content: space-between;
  padding: 2em;
  border-radius: 0 15px 15px 0;
  box-shadow: 2px 5px 15px 4px rgba(0, 0, 0, 0.2);
	overflow: hidden;
}

.chatInstances {
  display: flex;
  flex-direction: column;
	justify-content: start;
  line-height: 1;
  padding: 0.5rem 0.875rem;
  position: relative;
  word-wrap: break-word;
	height: 100%;
  overflow-y: auto;
}

#chatInstance {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: fit-content;
  width: fit-content;
  max-width: 55%;
  padding: .7em 2em;
	font-size: medium;
  border-radius: 50px;
  margin-top: 1em;
}

.chatSend {
  margin-left: auto;
  background-color: var(--color-blue-light);
  color: #fff;
}

.chatReceive {
  margin-right: auto;
	background-color: #e5e5ea;
	color: #000;
}

.chatInputContainer {
  position: relative;
  display: flex;
}

#chatInput {
  color: var(--color-blue);
  background-color: var(--color-background);
  margin-top: 1em;
  padding-left: 2em;
  width: 100%;
  transition: all 0.3s ease;
}

#chatInput::-webkit-input-placeholder {
  color: var(--color-blue);
}

.send-icon {
  position: absolute;
  background: var(--color-background);
  right: 15px;
  top: 50%;
  transform: translateY(-50%);
  width: 20px;
  cursor: pointer;
  height: 2em;
  margin: 0.5em 0 0.5em 0.5em;
  border-radius: 50px;
  transition: all 0.3s ease;
}

#chatInput:focus-within ~ .send-icon{
  transform: translate(.7em, -50%);
}

@media (max-width: 768px){
  .chats{
    width: 100%;
    min-height: 40em;
    border-radius: 15px;
  }

  .contacts{
    height: 100%;
    padding: 1.5em auto;
  }

  .contact{
    width: 95%;
    margin: auto;
  }

  .contact:nth-last-of-type(1){
    margin-bottom: 1em;
  }

  .chat{
    width: 100%;
    border-radius: 15px;
  }

  #chatInstance{
    border-radius: 11px;
  }
}
</style>
