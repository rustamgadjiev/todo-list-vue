<template>
  <div class="wrapper">
    <header class="header">
      <div class="container">
        <h2>Todo App</h2>
        <div class="form">
          <input type="text" v-model="valueInput" placeholder="Add Task">
          <button class="btn" @click="addTask">Add Task</button>
        </div>
      </div>
    </header>
    <div class="tasks">
      <div class="container">
        <ul>
          <li v-for="task in todoList" :key="task.index">
            <div>
              <div class="checkbox">
                <input v-bind:id="task.index" type="checkbox" @input="completedTask(task)" v-bind:checked="task.completed">
                <label v-bind:for="task.index"></label>
              </div>
              <span>{{ task.title }}</span>
            </div>
            <div class="btns">
              <button class="btn-rem" @click="deleteTask(task)">
                <svg xmlns="http://www.w3.org/2000/svg" width="20px" height="20px" viewBox="0 0 30 30"><path d="M 13 3 A 1.0001 1.0001 0 0 0 11.986328 4 L 6 4 A 1.0001 1.0001 0 1 0 6 6 L 24 6 A 1.0001 1.0001 0 1 0 24 4 L 18.013672 4 A 1.0001 1.0001 0 0 0 17 3 L 13 3 z M 6 8 L 6 24 C 6 25.105 6.895 26 8 26 L 22 26 C 23.105 26 24 25.105 24 24 L 24 8 L 6 8 z"/></svg>
              </button>
              <button class="btn-edit" @click="selectedTask(task)">
                <svg xmlns="http://www.w3.org/2000/svg"  viewBox="0 0 30 30" width="20px" height="20px"><path d="M 22.828125 3 C 22.316375 3 21.804562 3.1954375 21.414062 3.5859375 L 19 6 L 24 11 L 26.414062 8.5859375 C 27.195062 7.8049375 27.195062 6.5388125 26.414062 5.7578125 L 24.242188 3.5859375 C 23.851688 3.1954375 23.339875 3 22.828125 3 z M 17 8 L 5.2597656 19.740234 C 5.2597656 19.740234 6.1775313 19.658 6.5195312 20 C 6.8615312 20.342 6.58 22.58 7 23 C 7.42 23.42 9.6438906 23.124359 9.9628906 23.443359 C 10.281891 23.762359 10.259766 24.740234 10.259766 24.740234 L 22 13 L 17 8 z M 4 23 L 3.0566406 25.671875 A 1 1 0 0 0 3 26 A 1 1 0 0 0 4 27 A 1 1 0 0 0 4.328125 26.943359 A 1 1 0 0 0 4.3378906 26.939453 L 4.3632812 26.931641 A 1 1 0 0 0 4.3691406 26.927734 L 7 26 L 5.5 24.5 L 4 23 z"/></svg>
              </button>
            </div>
          </li>
        </ul>
      </div>
    </div>
    <div v-if="showModal" class="modal">
      <div class="modal-content">
        <span class="close-btn" @click="showModal = false">&times;</span>
        <h2>Edit Task</h2>
        <div>
          <input type="text" v-model="valueEditTask" placeholder="Edit Task">
          <button class="btn" @click="editTask(todoList.find(task => task.index === selectedTaskIndex))">Edit Task</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'App',
  components: {
  },
  data() {
    return {
      todoList: [],
      editedTask: {},
      showModal: false,
      selectedTaskIndex: null,
    }
  },
  methods: {
    addTask() {
      if (this.valueInput) {
        axios.post('https://6481fbdb29fa1c5c50326b3a.mockapi.io/todoList/', {
          title: this.valueInput
        })
          .then(response => {
            this.todoList.push(response.data);
            this.valueInput = '';
          })
          .catch(e => {
            console.log(e);
          })
      }
    },
    deleteTask(task) {
      axios.delete('https://6481fbdb29fa1c5c50326b3a.mockapi.io/todoList/' + task.index)
        .finally(() => {
          this.todoList = this.todoList.filter(todo => todo.index !== task.index);
        });
    },
    selectedTask(task) {
      this.selectedTaskIndex = task.index;
      this.showModal = true;
      this.valueEditTask = task.title;
    },
    editTask(task) {
      this.editedTask = { ...task, title: this.valueEditTask };
      axios.put('https://6481fbdb29fa1c5c50326b3a.mockapi.io/todoList/' + task.index, this.editedTask)
        .finally(() => {
          this.todoList[this.todoList.findIndex(todo => todo.index === task.index)] = this.editedTask;
          this.editedTask = {};
          this.showModal = false;
        })
    },
    completedTask(task) {
      axios.put('https://6481fbdb29fa1c5c50326b3a.mockapi.io/todoList/' + task.index, {
        completed: !task.completed
      })
        .then(response => {
          task.completed = response.data.completed;
        })
        .catch(e => {
          console.log(e);
        });
    }
  },
  mounted() {
    axios.get('https://6481fbdb29fa1c5c50326b3a.mockapi.io/todoList')
      .then(response => {
        this.todoList = response.data;
      })
      .catch(e => {
        console.log(e);
      });
  },
}
</script>

