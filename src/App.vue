<script setup>
import { ref, onMounted, computed, watch } from "vue";

const todos = ref([]);
const name = ref("");

const input_content = ref("");
const input_category = ref(null);
const filterStatus = ref("all");

const todos_filtered = computed(() => {
  let filteredTodos = todos.value;

  if (filterStatus.value === "completed") {
    filteredTodos = filteredTodos.filter((todo) => todo.done);
  } else if (filterStatus.value === "incomplete") {
    filteredTodos = filteredTodos.filter((todo) => !todo.done);
  }
  //Filtrer les todos selon le statut

  return filteredTodos.sort((a, b) => b.createdAt - a.createdAt);
});
//Trier les todos par date de creation

const addTodo = () => {
  if (input_content.value.trim() === "" || input_category.value === null) {
    return;
  }

  todos.value.push({
    content: input_content.value,
    category: input_category.value,
    done: false,
    createdAt: new Date().getTime(),
    isEditing: false,
  });

  input_content.value = "";
  input_category.value = null;
};
//Ajouter un todo a la liste

const editTodo = (todo) => {
  todo.isEditing = true;
};
//Editer un todo

const saveTodo = (todo) => {
  todo.isEditing = false;
};
//Sauvegarder un todo

const removeTodo = (todo) => {
  const index = todos.value.indexOf(todo);
  todos.value.splice(index, 1);
};
//Supprimer un todo

watch(
  todos,
  (newVal) => {
    localStorage.setItem("todos", JSON.stringify(newVal));
  },
  { deep: true }
);

watch(name, (newVal) => {
  localStorage.setItem("name", newVal);
});
//Sauvegarder les todos et le nom dans le local storage a chaque changement

onMounted(() => {
  name.value = localStorage.getItem("name") || "";
  todos.value = JSON.parse(localStorage.getItem("todos")) || [];
});
//Recuperer les todos et le nom du local storage au montage
</script>

<template>
  <main class="app">
    <h1 class="title1">TO DO LIST</h1>
    <section class="greeting">
      <h2 class="title">
        Hello, <input type="text" placeholder="Name here" v-model="name" />
      </h2>
    </section>

    <section class="create-todo">
      <h3>Create a todo</h3>

      <form @submit.prevent="addTodo">
        <h4>What's on your todo list ?</h4>
        <input
          type="text"
          placeholder="Buy some milk"
          v-model="input_content"
        />

        <h4>Pick a category</h4>
        <div class="options">
          <label>
            <input
              type="radio"
              name="category"
              value="business"
              v-model="input_category"
            />
            <span class="bubble business"></span>
            <div>Business</div>
          </label>

          <label>
            <input
              type="radio"
              name="category"
              value="personal"
              v-model="input_category"
            />
            <span class="bubble personal"></span>
            <div>Personal</div>
          </label>
        </div>

        <input type="submit" value="Add todo" />
      </form>
    </section>

    <section class="todo-list">
      <h3>TODO LIST</h3>
      <div class="filter-todos">
        <div class="options">
          <label>
            <input
              type="radio"
              name="filter"
              value="all"
              v-model="filterStatus"
            />
            <div>All</div>
          </label>

          <label>
            <input
              type="radio"
              name="filter"
              value="completed"
              v-model="filterStatus"
            />
            <div>Completed</div>
          </label>

          <label>
            <input
              type="radio"
              name="filter"
              value="incomplete"
              v-model="filterStatus"
            />
            <div>Incomplete</div>
          </label>
        </div>
      </div>
      <div class="list">
        <div
          v-for="todo in todos_filtered"
          :class="`todo-item ${todo.done && 'done'}`"
          :key="todo.createdAt"
        >
          <label>
            <input type="checkbox" v-model="todo.done" />
            <span :class="`bubble ${todo.category}`"></span>
          </label>

          <div class="todo-content" :class="{ done: todo.done }">
            <input v-if="todo.isEditing" type="text" v-model="todo.content" />
            <div v-else>{{ todo.content }}</div>
          </div>

          <div class="actions">
            <button v-if="!todo.isEditing" @click="editTodo(todo)" class="edit">
              EDIT
            </button>
            <button v-if="todo.isEditing" @click="saveTodo(todo)" class="save">
              SAVE
            </button>
            <button class="delete" @click="removeTodo(todo)">DELETE</button>
          </div>
        </div>
      </div>
    </section>
  </main>
</template>
