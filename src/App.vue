<template>
  <main>
    <div
      v-if="openTaskpopup || addTaskpopup || boardPopup"
      class="bg-black/40 fixed h-full w-full"
      id="fade"
    ></div>
    <div v-if="addTaskpopup" class="popup">
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
            v-model="task.title"
          />
        </div>
        <div>
          <label for="description">Description</label>
          <textarea id="description" v-model="task.description"></textarea>
        </div>
        <div>
          <label>Subtasks</label>
          <div
            class="flex justify-between"
            v-for="(sub, num) in subtask"
            v-bind:key="num"
          >
            <input
              @change="addSubtask"
              type="text"
              class="subtask"
              v-model="subtask[num]"
            />
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
            <select v-model="task.status">
              <option value="todo">To Do</option>
              <option value="inprogress">In Progress</option>
              <option value="done">Done</option>
            </select>
          </div>
        </div>
        <button class="bg-blue-500 text-white p-2 rounded">Add</button>
      </form>
    </div>
    <div v-if="boardPopup" class="board-popup">
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
    <div v-if="openTaskpopup" class="task-popup">
      <div class="flex place-items-center justify-between">
        <p class="font-medium">{{ task[0].title }}</p>
        <button>
          <img src="./assets/menu.svg" class="w-6" alt="menu" />
        </button>
      </div>
      <div>
        <p class="text-sm text-gray-500">{{ task[0].description }}</p>
      </div>
      <div>
        <p class="text-sm">
          Subtasks({{ task[0].completed }} of {{ task[0].subtask.length }})
        </p>
        <div
          v-for="task of task[0].subtask"
          v-bind:key="task"
          class="p-4 flex place-items-center relative bg-[#21212D] rounded mt-3"
        >
          <div>
            <input
              :data-slug="task.id"
              v-if="tasksValue.length > 0"
              :checked="task.completed"
              type="checkbox"
              @change="finishedSubtasks(task.id)"
            />
            <span class="checkbox"></span>
            <p
              v-if="tasksValue.length > 0"
              class="ml-8"
              :class="{ complete: task.completed }"
            >
              {{ task.name }}
            </p>
          </div>
        </div>
        <div class="select-style">
          <select v-model="task[0].status" @change="taskStatus">
            <option value="todo">To Do</option>
            <option value="inprogress">In Progress</option>
            <option value="done">Done</option>
          </select>
        </div>
      </div>
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
            class="flex place-items-center board"
            v-for="board of boards"
            v-bind:key="board"
            @click="activeStatus($event)"
          >
            <svg
              class="w-6 mr-2 fill-[#717785]"
              xmlns="http://www.w3.org/2000/svg"
              viewBox="0 0 24 24"
            >
              <path
                d="M22,11.05c0-.11,0-.22,0-.33l-.09-.6-.09-.39c0-.17-.08-.34-.13-.51s-.08-.27-.13-.4a2.17,2.17,0,0,1-.07-.24s0,0,0-.05a10.1,10.1,0,0,0-5.9-5.9s0,0-.05,0l-.23-.07-.42-.13c-.15,0-.31-.08-.46-.12l-.46-.1-.46-.07c-.16,0-.31,0-.48-.06s-.35,0-.52,0L12,2l-.39,0c-.17,0-.35,0-.52,0s-.32,0-.48.06l-.46.07-.46.1c-.15,0-.31.07-.46.12l-.42.13-.23.07s0,0-.05,0a10.1,10.1,0,0,0-5.9,5.9s0,0,0,.05a2.17,2.17,0,0,1-.07.24c0,.13-.09.26-.13.4s-.09.34-.13.51l-.09.39-.09.6c0,.11,0,.22,0,.33,0,.31,0,.63,0,.95s0,.64,0,.95c0,.11,0,.22,0,.33l.09.6.09.39c0,.17.08.34.13.51s.08.27.13.4a2.17,2.17,0,0,1,.07.24.43.43,0,0,1,0,.07,10,10,0,0,0,5.89,5.88s0,0,.05,0l.24.07.4.13.51.13.39.09.6.09.33,0c.31,0,.63.05.95.05s.64,0,.95-.05l.33,0,.6-.09.39-.09.51-.13.4-.13.24-.07s0,0,.05,0a10,10,0,0,0,5.89-5.88.43.43,0,0,1,0-.07c0-.08.05-.16.07-.24s.09-.26.13-.4.09-.34.13-.51l.09-.39.09-.6c0-.11,0-.22,0-.33,0-.31.05-.63.05-.95S22,11.36,22,11.05Zm-6.3-6.16a8,8,0,0,1,3.46,3.46l-2.86,1a5.14,5.14,0,0,0-1.64-1.64Zm-5.36-.7c.21-.05.41-.08.61-.11l.24,0a8.24,8.24,0,0,1,1.72,0l.24,0c.2,0,.4.06.61.11h.06l-1,2.86A4.49,4.49,0,0,0,12,7a4.4,4.4,0,0,0-.73.06l-1-2.86Zm-1.94.7,1,2.86A5.14,5.14,0,0,0,7.75,9.39l-2.86-1A8,8,0,0,1,8.35,4.89ZM4.19,13.71a4.17,4.17,0,0,1-.1-.6c0-.09,0-.17,0-.25a7.42,7.42,0,0,1,0-1.72c0-.08,0-.16,0-.25a4.17,4.17,0,0,1,.1-.6s0,0,0-.06l2.86,1a4.47,4.47,0,0,0,0,1.46l-2.86,1S4.19,13.73,4.19,13.71Zm4.16,5.4a8,8,0,0,1-3.46-3.46l2.86-1a5.14,5.14,0,0,0,1.64,1.64Zm5.36.7c-.21.05-.41.08-.61.11l-.24,0a8.24,8.24,0,0,1-1.72,0l-.24,0c-.2,0-.4-.06-.61-.11h-.06l1-2.86a4.47,4.47,0,0,0,1.46,0l1,2.86Zm-.67-5h0c-.17.06-.34.1-.5.14a2.73,2.73,0,0,1-1,0c-.16,0-.33-.08-.5-.14h0A3,3,0,0,1,9.2,13v0a3.23,3.23,0,0,1-.14-.51,2.63,2.63,0,0,1,0-1A3.23,3.23,0,0,1,9.19,11v0A3,3,0,0,1,11,9.2h0c.17-.06.34-.1.5-.14a2.73,2.73,0,0,1,1,0c.16,0,.33.08.5.14h0A3,3,0,0,1,14.8,11v0a3.23,3.23,0,0,1,.14.51,2.63,2.63,0,0,1,0,1,3.23,3.23,0,0,1-.14.51v0A3,3,0,0,1,13,14.8Zm2.61,4.31-1-2.86a5.14,5.14,0,0,0,1.64-1.64l2.86,1A8,8,0,0,1,15.65,19.11ZM20,12.86c0,.08,0,.16,0,.25a4.17,4.17,0,0,1-.1.6s0,0,0,.06l-2.86-1a4.47,4.47,0,0,0,0-1.46l2.86-1s0,0,0,.06a4.17,4.17,0,0,1,.1.6c0,.09,0,.17,0,.25a7.42,7.42,0,0,1,0,1.72Z"
              />
            </svg>
            {{ board }}
          </li>
          <li @click="newItem">+ Create New Board</li>
        </ul>
      </div>
    </div>
    <div class="w-full">
      <nav>
        <div class="flex place-items-center">
          <h1>{{ activeBoard }}</h1>
          <svg
            class="w-6 block md:hidden pt-2 mx-2"
            fill="#635FC6"
            xmlns="http://www.w3.org/2000/svg"
            viewBox="0 0 24 24"
          >
            <path
              d="M17,9.17a1,1,0,0,0-1.41,0L12,12.71,8.46,9.17a1,1,0,0,0-1.41,0,1,1,0,0,0,0,1.42l4.24,4.24a1,1,0,0,0,1.42,0L17,10.59A1,1,0,0,0,17,9.17Z"
            />
          </svg>
        </div>
        <div class="flex place-items-center">
          <button @click="taskpopup" class="btn btn-primary hidden md:block">
            + Add New Task
          </button>
          <button
            @click="taskpopup"
            class="btn btn-primary text-xl block md:hidden"
          >
            +
          </button>
          <button class="w-6 mx-3">
            <img src="./assets/menu.svg" />
          </button>
        </div>
      </nav>
      <div class="content">
        <div class="flex place-items-center min-w-full md:min-w-fit">
          <div>
            <div class="flex place-items-center">
              <span class="w-4 h-4 rounded-full bg-blue-300 mr-2"></span>
              <p class="uppercase">todo ({{ todo.length }})</p>
            </div>
            <div
              v-for="task of boardTasks"
              v-bind:key="task"
              v-show="boardTasks.length > 0"
            >
              <div
                v-if="task.status == 'todo'"
                :data-slug="task.id"
                class="card"
                @click="openTask($event)"
              >
                <p>{{ task.title }}</p>
                <span>
                  {{ task.completed }} of
                  {{ task.subtask.length }}
                  subtasks
                </span>
              </div>
            </div>
          </div>
        </div>
        <div class="flex place-items-center min-w-full md:min-w-fit">
          <div>
            <div class="flex place-items-center">
              <span class="w-4 h-4 rounded-full bg-purple-300 mr-2"></span>
              <p class="uppercase">in progress ({{ inprogress.length }})</p>
            </div>
            <div
              v-for="task of boardTasks"
              v-bind:key="task"
              :data-slug="task.id"
              @click="openTask($event)"
            >
              <div v-if="task.status == 'inprogress'" class="card">
                <p>{{ task.title }}</p>
                <span>
                  {{ task.completed }} of
                  {{ task.subtask.length }}
                  subtasks
                </span>
              </div>
            </div>
          </div>
        </div>
        <div class="flex place-items-center min-w-full md:min-w-fit">
          <div>
            <div class="flex place-items-center">
              <span class="w-4 h-4 rounded-full bg-green-300 mr-2"></span>
              <p class="uppercase">done ({{ done.length }})</p>
            </div>
            <div v-for="task of boardTasks" v-bind:key="task">
              <div
                v-if="task.status == 'done'"
                :data-slug="task.id"
                class="card"
                @click="openTask($event)"
              >
                <p>{{ task.title }}</p>
                <span>
                  {{ task.completed }} of
                  {{ task.subtask.length }}
                  subtasks
                </span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </main>
