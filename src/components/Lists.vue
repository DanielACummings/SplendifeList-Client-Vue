<template>
  <div>
    <h1>Lists</h1>
    <ul>
      <li v-for="list in lists" :key="list.id">
        <span :class="{ 'crossed-out': list.crossedOut }">{{ list.name }}</span>
        <ul>
          <li v-for="item in list.items" :key="item.id">
            <span :class="{ 'crossed-out': item.crossedOut }">{{ item.name }}</span>
          </li>
        </ul>
      </li>
    </ul>
    <form @submit.prevent="createList">
      <input type="text" v-model="newListName" placeholder="Enter new list name">
      <button type="submit">Create List</button>
    </form>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      lists: [],
      newListName: '',
      newItemName: ''
    }
  },
  mounted() {
    this.loadLists()
  },
  methods: {
    loadLists() {
      axios.get('lists')
        .then(response => {
          this.lists = response.data
          // Now that lists are loaded, fetch items for each list
          this.lists.forEach(list => this.getItemsByList(list.id));
        })
        .catch(error => {
          console.error(error)
        })
    },
    createList() {
      if (this.newListName.trim() === '') {
        alert('Please enter a list name')
        return
      }
      if (this.lists.find(list => list.name === this.newListName)) {
        alert('List name already in use')
        return
      }
      // TODO: Remove hard-coded test user ID once authentication code is added
      axios.post('lists', { name: this.newListName, user: 1})
        .then(() => {
          this.loadLists()
          this.newListName = ''
        })
        .catch(error => {
          console.error(error)
        })
    },
    getItemsByList(listId) {
      // TODO: Remove hard-coded test user ID once authentication code is added
      axios.get(`lists/${listId}/items?userId=1`)
        .then(response => {
          const list = this.lists.find(list => list.id === listId)
          list.items = response.data
        })
        .catch(error => {
          console.error(error)
        })
    },
    
    addItem(listId) {
      if (this.newItemName.trim() === '') {
        alert('Please enter a list item name')
        return
      }
      axios.post(`lists/${listId}/items`, { name: this.newItemName })
        .then(() => {
          this.loadLists()
          this.newItemName = ''
        })
        .catch(error => {
          console.error(error)
        })
    }
  }
}
</script>

<style>
.crossed-out {
  text-decoration: line-through;
}
.add-item-form {
  display: flex;
  align-items: center;
  margin-top: 0.5rem;
}
</style>
