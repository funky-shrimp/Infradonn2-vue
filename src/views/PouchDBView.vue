<script lang="ts">
import { ref } from 'vue'
import PouchDB from 'pouchdb'
import findPlugin from 'pouchdb-find'
PouchDB.plugin(findPlugin)

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
      search: '',
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
      const $pouchDB = new PouchDB('myLocalDb')

      $pouchDB.replicate.from(this.remoteDB).on('complete', () => {
        console.log('Connection with PouchDB established successfully')
      })

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
              //Permet de récupéré que les informations utilisateurs
              // contenues dans la propriété doc
              self.peoplesData = result.rows.map((row: any) => row.doc)
              //Enlève les index
              self.peoplesData = self.peoplesData.filter((row) => !row.hasOwnProperty('language'))
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
      let doc = null
      try {
        doc = await this.storage?.get(id)
      } catch (error: any) {
        return null
      }

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
        this.selectedPerson = null
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

    searchIndex(event: Event) {
      event.preventDefault()
      this.storage
        ?.find({
          selector: { firstName: { $regex: new RegExp(this.search, 'i') } }
        })
        .then((res: any) => {
          this.peoplesData = res.docs.map((doc: any) => ({
            ...doc, // Conserver tous les champs existants
            _id: doc._id // Ajouter le champ 'id' égal à '_id'
          }))
        })
        .catch((err: any) => {
          console.log(err)
        })
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

    if (this.storage) {
      this.storage
        .sync(this.remoteDB, {
          live: true,
          retry: true
        })
        .on('change', (info) => {
          console.log('change', info)
          this.fetchData()
        })
        .on('error', function (err) {
          console.log(err)
        })

      this.storage
        .createIndex({
          index: {
            name: 'firstNameIndex',
            fields: ['firstName']
          }
        })
        .then(function (result) {
          console.log('index created', result)
        })
        .catch(function (err) {
          console.log(err)
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
      <input type="text" v-model="search" placeholder="Search first name" />
      <input type="submit" value="Search" @click="searchIndex" />
    </form>

    <form>
      <div>
        <label for="personId">Id</label>
        <select @click="getSelectedPerson" v-model="selectedId" name="personId" id="personId">
          <option v-for="person in peoplesData" :key="person._id">
            {{ person._id }}
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
        <p>Id : {{ person._id }}</p>
        <p>Prénom : {{ person.firstName }}</p>
        <p>Nom : {{ person.lastName }}</p>
        <p>Age : {{ person.age }}</p>
        <div>
          <p>Hobbys :</p>

          <ul>
            <li v-for="(hobby, index) in person.hobbys" :key="index">
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