</template>
<script>
import axios from "axios";

export default {
  name: "App",
  data() {
    return {
      boards: ["main"],
      board: "",
      title: "",
      description: "",
      subtask: [],
      status: "",
      number: 1,
      tasksValue: [],
      todo: [],
      inprogress: [],
      subtaskId: 0,
      done: [],
      completed: [],
      activeBoard: "",
      id: 0,
      test: [],
      task: {},
      boardTasks: [],
      openTaskpopup: false,
      addTaskpopup: false,
      boardPopup: false,
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
    localStorage.getItem("boards") &&
      (this.boards = JSON.parse(localStorage.getItem("boards")));
    localStorage.getItem("boardTasks") &&
      (this.boardTasks = JSON.parse(localStorage.getItem("boardTasks")));
    localStorage.getItem("active") &&
      (this.activeBoard = JSON.parse(localStorage.getItem("active")));
    localStorage.getItem("tasks") &&
      (this.task = JSON.parse(localStorage.getItem("tasks")));
    console.log(this.boardTasks);
    if (this.activeBoard == "") {
      this.activeBoard = "main";
    }
    setTimeout(() => {
      const boards = document.querySelectorAll(".board");
      for (const board of boards) {
        if (board.innerText == this.activeBoard) {
          board.classList.add("active");
        }
      }
    }, 100);
    axios.get("http://localhost:3000/posts").then((response) => {
      this.tasksValue = response.data;
    });
  },
  watch: {
    openTaskpopup(newValue) {
      if (newValue == true) {
        window.addEventListener("click", function (e) {
          if (e.target.closest(".task-popup") || e.target.closest(".card")) {
            document.querySelector(".task-popup").classList.remove("hidden");
            document.getElementById("fade").classList.remove("hidden");
            newValue = true;
          } else {
            document.querySelector(".task-popup").classList.add("hidden");
            window.location.reload();
            newValue = false;
            document.getElementById("fade").classList.add("hidden");
          }
        });
      }
    },
  },
  methods: {
    finishedSubtasks(index) {
      this.task.filter((task) => {
        task.subtask.filter((subtask) => {
          if (subtask.id == index) {
            if (subtask.completed == false) {
              subtask.completed = true;
              this.completed.push(subtask);
              console.log(this.completed.length, "completed");
              task.completed = this.completed.length;
            } else {
              subtask.completed = false;
              this.completed.pop();
              task.completed = this.completed.length;
              console.log(this.completed.length, "completed");
            }
          }
        });
      });
      this.boardTasks.filter((task) => {
        if (task.id == this.task[0].id) {
          task.completed = this.completed.length;
        }
      });
      console.log(this.boardTasks, "task");
      localStorage.setItem("completed", JSON.stringify(this.completed));
      localStorage.setItem("tasks", JSON.stringify(this.task));
      localStorage.setItem("task", JSON.stringify(this.tasksValue));
      localStorage.setItem("boardTasks", JSON.stringify(this.boardTasks));
    },
    taskStatus() {
      console.log(this.tasksValue);
      this.todo = this.tasksValue.filter((task) => task.status == "todo");
      this.inprogress = this.tasksValue.filter(
        (task) => task.status == "inprogress"
      );
      this.done = this.tasksValue.filter((task) => task.status == "done");
      this.boardTasks = this.tasksValue.filter(
        (task) => task.board == this.activeBoard
      );
      localStorage.setItem("todo", JSON.stringify(this.todo));
      localStorage.setItem("inprogress", JSON.stringify(this.inprogress));
      localStorage.setItem("done", JSON.stringify(this.done));
      localStorage.setItem("boardTasks", JSON.stringify(this.boardTasks));
      localStorage.setItem("task", JSON.stringify(this.tasksValue));
    },
    activeStatus(e) {
      const list = document.querySelectorAll(".board");
      for (const board of list) {
        board.classList.remove("active");
      }
      e.currentTarget.classList.add("active");
      this.activeBoard = e.currentTarget.innerText;
      console.log(this.activeBoard);
      this.boardTasks = this.tasksValue.filter(
        (task) => task.board == this.activeBoard
      );
      this.todo = this.boardTasks.filter((task) => task.status == "todo");
      this.inprogress = this.boardTasks.filter(
        (task) => task.status == "inprogress"
      );
      this.done = this.boardTasks.filter((task) => task.status == "done");
      localStorage.setItem("todo", JSON.stringify(this.todo));
      localStorage.setItem("inprogress", JSON.stringify(this.inprogress));
      localStorage.setItem("done", JSON.stringify(this.done));
      localStorage.setItem("boardTasks", JSON.stringify(this.boardTasks));
      localStorage.setItem("active", JSON.stringify(this.activeBoard));
    },
    taskpopup() {
      this.addTaskpopup = true;
      this.task = {};
    },
    openTask(e) {
      this.openTaskpopup = true;
      if (
        this.subtask[0] == "" ||
        this.subtask[0] == undefined ||
        this.subtask[0] == null
      ) {
        this.subtask.shift();
      }
      this.task = this.tasksValue.filter((task) => {
        if (task.id == e.currentTarget.dataset.slug) {
          return task;
        }
      });
    },
    closePopup() {
      this.addTaskpopup = false;
      this.boardPopup = false;
    },
    addTask() {
      this.task.id = this.tasksValue.length + 1;
      this.task.board = this.activeBoard;
      this.task.completed = 0;
      if (
        this.task.title == "" ||
        this.task.title == undefined ||
        this.task.title == null ||
        this.task.subtask == "" ||
        this.task.subtask == undefined ||
        this.task.subtask == null
      ) {
        alert("Please fill all the fields");
      } else {
        console.log(this.task, "tasj");
        this.tasksValue.push(this.task);
        this.boardTasks.push(this.task);
        console.log(this.boardTasks, "boardTaskstasj");
        this.addTaskpopup = false;
      }
      console.log(this.tasksValue);
      this.todo = this.boardTasks.filter((task) => task.status == "todo");
      this.inprogress = this.boardTasks.filter(
        (task) => task.status == "inprogress"
      );
      this.done = this.boardTasks.filter((task) => task.status == "done");
      localStorage.setItem("task", JSON.stringify(this.tasksValue));
      localStorage.setItem("todo", JSON.stringify(this.todo));
      localStorage.setItem("inprogress", JSON.stringify(this.inprogress));
      localStorage.setItem("done", JSON.stringify(this.done));
      localStorage.setItem("subtask", JSON.stringify(this.subtaskValue));
      localStorage.setItem("boardTasks", JSON.stringify(this.boardTasks));
      this.task = {};
      this.subtask = [];
      this.subtaskValue = [];
    },
    addSubtask() {
      if (this.subtask[this.subtask.length - 1] != "") {
        this.subtask.push("");
        this.subtaskValue = this.subtask.filter((subtask) => {
          if (subtask != "" && subtask != undefined && subtask != null) {
            this.subtaskValue.push({
              name: subtask,
              id: this.subtaskId++,
              completed: false,
            });
            this.task.subtask = this.subtaskValue;
            console.log(this.task.subtask, "task");
          }
        });
      } else {
        alert("Please enter a subtask");
      }
    },
    removeSubtask(e) {
      if (this.subtask.length > 1) {
        console.log(e.target.parentNode);
        for (const task of this.subtask) {
          if (task == e.target.parentNode.children[0].value) {
            this.subtask.splice(this.subtask.indexOf(task), 1);
          }
          console.log(this.subtask);
        }
      }
    },

    newItem() {
      this.boardPopup = true;
    },
    addItem() {
      this.boards.push(this.board);
      localStorage.setItem("boards", JSON.stringify(this.boards));
      this.board = "";
      console.log(this.boards);
      document.querySelector(`.board-popup`).classList.add(`hidden`);
      document.querySelector(`#fade`).classList.add(`hidden`);
    },
  },
};
</script>
