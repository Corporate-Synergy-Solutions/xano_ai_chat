<template>
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">
  <section id="chat-section">
    <div class="container">
      <div id="chat-messages" class="chat-messages">

        <!-- <div  class="chat-messages-container" > -->
        <div ref="chatMessagesDiv" :class="['message', msg.role]" v-for="(msg, index) in conversationHistory"
          :key="index">
            <!-- {{ msg.content }}-->
          <div v-html="msg.content"></div>
          <div v-if="msg.role === 'assistant' && (msg.tool_args || msg.tool_response)" class="tool-info mt-2">
            <div v-if="msg.tool_args"><strong>Tool Args:</strong>
              <pre>{{ JSON.stringify(msg.tool_args, null, 2) }}</pre>
            </div>
            <div v-if="msg.tool_response"><strong>Tool Response:</strong>
              <pre>{{ JSON.stringify(msg.tool_response, null, 2) }}</pre>
            </div>
          </div>
        </div>
        <!-- </div> -->
        <div v-if="isTyping" class="typing-indicator-row">
          <div class="typing-indicator">Assistant is typing...</div>
        </div>



      </div>
    </div>
    <div class="chat-input-area-bg">
      <div class="chat-input-area">
        <div class="d-flex align-items-center position-relative">
          <textarea v-model="messageInput" @keydown.enter.exact.prevent="sendMessage"
            class="chat-input form-control me-2" placeholder="Type your message..."></textarea>
          <button :disabled="!messageInput.trim() || isSending" class="btn btn-primary send-button"
            @click="sendMessage">
            Send
          </button>
        </div>

      </div>
    </div>
  </section>
</template>

<script>
export default {
  data() {
    return {
      XANO_API_BASE_URL: "https://api.corporatesynergysolutions.com/api:5gkvXEsN",
      isSending: false,
      isTyping: false,
      messageInput: '',
      conversationHistory: [
        { role: 'system', content: 'Hello! I am your chatbot. How can I help you today?' }
      ],
      SYSTEM_PROMPT: "You are a helpful chatbot that interacts with the user."
    };
  },
  mounted() {
    this.scrollToBottom();
  },
  methods: {
    scrollToBottom() {
      this.$nextTick(() => {
        const div = this.$refs.chatMessagesDiv;
        if (div) div.scrollTop = div.scrollHeight;
      });
    },
    async sendMessage() {
      const userMessage = this.messageInput.trim();
      if (!userMessage) return;

      this.conversationHistory.push({ role: 'user', content: userMessage });
      this.messageInput = '';
      this.isSending = true;
      this.isTyping = true;
      this.scrollToBottom();

      try {
        const res = await fetch(`${this.XANO_API_BASE_URL}/chatbot`, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify({
            user_query: userMessage,
            conversation: this.conversationHistory,
            system_prompt: this.SYSTEM_PROMPT
          })
        });
        const data = await res.json();
        this.isTyping = false;

        if (res.ok) {
          const assistantMsg = {
            role: 'assistant',
            content: data.assistant,
            tool_args: data.tool_args,
            tool_response: data.tool_response
          };
          this.conversationHistory.push(assistantMsg);
        } else {
          this.conversationHistory.push({ role: 'system', content: `Error: ${data.message || 'Could not get response.'}` });
        }
      } catch (e) {
        this.conversationHistory.push({ role: 'system', content: 'An unexpected network error occurred.' });
      } finally {
        this.isSending = false;
        this.scrollToBottom();
      }
    }
  }
};
</script>

<style scoped>
:root {
  --primary-blue: #0d6efd;
  --light-bg: #f8f9fa;
  --border-color: #4b4c4e;
  --card-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
}

/* --- Login Page Styles --- */
.login-page-background {
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  background-color: #f7f7f7;
}

.login-card {
  background: #ffffff;
  padding: 2.5rem;
  border-radius: 8px;
  box-shadow: var(--card-shadow);
  width: 100%;
  max-width: 450px;
}


.login-title {
  text-align: center;
  font-size: 2rem;
  font-weight: 500;
  margin-bottom: 2rem;
  color: #212529;
}

.form-group {
  margin-bottom: 1.5rem;
}

.form-group label {
  display: block;
  margin-bottom: 0.5rem;
  font-weight: 500;
  color: #495057;
}

.form-control {
  display: block;
  width: 100%;
  padding: .375rem .75rem;
  font-size: 1rem;
  font-weight: 400;
  line-height: 1.5;
  color: var(--bs-body-color);
  -webkit-appearance: none;
  -moz-appearance: none;
  appearance: none;
  background-color: var(--bs-body-bg);
  background-clip: padding-box;
  border: var(--bs-border-width) solid var(--bs-border-color);
  border-radius: var(--bs-border-radius);
  transition: border-color .15s ease-in-out, box-shadow .15s ease-in-out;
}

