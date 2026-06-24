<template>
  <div class="tasks_container">
    <!-- Formulario para crear tarea -->
    <div class="add_task">
      <h2>Nueva Tarea</h2>
      <div class="form-group">
        <label>Título</label>
        <input type="text" v-model="title" placeholder="Título" />
      </div>
      <div class="form-group">
        <label>Descripción</label>
        <textarea v-model="description" placeholder="Descripción"></textarea>
      </div>
      <button @click="submitForm">Agregar Tarea</button>
    </div>

    <!-- Lista de tareas -->
    <div class="tasks_content">
      <h1>Tareas</h1>
      <ul class="tasks_list">
        <li v-for="task in tasks" :key="task.id">
          <h2>{{ task.title }}</h2>
          <p>{{ task.description }}</p>
          <button @click="toggleTask(task)">
            {{ task.completed ? 'Deshacer' : 'Completar' }}
          </button>
          <button @click="deleteTask(task)">Eliminar</button>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      tasks: [],
      title: '',
      description: ''
    }
  },
  methods: {
    async getData() {
      try {
        const response = await this.$http.get('http://localhost:8000/api/tasks/')
        this.tasks = response.data
      } catch (error) {
        console.log(error)
      }
    },
    async submitForm() {
      try {
        const response = await this.$http.post('http://localhost:8000/api/tasks/', {
          title: this.title,
          description: this.description,
          completed: false
        })
        this.tasks.push(response.data)
        this.title = ''
        this.description = ''
      } catch (error) {
        console.log(error)
      }
    },
    async toggleTask(task) {
      try {
        const response = await this.$http.put(`http://localhost:8000/api/tasks/${task.id}/`, {
          completed: !task.completed,
          title: task.title,
          description: task.description
        })
        let taskIndex = this.tasks.findIndex(t => t.id === task.id)
        this.tasks = this.tasks.map((t) => {
          if (this.tasks.findIndex(x => x.id === t.id) === taskIndex) {
            return response.data
          }
          return t
        })
      } catch (error) {
        console.log(error)
      }
    },
    async deleteTask(task) {
      let confirmation = confirm('¿Quieres eliminar esta tarea?')
      if (confirmation) {
        try {
          await this.$http.delete(`http://localhost:8000/api/tasks/${task.id}/`)
          this.getData()
        } catch (error) {
          console.log(error)
        }
      }
    }
  },
  created() {
    this.getData()
  }
}
</script>