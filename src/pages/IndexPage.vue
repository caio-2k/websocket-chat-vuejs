<template>
  <section class="m-5">
    <div class="w-full text-2xl text-center">Chat - PHP + Websocket (Ratchet) + Vue.JS</div>
    <div ref="messageContainer" id="messages"
      class="w-full lg:w-1/2 bg-white p-5 border border-gray-300 h-80 overflow-y-auto my-2 mx-auto text-base">
      <div class="basis-0 grow max-w-full">
        <ul class="list-none">
          <li v-for="message in state.messages" :key="message">
            <span class="text-gray-500" v-if="message.date">[{{ message.date }}]</span>
            <span class="font-bold px-1" v-if="message.user">{{ message.user }}:</span>
            <span class="text" :style="{ color: message.color }">
              {{ message.text }}
            </span>
          </li>
        </ul>
      </div>
    </div>
    <div class="w-full flex flex-row justify-center gap-5">

      <div class="w-44">
        <q-input outlined v-model="state.user" label="Usuário" dense />
      </div>

      <div class="w-96">
        <q-input outlined v-model="state.text" label="Mensagem" dense @keyup.enter="sendMessage" />
      </div>

    </div>
  </section>
</template>

<script setup>
import { reactive, onMounted, ref } from 'vue';

onMounted(() => {
  WsConnect();
});

const state = reactive({
  user: 'Anônimo',
  text: null,
  messages: [],
  ws: new WebSocket('ws://localhost:8080'),
});

const messageContainer = ref(null);

const WsConnect = (onOpen) => {
  // Evento que será chamado ao abrir conexão websocket
  state.ws.onopen = () => {
    addSuccessNotification('Conectado');

    // Se houver método de retorno
    if (onOpen) {
      onOpen();
    }
  };

  // Evento que será chamado quando houver erro na conexão
  state.ws.onerror = () => {
    addErrorNotification('Não foi possível conectar-se ao servidor');
  };

  // Evento que será chamado quando recebido dados do servidor
  state.ws.onmessage = (e) => {
    addMessage(JSON.parse(e.data));
  };
};

// Método responsável por adicionar uma mensagem de usuário
const addMessage = (data) => {
  state.messages.push(data);
  scrollDown();
};

const addSuccessNotification = (text) => {
  addMessage({ color: 'green', text });
};

const addErrorNotification = (text) => {
  addMessage({ color: 'red', text });
};

// Método responsável por enviar uma mensagem
// eslint-disable-next-line no-unused-vars
const sendMessage = () => {
  // Se não houver o texto da mensagem ou o nome de usuário
  if (!state.text || !state.user) {
    // Saindo do método
    return;
  }

  // Se a conexão não estiver aberta
  if (state.ws.readyState !== state.ws.OPEN) {
    // Exibindo notificação de erro
    addErrorNotification('Problemas na conexão. Tentando reconectar...');

    // Tentando conectar novamente e caso teha sucesso envia mensagem novamente
    WsConnect(() => {
      sendMessage();
    });
  }

  // Envia os dados para o servidor através do websocket
  state.ws.send(JSON.stringify({
    user: state.user,
    text: state.text,
  }));

  // Limpando o texto da mensagem
  state.text = null;
};

// Método responsável por "rolar" a scroll do chat para baixo
const scrollDown = () => {
  messageContainer.value.scrollTop = messageContainer.value.scrollHeight;
};

</script>

<style lang="css" scoped>
/* #messages span.name {
  font-weight: bold;
} */

/* #messages span.date {
  color: #999;
} */
</style>
