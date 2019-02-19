<template>
  <div class="app">
    <div class="container">
      <div class="row header">
        <h1 class="col s8 offset-s2 center-align teal-text">To-Do List!</h1>
      </div>
      <div class="row">
        <form @submit.prevent="submitTodo" class="col s8 offset-s2">
          <div class="input-field">
            <i class="material-icons prefix">list</i>
            <textarea
              v-model="newTodo"
              v-on:keydown.enter.prevent="submitTodo"
              id="icon_prefix2"
              class="materialize-textarea"
            ></textarea>
            <label for="icon_prefix2">What to do?</label>
          </div>
          <button class="btn waves-effect col s12">Add</button>
        </form>
      </div>
      <div class="row">
        <ul class="collection col s8 offset-s2">
          <template v-if="todos">
            <li class="collection-item" v-for="i in todos.size()">
              <p>
                <label>
                  <input
                    type="checkbox"
                    :checked="todos.get(i-1).completed"
                    @change="updateTodoCompleted(todos.get(i-1).id, !todos.get(i-1).completed)"
                  >
                  <span>{{todos.get(i-1).label}}</span>
                  <span>
                    <a @click.prevent="deleteTodo(todos.get(i-1).id)">
                      <i class="material-icons right teal-text">delete</i>
                    </a>
                  </span>
                </label>
              </p>
            </li>
          </template>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
function consoleLogBold(text) {
  console.log("%c" + text, "font-weight: bold;");
}

export default {
  name: "app",
  data() {
    return {
      todos: null,
      newTodo: "",
      todoListDB: null
    };
  },
  watch: {
    todos: {
      handler() {
        localStorage.todos = JSON.stringify(this.todos);
      },
      deep: true
    }
  },
  mounted() {
    window.addEventListener("wasmLoaded", () => {
      consoleLogBold("WebAssembly loaded");

      // instantiate our C++ implementation
      this.todoListDB = new window.Module.TodoList(".");
      this.todos = this.todoListDB.get_todos();

      // print the initial list
      consoleLogBold("Initial Todos:");
      this.logTodos();
    });
  },
  methods: {
    submitTodo() {
      consoleLogBold("Add Todo:");
      this.todoListDB.add_todo(this.newTodo);
      this.todos = this.todoListDB.get_todos();
      this.logTodos();
      this.newTodo = "";
    },
    deleteTodo(todoId) {
      consoleLogBold("Delete Todo:");
      this.todoListDB.delete_todo(todoId);
      this.todos = this.todoListDB.get_todos();
      this.logTodos();
    },
    updateTodoCompleted(todoId, completed) {
      consoleLogBold("Update Todo Status:");
      this.todoListDB.update_todo_completed(todoId, completed);
      this.todos = this.todoListDB.get_todos();
      this.logTodos();
    },
    logTodos() {
      const vectorData = this.todos;
      for (var i = 0; i < vectorData.size(); i++) {
        const todo = vectorData.get(i);
        console.log(todo.id + ". " + todo.label + " (" + todo.completed + ")");
      }
    }
  }
};
</script>

<style lang="scss">
.header {
  margin-top: 20px;
}
</style>
