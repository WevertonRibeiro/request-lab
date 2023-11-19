<template>
  <div class="requests-page">
    <div class="submenu">
      <div class="header">
        <h2>Requisições</h2>
        <button @click="newRequest">
          <IconSquarePlus />
        </button>
      </div>
      <div class="requests-wrapper">
        <ul class="requests-list">
          <li
            v-for="request in requests"
            :key="request.id"
            @click="changeCurrentRequest(request)"
            class="request-item"
            :class="{ active: currentRequest.id === request.id }"
          >
            <div>
              <span class="type" :class="request.type">{{ abbreviatedType(request.type) }}</span>
              <span class="title">{{ request.title }}</span>
            </div>
            <IconTrash @click.stop="deleteRequest(request.id)" />
          </li>
        </ul>
      </div>
    </div>
    <div class="content">
      <div class="header">
        <input
          v-if="onEditTitle"
          ref="edittitle"
          type="text"
          id="edittitle"
          v-model="currentRequest.title"
        />
        <h2 v-else @dblclick.prevent="editTitle">{{ currentRequest.title }}</h2>
        <button @click="saveCurrent" :class="{ mod: mutations }"><IconFloppyDisk /> Save</button>
      </div>
      <div class="request-wrapper">
        <div class="route-wrapper">
          <div class="route">
            <SelectField v-model="currentRequest.type" :items="selectItems" />
            <input type="text" name="url" id="url" v-model="currentRequest.url" />
            <button @click="request"><IconPaperPlane /></button>
          </div>
        </div>
        <div class="params" :class="{ loading: loading }">
          <TabsNavigationVue :items="tabs">
            <template v-slot:headers>
              <TextAreaField v-model="currentRequest.headers" name="headers" />
            </template>
            <template v-slot:body>
              <TextAreaField v-model="currentRequest.body" name="body" />
            </template>
          </TabsNavigationVue>
        </div>
        <div class="response-status">
          <h3>Response</h3>
          <div class="status">
            <h3>
              Status: <span>{{ status }}</span>
            </h3>
            <h3>
              Time: <span>{{ duration ? `${duration} ms` : '' }} </span>
            </h3>
          </div>
        </div>
        <div class="response">
          <pre v-html="data"></pre>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import IconSquarePlus from '../components/icons/IconSquarePlus.vue'
import IconPaperPlane from '../components/icons/IconPaperPlane.vue'
import IconFloppyDisk from '../components/icons/IconFloppyDisk.vue'
import IconTrash from '../components/icons/IconTrash.vue'

import SelectField from '../components/form/SelectField.vue'
import TextAreaField from '../components/form/TextAreaField.vue'
import TabsNavigationVue from '../components/TabsNavigation.vue'

import { ref, watch, onBeforeMount, onMounted } from 'vue'

import axios from 'axios'

const currentRequest = ref({})

const data = ref(null)
const status = ref(null)
const duration = ref(null)

const loading = ref(false)
const mutations = ref(false)
const onEditTitle = ref(false)
const edittitle = ref(null)

const requests = ref([])

const tabs = [
  {
    label: 'Headers',
    key: 'headers'
  },
  {
    label: 'Body',
    key: 'body'
  }
]

const selectItems = [
  {
    title: 'get',
    value: 'get'
  },
  {
    title: 'post',
    value: 'post'
  },
  {
    title: 'put',
    value: 'put'
  },
  {
    title: 'patch',
    value: 'patch'
  },
  {
    title: 'delete',
    value: 'delete'
  },
  {
    title: 'options',
    value: 'options'
  }
]

onBeforeMount(() => {
  const localRequests = localStorage.getItem('requestsList')
  if (localRequests) {
    requests.value = JSON.parse(localRequests)
    setCurrentRequest(requests.value[0])
  }
})

onMounted(() => {
  document.addEventListener('click', function (event) {
    if (edittitle.value && !edittitle.value.contains(event.target)) {
      onEditTitle.value = false
    }
  })
})

const editTitle = () => {
  onEditTitle.value = true
  setTimeout(() => {
    edittitle.value?.focus()
  }, 0)
}

const isJsonValide = (obj) => {
  try {
    JSON.parse(obj)
    return true
  } catch (error) {
    return false
  }
}

function formatJSON(obj, indent = 4) {
  const spaces = ' '.repeat(indent)

  if (typeof obj !== 'object' || obj === null) {
    return JSON.stringify(obj)
  }

  let formattedJSON = '{\n'

  const keys = Object.keys(obj)

  for (let i = 0; i < keys.length; i++) {
    const key = keys[i]
    const value = obj[key]

    formattedJSON += `${spaces}"${key}": ${formatJSON(value, indent + 4)}`

    if (i < keys.length - 1) {
      formattedJSON += ','
    }
    formattedJSON += '\n'
  }

  formattedJSON += spaces.slice(0, -4) + '}'
  return formattedJSON
}

