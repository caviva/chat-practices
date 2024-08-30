<template>
  <div class="chat-container">
    <div ref="messageContainer" class="messages">
      <div v-for="(message, index) in messages" :key="index" class="message">
        <strong>{{ message.sender }}:</strong> {{ message.text }}
      </div>
    </div>
    <div class="input-area">
      <input
        v-model="userInput"
        @keyup.enter="sendMessage"
        :placeholder="isLoading ? 'Cargando...' : 'Escribe tu mensaje...'"
        :disabled="isLoading"
      />
      <button @click="sendMessage" :disabled="isLoading">Enviar</button>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      userInput: "",
      messages: [],
      conversationHistory: [],
      isLoading: true, // Estado de carga
    };
  },
  async created() {
    try {
      // Cargar el contenido del prompt desde el archivo .txt en el directorio public
      const response = await fetch("/system-prompt.txt");
      if (!response.ok) {
        throw new Error("Error al cargar el archivo del prompt del sistema.");
      }
      const systemPrompt = await response.text();

      // Agregar el prompt del sistema al historial de la conversación
      this.conversationHistory.push({
        role: "system",
        content: systemPrompt,
      });

      // Mia inicia la conversación
      const initialMessage =
        "Hola, soy Mia. La Maestra en inteligencia artificial creada por NuHo de Caracol Televisión y LUMO Media Lab. ¿Estás interesado en conocer sobre el proceso ideación de CreActivos? o ¿quieres preguntarme sobre otros temas de inteligencia artificial? ";
      this.messages.push({
        sender: "Mia",
        text: initialMessage,
      });

      // Agregar el mensaje de Mia al historial de conversación
      this.conversationHistory.push({
        role: "assistant",
        content: initialMessage,
      });

      this.isLoading = false; // Desactivar el estado de carga

      // Desplazarse hacia abajo después de que Mia envíe el primer mensaje
      this.scrollToBottom();
    } catch (error) {
      console.error("Error cargando el prompt del sistema:", error);
      this.messages.push({
        sender: "Mia",
        text:
          "No pude cargar la información inicial. Por favor, intenta recargar la página.",
      });
      this.isLoading = false;
    }
  },
  updated() {
    this.scrollToBottom(); // Desplazar hacia abajo cuando se actualiza el componente
  },
  methods: {
    scrollToBottom() {
      const container = this.$refs.messageContainer;
      container.scrollTop = container.scrollHeight;
    },
    async sendMessage() {
      if (this.userInput.trim() === "" || this.isLoading) {
        return;
      }

      // Añadir el mensaje del usuario al chat
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

      try {
        // Llamar a la API de OpenAI con el formato para chat completions
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

        // Añadir la respuesta del bot al chat
        const aiResponse = completion.data.choices[0].message.content;
        this.messages.push({
          sender: "Mia",
          text: aiResponse,
        });

        // Actualizar el historial de la conversación con la respuesta del asistente
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
      }
    },
  },
};
</script>

<style>
.chat-container {
  display: flex;
  flex-direction: column;
  height: 80vh; /* Ocupa toda la altura de la pantalla */
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 10px;
  background-color: #f9f9f9;
}

.messages {
  flex: 1; /* Toma todo el espacio disponible */
  overflow-y: auto;
  margin-bottom: 20px;
  padding-right: 10px; /* Añadir padding para scroll */
}

.message {
  margin-bottom: 10px;
  word-wrap: break-word;
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
}

button:hover {
  background-color: #0056b3;
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
