<script lang="ts">
import { ref } from 'vue'
import PouchDb from 'pouchdb'

declare interface docStructure {
  _id: string
  firstName: string
  lastName: string
  age: number
  hobbys: string[]
}

export default {
  // Properties returned from data() become reactive state
  // and will be exposed on `this`.
  data() {
    return {
      peoplesData: [] as docStructure[],
      document: null as docStructure | null,
      storage: null as PouchDB.Database | null,
      selected: ""
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
      const storage = ref(this.storage)
      const self = this
      if (storage.value) {
        storage.value
          .allDocs({
            include_docs: true,
            attachments: true
          })
          .then(
            function (result: any) {
              console.log('fetchData success', result)
              self.peoplesData = result.rows
            }.bind(this)
          )
          .catch(function (error: any) {
            console.log('fetchData error', error)
          })
      }
    },

    createFakeAssDocument() {
      this.createDocument('Fake', 'Ass', 69, ['Fake', 'Ass', 'Document'])
      this.fetchData()
    },

    createDocument(firstName: string, lastName: string, age: number, hobbys: string[]) {
      const doc: docStructure = {
        _id: '',
        firstName: '',
        lastName: '',
        age: 0,
        hobbys: []
      }

      doc.firstName = firstName
      doc.lastName = lastName
      doc.age = age
      hobbys.forEach((element: string) => {
        doc.hobbys.push(element)
      })
      doc._id = new Date().toISOString()

      this.storage?.put(doc)
    },

    getDocById(id: string) {
      this.storage?.get(id).then(function (doc) {
        return doc
      })
    },

    stupid(event:Event){
      try{
        event.preventDefault()
        console.log(event)
      }catch(error:any){
        console.log(error.message)
      }
    },

    getSelectedPerson(){
      

    }
  },

  // Lifecycle hooks are called at different stages
  // of a component's lifecycle.
  // This function will be called when the component is mounted.
  mounted() {
    this.initDatabase()
    this.fetchData()
    console.log(this.peoplesData)
  }
}
</script>

<template>
  <div id="dbControl">
    <h1>Nombre de personnes: {{ peoplesData.length }}</h1>
    <button @click="createFakeAssDocument">Create Fake Ass Document</button>
    <form>
      <div>
        <label for="personId">Id</label>
        <select v-model="selected" name="personId" id="personId">
          <option v-for="person in peoplesData" :key="person._id" value="person._id" >
            {{ person.doc._id }}
          </option>
        </select>
      </div>
      <div>
        <label for="firstName">firstName</label>
        <input type="text" id="firstName" name="firstName"/>
      </div>
      <div>
        <label for="lastName">lastName</label>
        <input type="text" id="lastName" name="lastName" />
      </div>
      <div>
        <label for="age">age</label>
        <input type="number" id="age" name="age" />
      </div>
      <input type="submit" name="submit" value="submit" @click="stupid"/>
    </form>

    <div id="personCard">
      <div class="ucfirst" v-for="person in peoplesData" :key="person._id">
        <p>Id : {{ person.doc._id }}</p>
        <p>Prénom : {{ person.doc.firstName }}</p>
        <p>Nom : {{ person.doc.lastName }}</p>
        <p>Age : {{ person.doc.age }}</p>
        <div>
          <p>Hobbys :</p>

          <ul>
            <li v-for="(hobby, index) in person.doc.hobbys" :key="index">
              {{ hobby }}
            </li>
          </ul>
        </div>
      </div>
    </div>
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

#dbControl {
  display: flex;
  width: 100%;
  flex-direction: column;
}
#personCard {
  display: flex;
  flex-wrap: wrap;
}
.ucfirst {
  width: 200px;
  margin: 10px;
  padding: 10px;
  border: 1px solid black;
}
button {
  width: 100px;
}
form div {
  display: flex;
  flex-direction: column;
  flex-wrap: wrap;
  align-items: flex-start;
}
</style>