const setCurrentRequest = (request) => {
  if (request) {
    currentRequest.value = {
      ...request,
      headers: isJsonValide(request.headers)
        ? formatJSON(JSON.parse(request.headers))
        : request.headers,
      body: isJsonValide(request.body) ? formatJSON(JSON.parse(request.body)) : request.body
    }
    setTimeout(() => {
      mutations.value = false
    }, 0)
  }
}
setCurrentRequest(requests.value[0])

const changeCurrentRequest = (request) => {
  if (mutations.value) {
    if (window.confirm('As alterações serão perdidas. Deseja continuar?')) {
      setCurrentRequest(request)
    }
  } else {
    setCurrentRequest(request)
  }
}

watch(
  currentRequest,
  (newValue) => {
    const indiceDoObjeto = requests.value.findIndex(
      (objeto) => objeto.id === currentRequest.value.id
    )
    if (JSON.stringify(newValue) !== JSON.stringify(requests.value[indiceDoObjeto])) {
      mutations.value = true
    } else {
      mutations.value = false
    }
  },
  { deep: true }
)

const abbreviatedType = (type) => {
  switch (type) {
    case 'delete':
      return 'del'

    case 'options':
      return 'opt'

    default:
      return type
  }
}

const colorizeJson = (json) => {
  const strJson = JSON.stringify(json, null, 2)

  return strJson
    .replace(/"(.*?)":/g, '<span data-key>$1</span>:')
    .replace(/"([^"]*)"/g, '<span data-string>"$1"</span>')
    .replace(/(\d+|\d+\.\d+|\.\d+)(,|\n)/g, '<span data-number>$1</span>$2')
    .replace(/(true|false)(,|\n)/g, '<span data-bullean>$1</span>$2')
    .replace(/(null)(,|\n)/g, '<span data-bullean>$1</span>$2')
}

const setResponse = (dataResponse, statusCode, time) => {
  data.value = colorizeJson(dataResponse)
  status.value = statusCode
  duration.value = time
}

const request = async () => {
  const startTime = new Date()
  loading.value = true
  if (['get', 'delete', 'options'].includes(currentRequest.value.type)) {
    try {
      const r = await axios[currentRequest.value.type](
        currentRequest.value.url,
        currentRequest.value.headers
      )
      setResponse(r.data, r.status, new Date() - startTime)
    } catch (error) {
      console.log(error)
      setResponse(error.message, error.response.status, new Date() - startTime)
    } finally {
      loading.value = false
    }
  } else {
    try {
      const r = await axios[currentRequest.value.type](
        currentRequest.value.url,
        JSON.parse(currentRequest.value.body),
        currentRequest.value.headers
      )
      setResponse(r.data, r.status, new Date() - startTime)
    } catch (error) {
      console.log(error)
      setResponse(error.message, error.response.status, new Date() - startTime)
    } finally {
      loading.value = false
    }
  }
}

const updateLocalData = () => {
  localStorage.setItem('requestsList', JSON.stringify(requests.value))
}

function generateUniqueId() {
  const timestamp = new Date().getTime()
  const random = Math.floor(Math.random() * 1000)
  return `${timestamp}${random}`
}

const getRequestIndex = (id) => {
  return requests.value.findIndex((element) => element.id === id)
}

const newRequest = () => {
  const newRequest = {
    id: generateUniqueId(),
    type: 'get',
    title: 'New Request',
    url: '',
    headers: '',
    body: ''
  }
  requests.value.push(newRequest)
  setCurrentRequest(newRequest)
  updateLocalData()
}

const deleteRequest = async (id) => {
  const index = getRequestIndex(id)
  if (window.confirm('Tem certeza que deseja deletar essa requisição?')) {
    requests.value.splice(index, 1)
    updateLocalData()
  }
  if (currentRequest.value.id === id) {
    setCurrentRequest(requests.value[0])
  }
}

const saveCurrent = () => {
  const index = getRequestIndex(currentRequest.value.id)
  requests.value[index] = {
    ...currentRequest.value
  }
  mutations.value = false
  updateLocalData()
}
</script>

<style lang="scss" scoped>
pre {
  :deep() {
    font-size: 15px;
    span[data-key] {
      color: #9cdcfe;
    }
    span[data-string] {
      color: #ce9178;
    }

    span[data-number] {
      color: #94ce9b;
    }

    span[data-bullean] {
      color: #569cd6;
    }
  }
}

