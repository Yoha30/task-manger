<template>
  <main>
    <div class="bg-black/40 fixed h-full w-full hidden" id="fade"></div>
    <div class="popup hidden">
      <div class="flex justify-between">
        <h1 class="text-2xl font-bold">Add a new task</h1>
        <button class="text-2xl font-bold" @click="closePopup">X</button>
      </div>
      <form class="flex flex-col" @submit.prevent="addTask">
        <div>
          <label for="title">Title</label>
          <input
            placeholder="e.g Take coffee breake"
            type="text"
            id="title"
            v-model="title"
          />
        </div>
        <div>
          <label for="description">Description</label>
          <textarea id="description" v-model="description"></textarea>
        </div>
        <div>
          <label>Subtasks</label>
          <div
            class="flex justify-between"
            v-for="num of number"
            v-bind:key="num"
          >
            <input type="text" class="subtask" v-model="subtask[num]" />
            <button
              @click="removeSubtask($event)"
              type="button"
              class="text-2xl font-bold text-[#838C9E]"
            >
              X
            </button>
          </div>
          <button
            type="button"
            class="bg-white text-blue-500 font-medium rounded-full w-full p-2"
            @click="addSubtask"
          >
            + Add New Subtask
          </button>
        </div>
        <div>
          <label>Status</label>
          <div class="select-style">
            <select v-model="status">
              <option value="todo">To Do</option>
              <option value="inprogress">In Progress</option>
              <option value="done">Done</option>
            </select>
          </div>
        </div>
        <button class="bg-blue-500 text-white p-2 rounded">Add</button>
      </form>
    </div>
    <div class="board-popup hidden">
      <div class="flex justify-between">
        <h1 class="text-2xl font-bold">Add a new board</h1>
        <button class="text-2xl font-bold" @click="closePopup">X</button>
      </div>
      <form @submit.prevent="addItem">
        <label for="board">Board name</label>
        <input id="board" type="text" v-model="board" />
        <button class="bg-blue-500 text-white p-2 w-full rounded">Add</button>
      </form>
    </div>
    <div class="sidebar">
      <div>
        <img src="./assets/white-logo.png" alt="logo" class="logo" />
      </div>
      <div>
        <p
          class="uppercase font-medium text-sm mt-12 mb-8 pl-4"
          style="color: var(--sidebar-text-color)"
        >
          all boards ({{ boards.length }})
        </p>
        <ul class="sidebar-list">
          <li
            class="flex place-items-center"
            v-for="board of boards"
            v-bind:key="board"
          >
            <img src="./assets/lifebuoy.svg" class="w-6 mr-2" />
            {{ board }}
          </li>
          <li @click="newItem">+ Create New Board</li>
        </ul>
      </div>
    </div>
    <div class="w-full">
      <nav>
        <h1>Platform launch</h1>
        <div class="flex place-items-center">
          <button @click="taskpopup" class="btn btn-primary">
            + Add New Task
          </button>
          <button class="w-6 mx-3">
            <img src="./assets/menu.svg" />
          </button>
        </div>
      </nav>
      <div class="content">
        <div class="flex place-items-center">
          <div>
            <div class="flex place-items-center">
              <span class="w-4 h-4 rounded-full bg-blue-300 mr-2"></span>
              <p class="uppercase">todo ({{ todo.length }})</p>
            </div>
            <div v-for="task of tasksValue" v-bind:key="task">
              <div class="card" v-if="task.status == 'todo'">
                <p>{{ task.title }}</p>
                <span>0 of {{ task.subtask.length - 1 }} subtasks</span>
              </div>
            </div>
          </div>
        </div>
        <div class="flex place-items-center">
          <div>
            <div class="flex place-items-center">
              <span class="w-4 h-4 rounded-full bg-purple-300 mr-2"></span>
              <p class="uppercase">in progress ({{ inprogress.length }})</p>
            </div>
            <div v-for="task of tasksValue" v-bind:key="task">
              <div class="card" v-if="task.status == 'inprogress'">
                <p>{{ task.title }}</p>
                <span>0 of {{ task.subtask.length - 1 }} subtasks</span>
              </div>
            </div>
          </div>
        </div>
        <div class="flex place-items-center">
          <div>
            <div class="flex place-items-center">
              <span class="w-4 h-4 rounded-full bg-green-300 mr-2"></span>
              <p class="uppercase">done ({{ done.length }})</p>
            </div>
            <div v-for="task of tasksValue" v-bind:key="task">
              <div class="card" v-if="task.status == 'done'">
                <p>{{ task.title }}</p>
                <span>0 of {{ task.subtask.length - 1 }} subtasks</span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </main>
