<script setup>
import { ref } from "vue";
import axios from "axios";

let newTodos = ref("");
let todos = ref([]);

let completedTodos = ref([]);

// Retrieve todos from the server
const fetchData = async () => {
    try {
        const response = await axios.get("/api/todos");
        const allTodos = response.data;

        todos.value = allTodos.filter((todo) => todo.completed === 0);
        completedTodos.value = allTodos.filter((todo) => todo.completed === 1);
    } catch (error) {
        console.error(error);
    }
};

let addTodos = async () => {
    try {
        const response = await axios.post("/api/todos", {
            title: newTodos.value,
        });
        todos.value.push(response.data);
        newTodos.value = "";
    } catch (error) {
        console.error(error);
    }
};

const deleteTodo = async (todo) => {
    try {
        await axios.delete(`/api/todos/${todo.id}`);
        todos.value = todos.value.filter((e) => e !== todo);
    } catch (error) {
        console.error(error);
    }
};

const deleteComplete = async (todo) => {
    try {
        await axios.delete(`/api/todos/${todo.id}`);
        completedTodos.value = completedTodos.value.filter((e) => e !== todo);
    } catch (error) {
        console.error(error);
    }
};

const deleteAll = async () => {
    try {
        await axios.delete("/api/todos");

        todos.value = [];
    } catch (error) {
        console.error(error);
    }
};

const completed = async (todo) => {
    try {
        const response = await axios.put(`/api/todos/${todo.id}`, {
            completed: true,
        });

        // Update todos and completedTodos lists
        todos.value = todos.value.filter((e) => e.id !== todo.id);
        completedTodos.value.push(response.data);
    } catch (error) {
        console.error(error);
    }
};

const undoCompleteTodo = async (todo) => {
    try {
        const response = await axios.put(`/api/todos/${todo.id}`, {
            completed: false,
        });
        todos.value.push(response.data);
        completedTodos.value = completedTodos.value.filter((e) => e !== todo);
    } catch (error) {
        console.error(error);
    }
};

fetchData();
</script>

<template>
    <div class="d-flex justify-content-center align-items-center">
        <div class="card" style="width: 30rem">
            <div class="card-body">
                <h5 class="card-title">Todo APP</h5>
                <div>
                    <form @submit.prevent="addTodos" class="d-flex flex-row">
                        <input
                            type="text"
                            v-model="newTodos"
                            class="form-control me-2"
                            name="title"
                            placeholder="add your new todo"
                        />
                        <button class="btn btn-primary">+</button>
                    </form>
                </div>
                <p
                    class="border border-primary fw-bold text-center mt-3"
                    v-if="todos.length <= 0"
                >
                    TODO list is empty!
                </p>
                <ul class="list-group mt-5">
                    <li
                        class="list-group-item"
                        v-for="item in todos"
                        :key="item.id"
                    >
                        <span>{{ item.title }}</span>
                        <button
                            class="btn btn-sm btn-danger float-end ms-2"
                            @click="deleteTodo(item)"
                        >
                            -
                        </button>
                        <button
                            class="btn btn-sm btn-success float-end"
                            @click="completed(item)"
                        >
                            &check;
                        </button>
                    </li>
                </ul>
                <p class="mt-3">You have {{ todos.length }} task left</p>
                <button class="btn btn-danger btn-sm" @click="deleteAll">
                    Clear all
                </button>
            </div>
        </div>
    </div>

    <div class="d-flex justify-content-center align-items-center ms-4">
        <div class="card" style="width: 30rem">
            <div class="card-body">
                <h5 class="card-title">Completed TODO</h5>
                <p
                    class="border border-primary fw-bold text-center mt-3"
                    v-if="completedTodos.length <= 0"
                >
                    Empty List
                </p>
                <ul class="list-group mt-5">
                    <li
                        class="list-group-item"
                        v-for="item in completedTodos"
                        :key="item.id"
                    >
                        <span class="text-decoration-line-through">{{
                            item.title
                        }}</span>
                        <button
                            class="btn btn-sm btn-danger float-end ms-2"
                            @click="deleteComplete(item)"
                        >
                            -
                        </button>
                        <button
                            class="btn btn-sm btn-warning float-end"
                            @click="undoCompleteTodo(item)"
                        >
                            Undo
                        </button>
                    </li>
                </ul>
                <p class="mt-3">
                    You have {{ completedTodos.length }} completed task
                </p>
            </div>
        </div>
    </div>
</template>