.requests-page {
  display: flex;
  width: 100%;
  height: 100%;
  h2 {
    color: var(--color-text);
    font-size: 16px;
  }
  .header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 0 20px;
    height: 70px;
    width: 100%;
    border-bottom: solid 2px var(--color-border);
    input {
      background: transparent;
      color: var(--color-text);
      font-size: 16px;
      border: none;
      outline: 0;
      width: max-content;
      padding-right: 20px;
      font-family: 'JetBrains Mono';
    }
    button {
      background: #262626;
      border: none;
      outline: 0;
      cursor: pointer;
      color: var(--color-text);
      padding: 10px 15px;
      border-radius: 4px;
      display: flex;
      align-items: center;
      gap: 10px;
      font-size: 14px;
      font-weight: bold;
      &.mod {
        background: tomato;
        &:hover {
          background: rgb(255, 123, 99);
        }
      }
      &:hover {
        background: #363636;
      }
      svg {
        height: 16px;
        fill: var(--color-text);
      }
    }
  }
  .submenu {
    width: 400px;
    border-right: solid 2px var(--color-border);
    .header {
      display: flex;
      align-items: center;
      justify-content: space-between;
      padding: 0 20px;
      height: 70px;
      border-bottom: solid 2px var(--color-border);
      button {
        background: transparent;
        border: none;
        outline: 0;
        cursor: pointer;
        padding: 0;
        svg {
          height: 20px;
          fill: var(--color-text);
        }
      }
    }
    .requests-wrapper {
      .requests-list {
        list-style: none;
        .request-item {
          display: flex;
          align-items: center;
          justify-content: space-between;
          cursor: pointer;
          padding: 20px 15px;
          &.active {
            background: #262626;
            &:hover {
              background: #262626;
            }
          }
          &:hover {
            background: #121212;
          }
          div {
            display: flex;
            align-items: center;
            gap: 15px;
            span {
              display: block;
              font-size: 14px;
              &.type {
                min-width: 36px;
                text-align: right;
                text-transform: uppercase;
                font-weight: bold;
              }
              &.title {
              }
            }
          }
          svg {
            height: 15px;
            fill: var(--color-text);
            &:hover {
              fill: tomato;
            }
          }
        }
      }
    }
  }
  .content {
    display: flex;
    flex-direction: column;
    flex: 1;
    .request-wrapper {
      display: flex;
      position: relative;
      flex-direction: column;
      justify-content: space-between;
      flex: 1;

      &::before {
        content: '';
        display: block;
        width: 15px;
        height: 10px;
        position: absolute;
        right: 0;
        bottom: 0;
        background: var(--color-background);
        z-index: 5;
      }
      .route-wrapper {
        padding: 20px 20px 10px 19px;
        .route {
          display: flex;
          justify-content: space-between;
          border: solid 2px var(--color-border);
          border-radius: 5px;
          input,
          select {
            background: transparent;
            border: none;
            outline: 0;
            color: var(--color-text);
          }
          select {
            font-size: 16px;
            padding: 10px;
          }
          input[type='text'] {
            flex: 1;
            padding: 0 20px;
            border-left: solid 2px var(--color-border);
            letter-spacing: 1px;
          }
          button {
            width: 50px;
            background: transparent;
            border: none;
            cursor: pointer;
            border-left: solid 2px var(--color-border);
            svg {
              height: 15px;
              fill: var(--color-text);
            }
          }
        }
      }
      .params {
        border-bottom: 2px solid var(--color-border);
        flex: 1;
        padding: 0 20px 20px 20px;
        position: relative;
        overflow-x: hidden;
        &.loading {
          &::before {
            content: '';
            display: block;
            position: absolute;
            bottom: 0;
            left: 0;
            width: 400px;
            height: 2px;
            background: linear-gradient(to right, transparent, tomato, transparent);
            animation: loading 1.5s forwards infinite ease-out;
          }
        }
      }
      .response-status {
        display: flex;
        align-items: center;
        justify-content: space-between;
        padding: 10px 20px;
        h3 {
          font-size: 12px;
        }
        .status {
          display: flex;
          align-items: center;
          gap: 20px;
          span {
            color: #94ce9b;
          }
        }
      }
      .response {
        width: calc(100vw - 470px);
        height: 400px;
        overflow: auto;
        padding: 0px 20px 10px 20px;
        pre {
          white-space: pre-wrap;
        }
      }
    }
  }
}

@keyframes loading {
  from {
    left: -100%;
  }
  to {
    left: 100%;
  }
}
</style>