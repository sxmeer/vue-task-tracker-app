<template>
  <AddTask v-show="showAddTask" @add-task="addTask" />
  <Tasks
    @delete-task="deleteTask"
    @toggle-reminder="toggleReminder"
    :tasks="tasks"
  />
</template>

<script>
import AddTask from "../components/AddTask.vue";
import Tasks from "../components/Tasks.vue";
export default {
  name: "Home",
  props: {
    showAddTask: Boolean,
  },
  components: {
    Tasks,
    AddTask,
  },
  data() {
    return {
      tasks: [],
    };
  },
  methods: {
    async addTask(task) {
      const response = await fetch("api/tasks", {
        method: "POST",
        headers: {
          "Content-type": "application/json",
        },
        body: JSON.stringify(task),
      });
      const data = await response.json();
      this.tasks = [...this.tasks, data];
    },
    async toggleReminder(id) {
      const fetchedTask = await this.fetchTask(id);
      const updatedTask = { ...fetchedTask, reminder: !fetchedTask.reminder };
      const response = await fetch(`api/tasks/${id}`, {
        method: "PUT",
        headers: {
          "Content-type": "application/json",
        },
        body: JSON.stringify(updatedTask),
      });
      const data = await response.json();
      this.tasks = this.tasks.map((task) =>
        task.id === id ? { ...task, reminder: data.reminder } : task
      );
    },
    async deleteTask(id) {
      if (confirm("Are you sure?")) {
        const response = await fetch(`api/tasks/${id}`, {
          method: "DELETE",
        });
        response.status === 200
          ? (this.tasks = this.tasks.filter((task) => task.id !== id))
          : alert("Error deleting the task");
      }
    },
    async fetchTask(id) {
      const response = await fetch(`api/tasks/${id}`);
      const task = await response.json();
      return task;
    },
    async fetchTasks() {
      const response = await fetch(`api/tasks`);
      const tasks = await response.json();
      return tasks;
    },
  },
  async created() {
    this.tasks = await this.fetchTasks();
  },
};
</script>