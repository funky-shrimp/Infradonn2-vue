<script lang="ts">
import PouchDb from 'pouchdb'

declare interface docStructure {
  _id: string
  firstName: string
  lastName: string
  age: number
  hobbys: []
}

export default {
  // Properties returned from data() become reactive state
  // and will be exposed on `this`.
  data() {
    return {
      storage: null as PouchDB.Database | null
    }
  },

  // Methods are functions that mutate state and trigger updates.
  // They can be bound as event handlers in templates.
  methods: {
    initDatabase() {
      const $pouchDB = new PouchDb('http://localhost:5984/mydatabase')

      if ($pouchDB) {
        console.log('Connection with PouchDB established successfully')
        console.log($pouchDB)
      } else {
        console.warn('Something went wrong')
      }

      this.storage = $pouchDB
    },

    fetchData() {
      this.storage?.allDocs().then(function (doc) {
        console.log(doc)
      })
    },

    createDocument() {
      const doc: docStructure = {
        _id: "",
        firstName: "",
        lastName: "",
        age: 0,
        hobbys: []
      }

      

    }
  },

  // Lifecycle hooks are called at different stages
  // of a component's lifecycle.
  // This function will be called when the component is mounted.
  mounted() {
    this.initDatabase()
    this.fetchData()
  }
}
</script>

<template>
  <div class="couchDB">
    <h1>CouchDB</h1>
    <p>{{ storage }}</p>
  </div>
</template>

<style>
@media (min-width: 1024px) {
  .couchDB {
    min-height: 100vh;
    display: flex;
    align-items: center;
  }
}
</style>
