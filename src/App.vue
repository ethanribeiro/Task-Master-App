<template>
<div>
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
        const task = {
          id: uniqueId('task-'),
          label: taskLabel,
          done: false
        }
        this.TaskItems.push(task);
        this.addTaskToDB(task);
      },
      async addTaskToDB(task) {
        return new Promise((resolve, reject) => {
          let trans = this.db.transaction(['tasks'],'readwrite');
          trans.oncomplete = e => {
            console.log("Completed transfering task!!!")
            resolve();
          };

          let store = trans.objectStore('tasks');
          store.add(task);
          console.log(task);
        });
      },
      async getTasksFromDb() {
        return new Promise((resolve, reject) => {
          let trans = this.db.transaction(['tasks'],'readonly');
          trans.oncomplete = e => {
            resolve(tasks);
          };
          let store = trans.objectStore('tasks');
          let tasks = [];
          store.openCursor().onsuccess = e => {
            let cursor = e.target.result;
            if (cursor) {
              tasks.push(cursor.value)
              cursor.continue();
            }
          };
        });
      },
      updateDoneStatus(taskId) {
        const taskToUpdate = this.TaskItems.find((item) => item.id === taskId);
        taskToUpdate.done = !taskToUpdate.done;
        this.updateTaskFromDb(taskToUpdate);
      },
      async updateTaskFromDb(task) {
        return new Promise((resolve, reject) => {
          let trans = this.db.transaction(['tasks'], 'readwrite');
          trans.oncomplete = e => {
            resolve(task);
          };
          let store = trans.objectStore('tasks');
          console.log({...task});
          store.put({...task});
        })
      },
      deleteTask(taskId) {
        const itemIndex = this.TaskItems.findIndex((item) => item.id === taskId);
        this.TaskItems.splice(itemIndex, 1);
        this.deleteTaskFromDb(taskId);
        this.$refs.listSummary.focus();
      },
      async deleteTaskFromDb(taskId) {
        return new Promise((resolve, reject) => {
          let trans = this.db.transaction(['tasks'],'readwrite');
          trans.oncomplete = e => {
            resolve();
          };

          let store = trans.objectStore('tasks');
          store.delete(taskId);
        });
      },
      editTask(taskId, newLabel) {
        const taskToEdit = this.TaskItems.find((item) => item.id === taskId);
        taskToEdit.label = newLabel;
        this.updateTaskFromDb(taskToEdit);
      },
      async getDb() {
        return new Promise((resolve, reject) => {
          let request = window.indexedDB.open('taskdb', 1);
          console.log(request)
          request.onerror = e => {
            console.log('Error opening db', e);
            reject(e);
          };

          request.onsuccess = e => {
            resolve(e.target.result);
          };

          request.onupgradeneeded = e => {
            console.log('onupgradeneeded');
            let db = e.target.result;
            let objectStore = db.createObjectStore("tasks", { autoIncrement: true, keyPath:'id' });
          };
        });
      },

    },
    async created() {
      console.log("Hello")
      this.db = await this.getDb();
      console.log(this.db);
      this.TaskItems = await this.getTasksFromDb();
      console.log(this.TaskItems)
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
