<script lang="ts">
import { ref } from 'vue'
import PouchDb, { sync } from 'pouchdb'

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
      remoteDB: 'http://localhost:5984/mydatabase',
      peoplesData: [] as docStructure[],
      //document: null as docStructure | null,
      storage: null as PouchDB.Database | null,
      selectedId: '',
      selectedPerson: null as docStructure | any,
      selFirName: '',
      selLasName: '',
      selAge: ''
      //
    }
  },

  // Methods are functions that mutate state and trigger updates.
  // They can be bound as event handlers in templates.
  methods: {
    initDatabase() {
      const $pouchDB = new PouchDb(this.remoteDB)

      if ($pouchDB) {
        console.log('Connection with PouchDB established successfully')
        console.log($pouchDB)
      } else {
        console.warn('Something went wrong')
      }

      this.storage = $pouchDB
    },

    fetchData() {
      console.log('fetchData')
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
      /*
      .then(() => {
        console.log("salut")
        this.fetchData()
      })
      .catch((error) => {
        console.log(error)
      })
        */
    },

    async getDocById(id: string) {
      const doc = await this.storage?.get(id)
      return doc
    },

    getDocByIdPromise(id: string) {
      this.storage
        ?.get(id)
        .then((doc) => {
          return doc
        })
        .catch((err) => {
          console.log(err)
        })
    },

    submit(event: Event) {
      try {
        event.preventDefault()
        this.selectedPerson.firstName = this.selFirName
        this.selectedPerson.lastName = this.selLasName
        this.selectedPerson.age = this.selAge
        this.storage?.put(this.selectedPerson)
      } catch (error: any) {
        console.log(error.message)
      }
    },

    deletePerson(event: Event) {
      try {
        event.preventDefault()
        this.storage?.remove(this.selectedPerson)
      } catch (error: any) {
        console.log(error.message)
      }
    },

    updateLocalDatabase() {
      const db = ref(this.storage).value
      if (db) {
        db.replicate.from
          .bind(this)(this.remoteDB)
          .on('complete', () => {
            console.log('on replicate complete')
            this.fetchData()
          })
          .on('error', function (error) {
            console.log('error', error)
          })
      }
    },

    updateDistantDatabase() {
      this.storage
        ?.sync(this.remoteDB)
        .on('complete', function () {
          console.log('sync complete')
        })
        .on('error', function (err) {
          console.log('sync incomplete', err)
        })
    },

    async getSelectedPerson() {
      //const a = await this.getDocById(this.selectedId);
      //const b = this.getDocByIdPromise(this.selectedId);
      this.selectedPerson = await this.getDocById(this.selectedId)
      this.selFirName = this.selectedPerson?.firstName
      this.selLasName = this.selectedPerson?.lastName
      this.selAge = this.selectedPerson?.age
    }
  },

  // Lifecycle hooks are called at different stages
  // of a component's lifecycle.
  // This function will be called when the component is mounted.
  mounted() {
    this.initDatabase()
    this.fetchData()

    console.log(this.storage)

    if (this.storage) {
      this.storage
        .sync(this.remoteDB, {
          live: true,
          retry: true
        })
        .on('change', (info) => {
          console.log("change", info)
          this.fetchData()
        })
    }
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
        <select @click="getSelectedPerson" v-model="selectedId" name="personId" id="personId">
          <option v-for="person in peoplesData" :key="person._id">
            {{ person.doc._id }}
          </option>
        </select>
      </div>
      <div>
        <label for="firstName">firstName</label>
        <input v-model="selFirName" type="text" id="firstName" name="firstName" />
      </div>
      <div>
        <label for="lastName">lastName</label>
        <input v-model="selLasName" type="text" id="lastName" name="lastName" />
      </div>
      <div>
        <label for="age">age</label>
        <input v-model="selAge" type="number" id="age" name="age" />
      </div>
      <input type="submit" name="submit" value="submit" @click="submit" />
      <input type="submit" name="delete" value="delete" @click="deletePerson" />
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
