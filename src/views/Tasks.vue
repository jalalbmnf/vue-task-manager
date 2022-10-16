<script setup lang="ts">
import { onMounted, ref, computed, watch } from "vue";

import { ITask } from "../ts/models/tasks";

const taskTableHeader = [
  "Title",
  "Description",
  "Status",
  "Change status",
  "Actions",
];
const tasks = ref<ITask[]>([]);
const title = ref("");
const selectedTags = ref<string[]>([]);
const description = ref("");
const date = new Date().getTime();
const titleValidate = ref(false);
const descriptionValidate = ref(false);

const addTask = () => {
  if (title.value.trim() === "") {
    titleValidate.value = true;
  }
  if (description.value.trim() === "") {
    descriptionValidate.value = true;
  } else {
    tasks.value.push({
      title: title.value,
      description: description.value,
      status: false,
      date,
      tag: selectedTags.value,
    });
    titleValidate.value = false;
    descriptionValidate.value = false;
    title.value = "";
    description.value = "";
  }
};

const sortedTasks = computed(() =>
  tasks.value.sort((a: ITask, b: ITask) => {
    return a.date - b.date;
  })
);

const removeTask = (task: ITask) => {
  tasks.value = tasks.value.filter((t: ITask) => t !== task);
};

watch(
  tasks,
  (tasks) => {
    localStorage.setItem("tasks", JSON.stringify(tasks));
  },
  { deep: true }
);

onMounted(() => {
  tasks.value = JSON.parse(localStorage.getItem("tasks") as string) || [];
});
</script>

<template>
  <main class="tasks-container">
    <h1 class="tasks-title">Task Managment Application</h1>
    <form class="tasks-form" @submit.prevent="addTask">
      <input
        class="tasks-form__input"
        :class="{ error: titleValidate }"
        placeholder="Task title"
        v-model="title"
      />

      <input
        class="tasks-form__input"
        :class="{ error: descriptionValidate }"
        placeholder="Task description"
        v-model="description"
      />

      <button class="tasks-form__btn" type="submit">Submit</button>
    </form>
    <table v-if="!!sortedTasks.length" class="tasks-table">
      <thead class="tasks-table__header">
        <tr class="tasks-table__header-row">
          <th class="tasks-table__header-col" v-for="header in taskTableHeader">
            {{ header }}
          </th>
        </tr>
      </thead>
      <tbody class="tasks-table__body">
        <tr class="tasks-table__body-row" v-for="task in sortedTasks">
          <td class="tasks-table__body-col">
            <input type="text" v-model="task.title" />
          </td>
          <td class="tasks-table__body-col">
            <input
              type="text"
              v-model="task.description"
              :title="task.description"
            />
          </td>
          <td
            class="tasks-table__body-col"
            :style="{
              color: task.status ? '#23395d' : '#f5Ca7b',
            }"
          >
            {{ task.status ? "Done" : "Waiting" }}
          </td>
          <td class="tasks-table__body-col">
            <input
              class="tasks-table__checkbox"
              type="checkbox"
              v-model="task.status"
            />
          </td>
          <td class="tasks-table__body-col">
            <button class="tasks-table__btn" @click="removeTask(task)">
              Delete
            </button>
          </td>
        </tr>
      </tbody>
    </table>
    <h2 v-else class="no-data">You have to add task.</h2>
    <div class="tips">
      * You can edit task title or description by clicking to them.
    </div>
  </main>
</template>
<style lang="scss" src="./Tasks.scss"></style>
