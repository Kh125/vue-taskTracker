<template>    
    <AddTask v-show="showAddTask" @add-task="addTask"/>
    <Tasks @toggle-reminder="toggleReminder" @delete-task="deleteTask" :tasks="tasks" />
</template>

<script>
import Tasks from '../components/Tasks';
import AddTask from '../components/AddTask';
export default {
    name:'Home',
    props:{
        showAddTask:Boolean
    },
    components:{
        Tasks,
        AddTask
    },
    data(){
        return{
            tasks:[]
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