.login-btn {
  height: 48px;
  font-weight: 500;
  font-size: 1rem;
  border-radius: 6px;
  display: inline-block;
  border: 1px solid;
  width: auto !important;
  padding: 6px 23px;
  background-color: #219be5;
  color: #fff;
}

/* --- Chat Page Styles --- */
.chat-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.75rem 1.5rem;
  background-color: #F0FFF0;
  border-bottom: 1px solid var(--border-color);
  flex-shrink: 0;
}

.chat-title {
  font-weight: 600;
  font-size: 1.25rem;
}

.message-content {
  color: #000;
  border: 1px solid;
  padding: 7px 11px;
  border: 1px solid #e9ecef;
}

.user-info {
  display: flex;
  align-items: center;
}

.chat-messages-container {
  flex-grow: 1;
  overflow-y: auto;
  padding: 1.5rem;
  background-color: #ffffff;
}

.message-row {
  display: flex;
  margin-bottom: 1rem;
}

.message-bubble {
  padding: 0.75rem 1.25rem;
  border-radius: 12px;
  max-width: 75%;
}

.message-role-system {
  justify-content: center;
}

.message-role-system .message-bubble {
  background-color: transparent;
  border: 1px solid #e0e0e0;
  color: #6c757d;
  text-align: center;
}

.message-role-user {
  justify-content: flex-end;
}

.message-role-user .message-bubble {
  background-color: var(--primary-blue);
  color: white;
}

.message-role-assistant {
  justify-content: flex-start;
}

.message-role-assistant .message-bubble {
  background-color: #f1f3f5;
  color: #212529;
  border: 1px solid #e9ecef;
}

.typing-indicator-row {
  display: flex;
  justify-content: flex-start;
}

.typing-indicator {
  font-style: italic;
  color: #6c757d;
}

.chat-input-bar {
  padding: 15px 20px;
  border-top: 1px solid #dee2e6;
  background-color: #f8f9fa;
  flex-shrink: 0;
  display: flex;
  justify-content: space-between;
}

.chat-input {
  border-radius: 20px;
  padding: 10px 15px;

}

.chat-input:focus {
  outline: none;
}

.send-button {
  height: 48px;
  width: 80px;
  border-radius: 6px;
  font-weight: 500;
  color: #fff;
  background-color: #219be5;
  text-align: center;
}

.tool-info pre {
  background: rgba(0, 0, 0, 0.05);
  padding: 10px;
  margin-top: 10px;
  border-radius: 5px;
  font-size: 0.85rem;
  white-space: pre-wrap;
}

/* Chat Section Styling */
#chat-section {
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  gap: 12px;
  position: relative;
  height: calc(100vh - 145px);
}

.chat-messages {
  flex-grow: 1;
  /* Allows this area to expand and scroll */

  padding: 20px;
  display: flex;
  /* Use flex column for messages */
  flex-direction: column;
  gap: 12px;
  /* Spacing between messages */
}

.message {
  padding: 10px 15px;
  border-radius: 18px;
  /* More rounded corners */
  max-width: 75%;
  /* Limit message width */
  word-wrap: break-word;
  line-height: 1.4;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.05);
  /* Subtle shadow */
}

.message.user {
  background-color: #007bff;
  color: white;
  align-self: flex-end;
  border-bottom-right-radius: 5px;
  /* Slightly different shape */
  margin-left: auto;
  /* Ensure right alignment */
}

.message.assistant {
  background-color: #e9ecef;
  color: #212529;
  align-self: flex-start;
  border-bottom-left-radius: 5px;
  /* Slightly different shape */
  margin-right: auto;
  /* Ensure left alignment */
}

/* System messages (like initial greeting or errors) */
.message.system {
  background-color: #f8f9fa;
  /* Very light background */
  color: #6c757d;
  /* Muted text color */
  align-self: center;
  /* Center align system messages */
  text-align: center;
  font-style: italic;
  font-size: 0.9em;
  max-width: 90%;
  border: 1px solid #dee2e6;
  /* Subtle border */
  box-shadow: none;
  /* No shadow for system messages */
}

/* Chat Input Area Styling */
.chat-input-area-bg {
  background-color: #fff;
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
}

.chat-input-area {
  width: 769px;
  margin: 0 auto;

  padding: 0.8rem;
}

.chat-input-area .form-control {
  border-radius: 20px;
  /* Rounded input */
  padding: 10px 15px;
}

.chat-input-area .send-button {
  border-radius: 20px;
  padding: 10px 20px;
  font-weight: 500;
  position: absolute;
  right: 1rem;
  top: 0;
  bottom: 0;
  margin: auto;
  height: fit-content;
}
</style>
