<template>
  <div class="chat-container">
    <div ref="messageContainer" class="messages">
      <div
        v-for="(message, index) in messages"
        :key="index"
        :class="['message', message.sender === 'Yo' ? 'user-message' : 'bot-message']"
      >
        <img
          v-if="message.sender === 'Mia'"
          src="https://avatar.iran.liara.run/public/85"
          alt="Mia Avatar"
          class="avatar"
        />
        <img
          v-else
          src="https://avatar.iran.liara.run/public/44"
          alt="User Avatar"
          class="avatar"
        />
        <div class="message-content">
          <strong>{{ message.sender }}:</strong>
          <span v-html="formatMessage(message.text)"></span>
        </div>
      </div>
      <div v-if="isTyping" class="typing-indicator">
        <img
          src="https://avatar.iran.liara.run/public/85"
          alt="Mia Avatar"
          class="avatar"
        />
        <div class="dots">
          <div></div>
          <div></div>
          <div></div>
        </div>
      </div>
    </div>
    <div class="input-area">
      <input
        v-model="userInput"
        @keyup.enter="debouncedSendMessage"
        :placeholder="isLoading ? 'Cargando...' : 'Escribe tu mensaje...'"
        :disabled="isLoading"
      />
      <button @click="debouncedSendMessage" :disabled="isLoading">Enviar</button>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import debounce from "lodash.debounce";

export default {
  data() {
    return {
      userInput: "",
      messages: [],
      conversationHistory: [],
      isLoading: true,
      isTyping: false,
    };
  },
  async created() {
    await this.loadSystemPrompt();
    this.isLoading = false;
  },
  updated() {
    this.scrollToBottom();
  },
  methods: {
    formatMessage(text) {
      const urlPattern = /(https?:\/\/[^\s.,!?)]+(?:\.[^\s.,!?)]+)*\/?)/g;
      return text.replace(urlPattern, '<a href="$1" target="_blank">$1</a>');
    },
    async loadSystemPrompt() {
      try {
        const response = await fetch("/system-prompt.txt");
        if (!response.ok) {
          throw new Error("Error al cargar el archivo del prompt del sistema.");
        }
        const systemPrompt = await response.text();

        this.conversationHistory.push({
          role: "system",
          content: systemPrompt,
        });

        const initialMessage =
          "Hola, soy Mia. La Maestra en inteligencia artificial creada por NuHo de Caracol Televisión y LUMO Media Lab. ¿Estás interesado en conocer sobre el proceso ideación de CreActivos? o ¿quieres preguntarme sobre otros temas de inteligencia artificial? ";
        this.messages.push({
          sender: "Mia",
          text: initialMessage,
        });

        this.conversationHistory.push({
          role: "assistant",
          content: initialMessage,
        });

        this.scrollToBottom();
      } catch (error) {
        console.error("Error cargando el cerebro de Mia: ", error);
        this.messages.push({
          sender: "Mia",
          text:
            "No pude cargar el cerebro de Mia. Por favor, intenta recargar la página.",
        });
      }
    },
    scrollToBottom() {
      const container = this.$refs.messageContainer;
      container.scrollTop = container.scrollHeight;
    },
    async sendMessage() {
      if (this.userInput.trim() === "" || this.isLoading) {
        return;
      }

      const userMessage = {
        role: "user",
        content: this.userInput,
      };
      this.messages.push({
        sender: "Yo",
        text: this.userInput,
      });
      this.conversationHistory.push(userMessage);
      this.userInput = "";
      this.isLoading = true;
      this.isTyping = true;

      try {
        const completion = await axios.post(
          "https://api.openai.com/v1/chat/completions",
          {
            model: "gpt-4",
            messages: this.conversationHistory,
          },
          {
            headers: {
              Authorization: `Bearer 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`,
              "Content-Type": "application/json",
            },
          }
        );

        const aiResponse = completion.data.choices[0].message.content;
        this.messages.push({
          sender: "Mia",
          text: aiResponse,
        });

        this.conversationHistory.push({
          role: "assistant",
          content: aiResponse,
        });
      } catch (error) {
        console.error("Error al obtener la respuesta de OpenAI:", error);
        this.messages.push({
          sender: "Mia",
          text: "¡Ups! Tengo problemas ahorita, por favor intenta de nuevo más tarde.",
        });
      } finally {
        this.isLoading = false;
        this.isTyping = false;
      }
    },
    debouncedSendMessage: debounce(function () {
      this.sendMessage();
    }, 300),
  },
};
</script>

<style>
.chat-container {
  display: flex;
  flex-direction: column;
  height: 80vh;
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 10px;
  background-color: #f9f9f9;
}

.messages {
  flex: 1;
  overflow-y: auto;
  margin-bottom: 20px;
  padding-right: 10px;
  transition: all 0.3s ease;
}

.message {
  display: flex;
  align-items: flex-start;
  margin-bottom: 10px;
  word-wrap: break-word;
  padding: 10px;
  border-radius: 10px;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
  max-width: 70%;
}

.bot-message {
  background-color: #e0e0e0;
  text-align: left;
  margin-right: auto;
  border-bottom-left-radius: 0;
  opacity: 0;
  animation: fadeIn 0.5s forwards;
}

.user-message {
  background-color: #007bff;
  color: white;
  text-align: right;
  margin-left: auto;
  border-bottom-right-radius: 0;
  opacity: 0;
  animation: fadeIn 0.5s forwards;
}

.message-content {
  max-width: 100%;
}

.avatar {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  margin-right: 10px;
}

.typing-indicator {
  display: flex;
  align-items: center;
  margin-left: auto;
}

.dots {
  display: flex;
  justify-content: space-between;
  width: 20px;
}

.dots div {
  width: 6px;
  height: 6px;
  background-color: #555;
  border-radius: 50%;
  animation: blink 1s infinite;
}

@keyframes blink {
  50% {
    opacity: 0;
  }
}

@keyframes fadeIn {
  to {
    opacity: 1;
  }
}

.input-area {
  display: flex;
  align-items: center;
}

input {
  flex: 1;
  padding: 10px;
  border-radius: 5px;
  border: 1px solid #ccc;
  margin-right: 10px;
}

button {
  padding: 10px;
  border: none;
  border-radius: 5px;
  background-color: #007bff;
  color: white;
  cursor: pointer;
  transition: background-color 0.3s ease, transform 0.3s ease;
}

button:hover {
  background-color: #0056b3;
  transform: scale(1.05);
}

button:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
}

@media (max-width: 600px) {
  .chat-container {
    padding: 10px;
  }

  input {
    padding: 8px;
  }

  button {
    padding: 8px;
  }
}
</style>