</template>
<script>
export default {
  name: "App",
  data() {
    return {
      boards: [],
      board: "",
      title: "",
      description: "",
      subtask: [],
      status: "",
      number: 1,
      tasksValue: [],
      todo: [],
      inprogress: [],
      done: [],
    };
  },
  mounted() {
    localStorage.getItem("task") &&
      (this.tasksValue = JSON.parse(localStorage.getItem("task")));
    localStorage.getItem("todo") &&
      (this.todo = JSON.parse(localStorage.getItem("todo")));
    localStorage.getItem("inprogress") &&
      (this.inprogress = JSON.parse(localStorage.getItem("inprogress")));
    localStorage.getItem("done") &&
      (this.done = JSON.parse(localStorage.getItem("done")));
    console.log(this.tasksValue, this.inprogress);
  },
  methods: {
    taskpopup() {
      document.querySelector(`#fade`).classList.remove(`hidden`);
      document.querySelector(`.popup`).classList.remove(`hidden`);
    },
    closePopup() {
      document.querySelector(`#fade`).classList.add(`hidden`);
      document.querySelector(`.popup`).classList.add(`hidden`);
      document.querySelector(`.board-popup`).classList.add(`hidden`);
    },
    addTask() {
      document.querySelector(`#fade`).classList.add(`hidden`);
      document.querySelector(`.popup`).classList.add(`hidden`);
      if (this.subtask[this.subtask.length - 1] == "") {
        this.subtask.pop();
      }
      if (this.title == "" || this.subtask == "") {
        alert("Please enter a title");
        this.number = 1;
      } else {
        this.tasksValue.push({
          title: this.title,
          description: this.description,
          subtask: this.subtask,
          status: this.status,
        });
      }
      switch (this.status) {
        case "todo":
          this.todo.push(this.status);
          break;
        case "inprogress":
          this.inprogress.push(this.status);
          break;
        case "done":
          this.done.push(this.status);
          break;
      }
      localStorage.setItem("task", JSON.stringify(this.tasksValue));
      localStorage.setItem("todo", JSON.stringify(this.todo));
      localStorage.setItem("inprogress", JSON.stringify(this.inprogress));
      localStorage.setItem("done", JSON.stringify(this.done));
      this.title = "";
      this.description = "";
      this.subtask = [];
      this.status = "";
      this.number = 1;
    },
    addSubtask() {
      console.log(this.number);
      if (this.subtask[this.number].value != "") {
        this.number++;
      } else {
        alert("Please enter a subtask");
      }
    },
    removeSubtask(e) {
      if (this.number > 1) {
        console.log(e.target.parentNode);
        for (const task of this.subtask) {
          if (task == e.target.parentNode.children[0].value) {
            this.subtask.splice(this.subtask.indexOf(task), 1);
            this.number--;
          }
          console.log(this.subtask);
        }
      }
    },

    newItem() {
      console.log(`new item`);
      document.querySelector(`.board-popup`).classList.remove(`hidden`);
      document.querySelector(`#fade`).classList.remove(`hidden`);
    },
    addItem() {
      this.boards.push(this.board);
      this.board = "";
      console.log(this.boards);
      document.querySelector(`.board-popup`).classList.add(`hidden`);
      document.querySelector(`#fade`).classList.add(`hidden`);
    },
  },
};
</script>
