<template>
  <div class="container">

    <app-alert :alert="alert" @close="alert=null"></app-alert>
    <form class="card" @submit.prevent="createPerson">
      <h2>Работа с базой данных</h2>

    <div class="card">
      <div class="form-control">
        <label for="name">Введите имя</label>
        <input type="text" id="name" v-model.trim="name">
      </div>
      <button class="btn primary" :disabled="name.length === 0">Добавить человека</button>
    </div>
    </form>
    <app-loader v-if="loader"></app-loader>
    <app-people-list
        :people="people"
        @load="loadPeople"
        @remove="removePerson"
        v-else
    ></app-people-list>
  </div>

</template>

<script>

import AppPeopleList from "@/AppPeopleList";
import AppAlert from "@/AppAlert";
import AppLoader from "@/AppLoader";
import axios from 'axios'

export default {
  data() {
    return {
      name: '',
      people: [],
      alert: null,
      loader: false
    }
  },
  mounted() {
    this.loadPeople()
  },
  methods: {

    async createPerson() {
      const response = await fetch('https://vue-http-ac993-default-rtdb.firebaseio.com/people.json', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          firstName: this.name
        })
      })

      const firebaseData = await response.json()
      this.people.push({
        firstName: this.name,
        id: firebaseData.name
      })
      this.name = ''


    },
    async loadPeople() {
      try {
        this.loader = true
        const {data} = await axios.get('https://vue-http-ac993-default-rtdb.firebaseio.com/people.json')
        if (!data) {
          throw new Error('Список людей пуст')
        }
        this.people = Object.keys(data).map(key => {
          return {
            id: key,
            ...data[key]
          }
        })
        this.loader = false
      }catch (e) {
        this.alert = {
          type: 'danger',
          title: 'Ошибка!',
          text: e.message
        }
        this.loader = false
      }


    },
    async removePerson(id) {
      try {
        const name = this.people.find(person => person.id === id).firstName
        await axios.delete(`https://vue-http-ac993-default-rtdb.firebaseio.com/people/${id}.json`)
        this.people = this.people.filter(person => person.id !== id)
        this.alert = {
          type: 'primary',
          title: 'Успешно!',
          text: `Пользователь "${name}" был удален`
        }
      } catch (e) {
        this.alert = {
          type: 'danger',
          title: 'Ошибка!',
          text: 'Пользователь не удален'
        }
      }

    },

  },
  components: {
    AppPeopleList,
    AppAlert,
    AppLoader
  }

}
</script>

<style>

</style>
