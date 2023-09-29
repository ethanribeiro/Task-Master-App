<template>
  <h1>TASK MASTER</h1>
  <div id="app">
    <h1>Your Tasks</h1>
    <task-form @task-added="addTask"></task-form>
    <h2 id="list-summary" ref="listSummary" tabindex="-1">{{ listSummary }}</h2>
    <ul aria-labelledby="list-summary" class="stack-large">
      <li v-for="item in TaskItems" :key="item.id">
        <task-item 
          :label="item.label" 
          :done="item.done" 
          :id="item.id" 
          @checkbox-changed="updateDoneStatus(item.id)"
          @item-deleted="deleteTask(item.id)"
          @item-edited="editTask(item.id, $event)">
        </task-item>
      </li>
    </ul>
  </div>
</template>

<script>
  import TaskItem from './components/TaskItem.vue';
  import TaskForm from './components/TaskForm.vue';
  import uniqueId from 'lodash.uniqueid';

  export default {
    name: 'App',
    components: {
      TaskItem, TaskForm
    },
    data() {
      return {
        TaskItems: [],
      };
    },
    methods: {
      addTask(taskLabel) {
        console.log("Task added:", taskLabel);
        this.TaskItems.push({
          id: uniqueId('task-'),
          label: taskLabel,
          done: false
        });
      },
      updateDoneStatus(taskId) {
        const taskToUpdate = this.TaskItems.find((item) => item.id === taskId);
        taskToUpdate.done = !taskToUpdate.done;
      },
      deleteTask(taskId) {
        const itemIndex = this.TaskItems.findIndex((item) => item.id === taskId);
        this.TaskItems.splice(itemIndex, 1);
        this.$refs.listSummary.focus();
      },
      editTask(taskId, newLabel) {
        const taskToEdit = this.TaskItems.find((item) => item.id === taskId);
        taskToEdit.label = newLabel;
      },
    },
    computed: {
      listSummary() {
        const numberFinishedItems = this.TaskItems.filter((item) => item.done).length;
        return `${numberFinishedItems} out of ${this.TaskItems.length} items completed`;
      },
    },
  };
</script>

<style>
  /* Global styles */
  .btn {
    padding: 0.8rem 1rem 0.7rem;
    border: 0.2rem solid #4d4d4d;
    cursor: pointer;
    text-transform: capitalize;
  }
  .btn__danger {
    color: #fff;
    background-color: #ca3c3c;
    border-color: #bd2130;
  }
  .btn__filter {
    border-color: lightgrey;
  }
  .btn__danger:focus {
    outline-color: #c82333;
  }
  .btn__primary {
    color: #fff;
    background-color: #000;
  }
  .btn-group {
    display: flex;
    justify-content: space-between;
  }
  .btn-group > * {
    flex: 1 1 auto;
  }
  .btn-group > * + * {
    margin-left: 0.8rem;
  }
  .label-wrapper {
    margin: 0;
    flex: 0 0 100%;
    text-align: center;
  }
  [class*="__lg"] {
    display: inline-block;
    width: 100%;
    font-size: 1.9rem;
  }
  [class*="__lg"]:not(:last-child) {
    margin-bottom: 1rem;
  }
  @media screen and (min-width: 620px) {
    [class*="__lg"] {
      font-size: 2.4rem;
    }
  }
  .visually-hidden {
    position: absolute;
    height: 1px;
    width: 1px;
    overflow: hidden;
    clip: rect(1px 1px 1px 1px);
    clip: rect(1px, 1px, 1px, 1px);
    clip-path: rect(1px, 1px, 1px, 1px);
    white-space: nowrap;
  }
  [class*="stack"] > * {
    margin-top: 0;
    margin-bottom: 0;
  }
  .stack-small > * + * {
    margin-top: 1.25rem;
  }
  .stack-large > * + * {
    margin-top: 2.5rem;
  }
  @media screen and (min-width: 550px) {
    .stack-small > * + * {
      margin-top: 1.4rem;
    }
    .stack-large > * + * {
      margin-top: 2.8rem;
    }
  }
  /* End global styles */
  #app {
    background: #fff;
    margin: 2rem 0 4rem 0;
    padding: 1rem;
    padding-top: 0;
    position: relative;
    box-shadow:
      0 2px 4px 0 rgba(0, 0, 0, 0.2),
      0 2.5rem 5rem 0 rgba(0, 0, 0, 0.1);
  }
  @media screen and (min-width: 550px) {
    #app {
      padding: 4rem;
    }
  }
  #app > * {
    max-width: 50rem;
    margin-left: auto;
    margin-right: auto;
  }
  #app > form {
    max-width: 100%;
  }
  #app h1 {
    display: block;
    min-width: 100%;
    width: 100%;
    text-align: center;
    margin: 0;
    margin-bottom: 1rem;
  }
</style>


<!-- <script setup>
  import { ref, onMounted, computed, watch } from 'vue'

  const tasks = ref([])
  const name = ref('')
  const input_content = ref('')
  const input_category = ref(null)
  const tasks_asc = computed(() => tasks.value.sort((a, b) => {
    return a.createdAt - b.createdAt
  }))

  watch(name, (newVal) => {
    localStorage.setItem('name', newVal)
  })

  watch(tasks, (newVal) => {
    localStorage.setItem('tasks', JSON.stringify(newVal))
  }, { deep: true })

  const addTask = () => {
    if (input_content.value.trim === '' || input_category.value === null) {
      return
    }

    tasks.value.push({
      content: input_content.value,
      category: input_category.value,
      done: false,
      createdAt: new Date().getTime()
    })

    input_content.value = ''
    input_category.value = null
  }

  const removeTask = task => {
    tasks.value = tasks.value.filter(t => t !== task)
  }

  onMounted(() => {
    name.value = localStorage.getItem('name') || ''
    tasks.value = JSON.parse(localStorage.getItem('tasks')) || []
  })

</script>

<template>
  <main class="app">
    <section class="greeting">
      <h2>
        Greetings, <input type="text" placeholder="Your name here" v-model="name" />
      </h2>
    </section>
    <section class="create-task">
      <h3>Create Task:</h3>

      <form @submit.prevent="addTask">
        <h4>What is your next task?</h4>
        <input type="text" placeholder="Task content" v-model="input_content" />
        <h4>Business or Personal?</h4>
        <div class="options">
          <label>
            <input type="radio" name="category" value="business" v-model="input_category" />
            <span class="bubble business"></span>
            <div>Business</div>
          </label>
          <label>
            <input type="radio" name="category" value="personal" v-model="input_category" />
            <span class="bubble personal"></span>
            <div>Personal</div>
          </label>
        </div>
        <input type="submit" value="Add Task" />
      </form>
    </section>
    <section class="task-list">
      <h3>Task List:</h3>
      <div class="list">
        <div v-for="task in tasks_asc" :class="`task-item ${task.done && 'done'}`">
          <label>
            <input type="checkbox" v-model="todo.done" />
            <span :class="`bubble ${task.category}`"></span>
          </label>
          <div class="task-content">
            <input type="text" v-model="task.content" />
          </div>
          <div class="actions">
            <button class="remove" @click="removeTask(task)">Remove</button>
          </div>
        </div>
      </div>
    </section>
  </main>
</template> -->


