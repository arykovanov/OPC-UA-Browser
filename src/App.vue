<script setup lang="ts">
import AuthModal from './components/AuthModal.vue'
import IconRealtimeLogic from './components/icons/IconRealtimeLogic.vue'
import IconSettings from './components/icons/IconSettings.vue'
import MessageLog from './components/MessageLog.vue'
import UaNodeTree from './components/UaNodeTree.vue'
import UaAttributes from './components/UaAttributes.vue'

import { uaApplication, AttributeValueType } from './stores/UaState'
import { onMounted, onUnmounted, provide, ref } from 'vue'

function connectServer(evt: CustomEvent) {
  uaApplication().connect(evt.detail)
}

const attributes = ref<AttributeValueType[]>([])

async function selectNode(nodeId: string) {
  const attrs = await uaApplication().readAttributes(nodeId)
  if (attrs) {
    attributes.value = attrs
  }
}

provide('selectNode', selectNode)

onMounted(() => {
  const auth = document.getElementById('show-settings-button')
  if (auth) {
    setTimeout(() => {
      auth.click()
    }, 100)
  }

  // Read time on server preiodically to keep OPCUA session alive
  const readPeriodMs: number = 60000
  const interval = setInterval(async () => {
    const app = uaApplication()
    if (!app.connected) {
      return
    }

    try {
      await app.readAttributes("i=2258")

      // TODO: Display server time and server information on a page.
      // const attrs: any = await app.readAttributes("i=2258")
      // const time = attrs.find((val: any) => {
      //   return val.name == "Value"
      // })

      // if (time) {
      //   const date = new Date(time.value.dateTime * 1000)
      //   serverTime.value = date.toDateString() + " " + date.toLocaleTimeString()
      // }
    }
    catch (e) {
      console.error(e);
    }

  }, readPeriodMs)

  onUnmounted(()=>{
    clearInterval(interval)
  })

})

</script>

<template>
  <div class="opcua-browser-app" @endpoint.prevent="connectServer" >
    <AuthModal id="auth-dialog" />

    <header>
      <IconRealtimeLogic class="realtimelogic-header-logo" />
      <div class="opcua-header">
        <button
          id="show-settings-button"
          type="button"
          style="margin: 1em"
          class="btn btn-sm"
          data-bs-toggle="modal"
          data-bs-target="#auth-dialog"
        >
          <IconSettings />
        </button>
      </div>
    </header>

    <aside>
      <UaNodeTree />
    </aside>
    <main>
      <UaAttributes :attributes="attributes" />
    </main>

    <footer>
      <MessageLog />
    </footer>
  </div>
</template>


<style lang="scss">

.opcua-browser-app {
  display: flex;
  flex-direction: column;
  header {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    width: 100%;
    background: #000000;
    display: flex;
    align-items: center;
    height: 4em;
    z-index: 1000;

    .realtimelogic-header-logo {
      height: 100%;
      display: flex;
      align-items: center;
      padding: 0.5em;
    }

    .opcua-header {
      flex: 1;
      height: 5vh;
      display: flex;
      justify-content: flex-end;
      align-items: center;
    }
  }
  
  aside {
    position: fixed;
    display: flex;
    flex-direction: column;
    background: #1D1E22;
    left: 0;
    bottom: 0;
    width: 300px;
    top: 4em;
    border-right: 1px solid #262323;
    overflow: scroll;
  }

  main {
    background: #1D1E22;
    position: fixed;
    left: 301px;
    top: 4em;
    bottom: 150px;
    right: 0;
    overflow: auto;
    padding: 4px;
  }

  footer {
    position: fixed;
    display: flex;
    left: 301px;
    bottom: 0;
    right: 0;
    height: 150px;
    padding: 4px;
  }
}

</style>