<template>
  <main class="main" id="app">
    <div>Last updated time: {{ animatedTime }}</div>

    <button @click="clear">Remove all</button>

    <h2>Add</h2>

    <form @submit.prevent="save">
      <label for="input-label">Label:</label>
      <input required id="input-label" name="label" v-model="label" />

      <label for="input-notes">Notes:</label>
      <textarea id="input-notes" name="notes" v-model="notes"></textarea>

      <div class="current-coords">
        <div>{{ currentLatitude }} | {{ currentLongitude }}</div>
        <a :href="mapLink">Show on map</a>
      </div>

      <button>Save</button>
    </form>

    <h2>Saved</h2>

    <ul class="saved-list" v-if="count">
      <li v-for="item in itemsArr" :key="item.key">
        <div>
          <div>{{ item.label }}</div>
          <small>{{ item.latitude }} | {{ item.longitude }}</small>
        </div>
        <button aria-label="Remove" @click="remove(item.key)">X</button>
      </li>
    </ul>
    <div v-else>Nothing here, add one!</div>
  </main>
</template>

<script>
import debounce from 'lodash.debounce'
import gsap from 'gsap'

const LOCAL_STORE_KEY = 'store'

function genTimestamp() {
  return +new Date()
}

function loadItemsFromStoreage() {
  const items = localStorage.getItem(LOCAL_STORE_KEY)
  return items ? JSON.parse(items) : {}
}

export default {
  data() {
    return {
      label: '',
      notes: '',
      currentLatitude: 0,
      currentLongitude: 0,
      lastUpdateTime: 0,
      tweenTime: 0,
      items: {}
    }
  },
  computed: {
    itemsArr() {
      return Object.values(this.items)
    },

    count() {
      return this.itemsArr.length
    },

    animatedTime() {
      return this.tweenTime.toFixed(0)
    },

    mapLink() {
      return `https://www.google.com/maps/search/?api=1&query=${this.currentLatitude},${this.currentLongitude}`
    }
  },
  watch: {
    items(newItems) {
      localStorage.setItem(LOCAL_STORE_KEY, JSON.stringify(newItems))
    },

    lastUpdateTime(newValue) {
      gsap.to(this.$data, 0.5, { tweenTime: newValue })
    }
  },
  methods: {
    save() {
      const timestamp = genTimestamp()
      const item = {
        key: timestamp,
        timestamp,
        label: this.label,
        notes: this.notes,
        latitude: this.currentLatitude,
        longitude: this.currentLongitude
      }

      this.items = {
        [timestamp]: item,
        ...this.items
      }

      this.label = ''
      this.notes = ''
    },

    remove(key) {
      const copy = { ...this.items }
      delete copy[key]
      this.items = copy
    },

    updateCurrentPos(pos) {
      this.lastUpdateTime = genTimestamp()
      this.currentLatitude = pos.coords.latitude
      this.currentLongitude = pos.coords.longitude
    },

    clearForm() {
      this.label = ''
      this.notes = ''
    },

    clear() {
      localStorage.removeItem(LOCAL_STORE_KEY)
      this.items = {}
    }
  },
  mounted() {
    this.items = loadItemsFromStoreage()

    navigator.geolocation.watchPosition(debounce(this.updateCurrentPos, 500))
  }
}
</script>

<style>
body {
  margin: 0;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

form {
  margin-top: 20px;
  display: flex;
  flex-direction: column;
  width: 300px;
  max-width: 100%;
}

input,
textarea {
  margin-bottom: 10px;
}

.hidden {
  display: none;
}

.saved-list {
  list-style: none;
  padding: 0;
}

.saved-list li {
  padding: 10px;
  margin: 15px 0;
  border: 1px solid #ddd;
  display: flex;
  justify-content: space-between;
}

.current-coords {
  margin-bottom: 15px;
  text-align: center;
}
</style>
