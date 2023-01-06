<script>
const LOCALSTORAGE_SET_DATA = "todoList";
const LOCALSTORAGE_GET_DATA = "todoList";
const dataTodo = JSON.parse(localStorage.getItem(LOCALSTORAGE_GET_DATA));
export default {
  data() {
    return {
      newTodoText: "",
      todoList: dataTodo ?? [],
      sorted: 0,
      checkedAll: false,
    };
  },
  computed: {
    remaining() {
      let arr = this.todoList.filter((e) => !e.isCompleted);
      return arr.length;
    },
    sortedTodoList() {
      let temp = [...this.todoList];
      temp.sort((a, b) => a.name.localeCompare(b.name));
      return temp;
    },
    reversedTodoList() {
      let temp = this.sortedTodoList.slice().reverse();
      return temp;
    },
    displayingTodoList() {
      if (this.sorted % 3 == 0) {
        return this.todoList;
      } else if (this.sorted % 3 == 1) {
        return this.sortedTodoList;
      } else {
        return this.reversedTodoList;
      }
    },
    markedAll: {
      get() {
        return this.remaining == 0;
      },
      set(val) {
        this.markAll(val);
      },
    },
  },
  methods: {
    countDuplicate(value) {
      let count = 0;
      for (let i = 0; i < this.todoList.length; i++) {
        if (value == this.todoList[i].name) {
          count++;
        }
      }
      return count;
    },
    addNewTodo() {
      if (
        this.newTodoText.trim() == "" ||
        this.countDuplicate(this.newTodoText.trim()) > 0
      ) {
        return;
      }
      this.todoList.push({
        name: this.newTodoText.trim(),
        isCompleted: false,
        isDisplay: true,
      });
      this.newTodoText = "";
      console.log(this.todoList);
    },
    displayAll() {
      this.todoList.forEach((element) => {
        element.isDisplay = true;
      });
    },
    displayActive() {
      this.todoList.forEach((element) => {
        if (element.isCompleted == true) {
          element.isDisplay = false;
        } else {
          element.isDisplay = true;
        }
      });
    },
    displayCompleted() {
      this.todoList.forEach((element) => {
        if (element.isCompleted == true) {
          element.isDisplay = true;
        } else {
          element.isDisplay = false;
        }
      });
    },
    clearCompleted() {
      this.todoList = this.todoList.filter((e) => !e.isCompleted);
    },
    checkEditingValue(value) {
      if (
        value.name.trim() == "" ||
        this.countDuplicate(value.name.trim()) > 1
      ) {
        value.isEditing = true;
        return value.isEditing;
      } else {
        value.isEditing = false;
        return value.isEditing;
      }
    },
    sortList() {
      if (this.sorted == 2) {
        this.sorted = 0;
      } else {
        this.sorted += 1;
      }
      console.log(this.sorted);
    },
    markAll(val) {
      this.todoList.forEach((element) => {
        element.isCompleted = val;
      });
    },
  },
  updated() {
    localStorage.setItem(LOCALSTORAGE_SET_DATA, JSON.stringify(this.todoList));
  },
};
</script>

<template>
  <section class="todoapp">
    <header class="header">
      <h1>Todo-List</h1>
      <span class="tooltip" v-if="this.countDuplicate(newTodoText) > 0">
        Task existed
      </span>
      <input
        id="toggle-all"
        class="toggle-all"
        type="checkbox"
        v-model="markedAll"
      />
      <label for="toggle-all"></label>
      <input
        class="new-todo"
        placeholder="What needs to be done?"
        autofocus
        v-model="newTodoText"
        @keydown.enter="addNewTodo"
      />
    </header>
    <section class="main">
      <ul class="todo-list">
        <li
          v-for="(value, index) in displayingTodoList"
          v-bind:key="index"
          class="todo"
          v-show="value.isDisplay"
        >
          <div class="view">
            <input
              class="toggle"
              type="checkbox"
              v-model="value.isCompleted"
              @click="value.isCompleted = !value.isCompleted"
            />
            <label @dblclick="value.isEditing = !value.isEditing">
              <p
                v-bind:class="{
                  completed: value.isCompleted,
                }"
                v-if="!value.isEditing"
              >
                {{ value.name }}
              </p>
              <input
                class="edit"
                v-model="value.name"
                @keydown.enter="checkEditingValue(value)"
                @blur="checkEditingValue(value)"
                @vnode-mounted="({ el }) => el.focus()"
                v-if="value.isEditing == true"
                placeholder="Do not leave this field blank"
              />
              <span
                class="tooltip"
                v-if="
                  this.countDuplicate(value.name) > 1 && value.isEditing == true
                "
              >
                Task existed
              </span>
            </label>
            <button class="destroy" @click="todoList.splice(index, 1)"></button>
          </div>
        </li>
      </ul>
    </section>
    <footer class="footer">
      <span class="todo-count">
        <strong> {{ remaining }} </strong>
        <span>{{ remaining === 0 ? " item" : " items" }} left</span>
      </span>
      <ul class="filters">
        <li>
          <a class="selected" name="filterList" @click="displayAll" href="#/"
            >All</a
          >
        </li>
        <li>
          <a
            class="selected"
            name="filterList"
            href="#/active"
            @click="displayActive"
            >Active</a
          >
        </li>
        <li>
          <a
            class="selected"
            name="filterList"
            href="#/completed"
            @click="displayCompleted"
          >
            Completed
          </a>
        </li>
        <li>
          <a class="selected" name="sort" href="#/sort" @click="sortList()">
            Sort
            <span>
              {{ this.sorted % 3 == 0 ? "" : this.sorted % 3 == 1 ? "⬇" : "⬆" }}
            </span>
          </a>
        </li>
      </ul>
      <button
        class="clear-completed"
        @click="clearCompleted"
        v-if="todoList.length > remaining"
      >
        Clear completed
      </button>
    </footer>
  </section>
</template>