<style>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  background: linear-gradient(90deg, #3111FF, #6473FF);
  height: 100vh;
  background-repeat: no-repeat;
  font-family: sans-serif;
  color: #fff;
  font-size: 16px;
  font-weight: 700;
}

#app {
  text-align: center;
  padding-top: 20px;
}

.wrapper {
  max-width: 500px;
  min-height: 400px;
  height: 100%;
  background-color: #161A2B;
  margin: 0px auto;
  padding: 20px 10px;
  border-radius: 10px;
}

.form {
  margin-top: 20px;
}

.form input {
  padding: 10px;
  border: 0;
  outline: none;
  background-color: transparent;
  color: #fff;
  border: 2px solid #620CFF;
  border-radius: 3px 0 0 3px;
  font-size: 16px;
  font-weight: 700;
}

.form .btn {
  background: linear-gradient(90deg, #620CFF, #9901FB);
  border: 0;
  outline: none;
  color: #fff;
  border-radius: 0 3px 3px 0;
  padding: 12px 10px;
  cursor: pointer;
  font-size: 16px;
  font-weight: 700;
  transition-duration: 0.3s;
}

.form .btn:hover {
  filter: brightness(115%);
}

.tasks {
  margin-top: 30px;
}

.tasks .btns {
  display: flex;
  align-items: center;
  gap: 10px;
}

.tasks .btns button {
  border: 0;
  outline: none;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: transparent;
  cursor: pointer;
  padding: 5px;
  border-radius: 3px;
  transition-duration: .3s;
}

.tasks .btns button:hover {
  background-color: #ffffff6e;
}

.tasks ul li {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 5px;
  width: 100%;
  background: linear-gradient(90deg, #149FFF, #127BFF);
  padding: 11px;
  border-radius: 5px;
  transition-duration: .3s;
}
.tasks ul li:hover {
  filter: brightness(115%);
}
.tasks ul li:not(:first-child) {
  margin-top: 5px;
}
.tasks ul li:nth-child(2n) {
  background: linear-gradient(90deg, #FF7614, #FF5412);
}
.tasks ul li:nth-child(3n) {
  background: linear-gradient(90deg, #5E0CFF, #9A00FA);
}
.tasks ul li:nth-child(4n) {
  background: linear-gradient(90deg, #FF0CF0, #FA0088);
}
.tasks ul li div:first-child {
  display: flex;
  align-items: center;
}

.tasks .checkbox input {
  display: none;
}

.tasks .checkbox label {
  display: inline-block;
  position: relative;
  padding-left: 28px;
  margin-right: 5px;
  margin-top: 22px;
  cursor: pointer;
  font-size: 18px;
  color: #333;
}

.tasks .checkbox label:before {
  content: "";
  display: inline-block;
  width: 20px;
  height: 20px;
  margin-right: 10px;
  position: absolute;
  left: 0;
  bottom: 0px;
  background-color: transparent;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.tasks .checkbox input:checked + label:before {
  content: "\2713";
  text-align: center;
  line-height: 22px;
  font-size: 16px;
  color: #fff;
  background-color: #38c172;
  border-color: #38c172;
}

.modal {
  display: block;
  position: absolute;
  z-index: 10;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  overflow: hidden;
  background-color: rgba(0, 0, 0, 0.4);
  animation: modal-fade-in 0.3s ease-out forwards;
}

.modal-content {
  background-color: #161A2B;
  border-radius: 10px;
  margin: 5% auto;
  padding: 20px;
  max-width: 480px;
  width: 100%;
  min-height: 250px;
  position: relative;
  display: flex;
  flex-direction: column;
  animation: modal-slide-down 0.3s ease-out forwards;
}

.modal-content input {
  padding: 10px;
  border: 0;
  outline: none;
  background-color: transparent;
  color: #fff;
  border: 2px solid #620CFF;
  border-radius: 3px 0 0 3px;
  font-size: 16px;
  font-weight: 700;
  margin-top: 50px;
}

.modal-content .btn {
  background: linear-gradient(90deg, #620CFF, #9901FB);
  border: 0;
  outline: none;
  color: #fff;
  border-radius: 0 3px 3px 0;
  padding: 12px 10px;
  cursor: pointer;
  font-size: 16px;
  font-weight: 700;
  transition-duration: 0.3s;
}

.close-btn {
  color: #5a5a5a;
  position: absolute;
  z-index: 11;
  right: 20px;
  top: 10px;
  font-size: 28px;
  cursor: pointer;
}

@keyframes modal-fade-in {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

@keyframes modal-slide-down {
  from {
    opacity: 0;
    transform: translateY(-50px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
</style>
