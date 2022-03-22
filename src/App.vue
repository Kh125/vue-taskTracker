<template>
  <div class="container">    
    <Header @toggle-add-task="toggleAddTask" title="Vue Task Tracker" :showAddTask="showAddTask"/>   
    <div v-show="showAddTask">
      <AddTask @add-task="addTask"/>
    </div>
    <Tasks @toggle-reminder="toggleReminder" @delete-task="deleteTask" :tasks="tasks" /> 
  </div>  
</template>

<script>
import AddTask from './components/AddTask';
import Header from './components/Header';
import Tasks from './components/Tasks';

export default {
  name: 'App',
  components: {
    AddTask,
    Header,
    Tasks,
  },
  data(){
    return{
      tasks:[],
      showAddTask:false
    }
  },
  methods:{
    async deleteTask(id){
      if(confirm('Are you sure?')) {
        const res = await fetch(`http://localhost:5000/tasks/${id}`,{
          method: 'DELETE'
        })
        res.status = 200 ? this.tasks = this.tasks.filter((task)=> task.id !== id) : alert('Connection Error')
      }
    },
    async toggleReminder(id){
      const toggleTask = await this.fetchTask(id);
      const updateTask = {...toggleTask, reminder:!toggleTask.reminder}      

      const res = await fetch(`http://localhost:5000/tasks/${id}`,{
        method: 'PUT',
        headers: {
          'Content-type': 'Application/json'
        },
        body: JSON.stringify(updateTask)
      })
      const data = await res.json()
      res.status === 200 ? this.tasks = this.tasks.map((task)=> task.id === id ? {...task, reminder: data.reminder} : task) : alert('Connection error')
    },
    async addTask(newTask){
      const res = await fetch('http://localhost:5000/tasks', {
        method: 'POST',
        headers:{
          'Content-type': 'application/json'
        },
        body: JSON.stringify(newTask)
      })
      const data = await res.json()
      this.tasks = [...this.tasks, data]
    },
    toggleAddTask(){
      this.showAddTask = !this.showAddTask
    },
    async fetchTasks(){
      const result = await fetch('http://localhost:5000/tasks')
      const data = await result.json()
      return data
    },
    async fetchTask(id){
      const result = await fetch(`http://localhost:5000/tasks/${id}`)
      const data = await result.json()
      return data
    }
  },
  async created(){
    this.tasks = await this.fetchTasks()
  }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@300;400&display=swap');
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}
body {
  font-family: 'Poppins', sans-serif;
}
.container {
  max-width: 500px;
  margin: 30px auto;
  overflow: auto;
  min-height: 300px;
  border: 1px solid steelblue;
  padding: 30px;
  border-radius: 5px;
}
.btn {
  display: inline-block;
  background: #000;
  color: #fff;
  border: none;
  padding: 10px 20px;
  margin: 5px;
  border-radius: 5px;
  cursor: pointer;
  text-decoration: none;
  font-size: 15px;
  font-family: inherit;
}
.btn:focus {
  outline: none;
}
.btn:active {
  transform: scale(0.98);
}
.btn-block {
  display: block;
  width: 100%;
}
</style>
