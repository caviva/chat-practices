<template>
  <div class="chat-container">
    <div ref="messageContainer" class="messages">
      <div
        v-for="(message, index) in messages"
        :key="index"
        :class="['message', message.sender === 'Me' ? 'user-message' : 'bot-message']"
      >
        <img
          v-if="message.sender === 'UFSCar'"
          src="https://api.dicebear.com/9.x/bottts/svg"
          alt="UFSCar Avatar"
          class="avatar"
        />
        <div class="message-content">
          <strong>{{ message.sender }}:</strong>
          <span v-html="formatMessage(message.text)"></span>
        </div>
      </div>
      <div v-if="isTyping" class="typing-indicator">
        <img
          src="https://api.dicebear.com/9.x/bottts/svg"
          alt="UFSCar Avatar"
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
        :placeholder="isLoading ? 'Loading...' : 'Type your message...'"
        :disabled="isLoading"
      />
      <button @click="debouncedSendMessage" :disabled="isLoading">Send</button>
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
      let formattedText = text.replace(urlPattern, '<a href="$1" target="_blank">$1</a>');

      formattedText = formattedText
        .replace(/(^|\n)- (.*?)(\n|$)/g, "$1<ul><li>$2</li></ul>$3")
        .replace(/(^|\n)\* (.*?)(\n|$)/g, "$1<ul><li>$2</li></ul>$3");

      formattedText = formattedText.replace(
        /(^|\n)[0-9]+\. (.*?)(\n|$)/g,
        "$1<ol><li>$2</li></ol>$3"
      );

      return formattedText;
    },
    async loadSystemPrompt() {
      try {
        const response = await fetch("/system-prompt.txt");
        if (!response.ok) {
          throw new Error("Error loading the system prompt file.");
        }
        const systemPrompt = await response.text();

        this.conversationHistory.push({
          role: "system",
          content: systemPrompt,
        });

        const initialMessage =
          "Hello! I'm a virtual assistant here to guide you through best practices in software testing. I'm ready to discuss and explore the topic with you.";
        this.messages.push({
          sender: "UFSCar",
          text: initialMessage,
        });

        this.conversationHistory.push({
          role: "assistant",
          content: initialMessage,
        });

        this.scrollToBottom();
      } catch (error) {
        console.error("Error loading UFSCar's brain: ", error);
        this.messages.push({
          sender: "UFSCar",
          text: "I couldn't load UFSCar's brain. Please try reloading the page.",
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
        sender: "Me",
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
          sender: "UFSCar",
          text: aiResponse,
        });

        this.conversationHistory.push({
          role: "assistant",
          content: aiResponse,
        });
      } catch (error) {
        console.error("Error getting response from OpenAI:", error);
        this.messages.push({
          sender: "UFSCar",
          text: "Oops! I'm having trouble right now, please try again later.",
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

.message-content ul,
.message-content ol {
  margin: 0;
  padding-left: 20px;
}

.message-content li {
  margin-bottom: 5px;
}

.message-content a {
  color: #007bff;
  text-decoration: underline;
}

.message-content a:hover {
  text-decoration: none;
}
</style>
