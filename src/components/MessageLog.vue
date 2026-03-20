<script setup lang="ts">
import { watch, nextTick, computed } from 'vue';
// eslint-disable-next-line @typescript-eslint/no-unused-vars
import { TransitionGroup } from 'vue'; // TransitionGroup is used in template
import { storeToRefs } from 'pinia';
import { LogMessageType, uaApplication } from '../stores/UaState'

const store = uaApplication()
const { messages } = storeToRefs(store)

// Create array of reverse indices (lightweight - just numbers)
// This allows iterating backwards without copying the messages array
const reverseIndices = computed(() => {
  const msgs = messages.value
  const length = msgs.length
  return Array.from({ length }, (_, i) => length - 1 - i)
})

watch(messages, () => {
  nextTick(() => {
    const messagesDiv = document.getElementById("ua-messages-list")
    if (messagesDiv) {
      messagesDiv.scrollTop = 0
    }
  })
}, { flush: 'post' })

function formatDate(date: Date) {
  return date.toLocaleDateString('en-US', { hour12: false, hour: '2-digit', minute: '2-digit', second: '2-digit' })
}

function cleaMessages() {
  // Use the store's clearMessages method
  store.clearMessages()
}

</script>
<template>
  <div class="ua-messages-root">
    <div id="ua-messages-list" class="ua-messages-list">
      <TransitionGroup name="message" tag="div">
        <div
          v-for="reverseIndex in reverseIndices"
          :key="`msg-${messages[reverseIndex].time.getTime()}-${reverseIndex}`"
          class="ua-messages-row" :class="{'ua-messages-row-err' : messages[reverseIndex].type == LogMessageType.Error}"
        >
          <div class="ua-messages-type" >
            <img v-if="messages[reverseIndex].type == LogMessageType.Info" class="ua-message-icon" src="../assets/icon-info.svg" alt="Info"/>
            <img v-if="messages[reverseIndex].type == LogMessageType.Error" class="ua-message-icon" src="../assets/icon-warning.svg" alt="Warning"/>
          </div>
          <div class="ua-messages-time" > {{ formatDate(messages[reverseIndex].time) }} </div>
          <div class="ua-messages-message"> <pre>{{ messages[reverseIndex].details }}</pre> </div>
        </div>
      </TransitionGroup>
    </div>
    <div class="ua-messages-toolbox">
      <button
        type="button" class="btn btn-secondary btn-sm"
        style="--bs-btn-padding-y: .25rem; --bs-btn-padding-x: .5rem; --bs-btn-font-size: .75rem;"
        @click.prevent="cleaMessages">
          Clear
      </button>
    </div>
  </div>
</template>

<style lang="scss">
.ua-messages-root {
  flex: 1;
  display: flex;
  flex-direction: row;
  flex-wrap: nowrap;
}

.ua-messages-toolbox {
  flex: 0 0 auto;
  border-top: 1px solid #24262A;
  border-bottom: 1px solid #24262A;
  padding: 2px 20px;
  font-size: 8px;
  text-align: right;
}

.ua-messages-list {
  flex: 1;
  overflow: auto;
  display: flex;
  flex-direction: column;
}

// Transition animations for new messages
.message-enter-active {
  transition: all 0.3s ease-out;
}

.message-enter-from {
  opacity: 0;
  transform: translateY(-10px);
}

.message-enter-to {
  opacity: 1;
  transform: translateY(0);
}

.message-leave-active {
  transition: all 0.2s ease-in;
}

.message-leave-from {
  opacity: 1;
  transform: translateY(0);
}

.message-leave-to {
  opacity: 0;
  transform: translateY(-10px);
}

.message-move {
  transition: transform 0.3s ease;
}

.ua-messages-row {
  display: grid;
  grid-template-columns: max-content max-content 1fr;
  grid-gap: 15px;
  padding: 5px 0;
  margin: 0px;
  border-collapse: collapse;
  font-size: 12px;
  align-items: center;
  background: #030329;
  border-top: 1px solid #525252;
  &.ua-messages-row-err {
    background: #504040;
  }

  .ua-messages-time {
    white-space: nowrap;
    * {
      white-space: nowrap;
    }
  }

  .ua-messages-type {
    justify-self: center;
    white-space: nowrap;
    * {
      white-space: nowrap;
    }
  }

  .ua-messages-message {
    min-width: 0;
    word-wrap: break-word;
    overflow-wrap: break-word;
    
    pre {
      margin: 0;
      white-space: pre-wrap;
      word-wrap: break-word;
      overflow-wrap: break-word;
    }
  }

  .ua-message-icon {
    width: 15px;
    height: 15px;
  }
}

</style>
