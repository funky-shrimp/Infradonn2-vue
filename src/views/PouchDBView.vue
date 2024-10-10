<script lang="ts">

export default {
  // Properties returned from data() become reactive state
  // and will be exposed on `this`.
  data() {
    return {
      pouchDB: null
    }
  },

  // Methods are functions that mutate state and trigger updates.
  // They can be bound as event handlers in templates.
  methods: {
    initDatabase() {
      this.pouchDB = $pouchDB.name
    }
  },

  // Lifecycle hooks are called at different stages
  // of a component's lifecycle.
  // This function will be called when the component is mounted.
  mounted() {
    this.initDatabase()
  }
}
</script>

<template>
  <div class="couchDB">
    <h1>CouchDB</h1>
    <p>{{ pouchDB }}</p>
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
