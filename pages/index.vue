<template lang="pug">
.container
  user-banner
  div
    logo
    h1.title frourio-todo-app
    div(v-if="!$fetchState.pending")
      form(@submit.prevent="createTask")
        input(v-model="newLabel" type="text")
        input(type="submit" value="ADD")
      ul.tasks
        li(v-for="task in tasks" :key="task.id")
          label
            input(type="checkbox" :checked="task.done" @change="toggleDone(task)")
            span {{ task.label }}
          input(type="button" value="DELETE" style="float: right" @click="deleteTask(task)")
</template>

<script lang="ts">
import Vue from 'vue'
import { Task } from '$prisma/client'

export default Vue.extend({
  async fetch() {
    await this.fetchTasks()
  },
  data() {
    return {
      tasks: [] as Task[],
      newLabel: ''
    }
  },
  methods: {
    async fetchTasks() {
      this.tasks = await this.$api.tasks.$get()
    },
    async createTask() {
      if (!this.newLabel) return

      await this.$api.tasks.post({ body: { label: this.newLabel } })
      this.newLabel = ''
      await this.fetchTasks()
    },
    async toggleDone(task: Task) {
      await this.$api.tasks
        ._taskId(task.id)
        .patch({ body: { done: !task.done } })
      await this.fetchTasks()
    },
    async deleteTask(task: Task) {
      await this.$api.tasks._taskId(task.id).delete()
      await this.fetchTasks()
    }
  }
})
</script>

<style lang="scss" scoped>
.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.title {
  font-family: 'Quicksand', 'Source Sans Pro', -apple-system, BlinkMacSystemFont,
    'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  display: block;
  font-weight: 300;
  font-size: 100px;
  color: #35495e;
  letter-spacing: 1px;
}

.tasks {
  width: 300px;
  padding: 0;
  margin: 20px auto 0;
  list-style-type: none;
  text-align: left;
}

.tasks > li {
  margin-top: 10px;
  border-bottom: 1px solid #eee;
}
</style>
