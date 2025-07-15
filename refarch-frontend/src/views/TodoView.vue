<template>
    <v-container>
        <h1 class="my-4"><span class="text-blue">ToDo</span>-List</h1>

        <v-btn append-icon="$plus" variant="outlined" class="mt-6 mb-10" @click="newTask">
          Add new Task
        </v-btn>

        <v-form v-model="valid" v-if="visible" class="mb-16">
        <v-container>
          <v-row>
            <v-col
              cols="12"
              md="4"
            >
              <v-text-field
                v-model="name"
                label="Name"
                required
              ></v-text-field>
            </v-col>

            <v-col
              cols="12"
              md="4"
            >
              <v-text-field
                v-model="description"
                label="Description"
              ></v-text-field>
            </v-col>

            <v-col
              cols="12"
              md="4"
            >
              <v-text-field
                v-model="finish_until"
                label="Finish until"
              ></v-text-field>
            </v-col>
          </v-row>
          <v-row v-if="warning">
            <p class="ml-2 text-red">Please enter a name for the new task.</p>
          </v-row>
          <v-row>
            <v-btn class="mt-2" color="blue" @click="addNewTask(name, description, finish_until)">Add new Task</v-btn>
          </v-row>
        </v-container>
        </v-form>


        <h2 clas="mb-4">Todo</h2>
        <v-table v-if="current" class="mb-16">
          <thead>
            <tr class="text-blue">
              <th class="text-left">
                Done
              </th>
              <th class="text-left">
                Name
              </th>
              <th class="text-left">
                Description
              </th>
              <th class="text-left">
                Finish until
              </th>
              <th class="text-left">
                Created at
              </th>
            </tr>
          </thead>
          <tbody>
            <tr
              v-for="task in tasks"
              :key="task.name"
            >
              <td><v-checkbox @click="completeTask(task)"></v-checkbox></td>
              <td>{{ task.name }}</td>
              <td>{{ task.description }}</td>
              <td>{{ task.finish_until }}</td>
              <td>{{ task.created_at }}</td>
            </tr>
          </tbody>
        </v-table>


        <h2 class="mb-4">Completed</h2>
        <v-table>
          <thead>
            <tr class="text-blue">
              <th class="text-left">
                Done
              </th>
              <th class="text-left">
                Name
              </th>
              <th class="text-left">
                Description
              </th>
              <th class="text-left">
                Finish until
              </th>
              <th class="text-left">
                Created at
              </th>
            </tr>
          </thead>
          <tbody>
            <tr
              v-for="task in completedTasks"
              :key="task.name"
            >
              <td><v-checkbox :model-value="true" @click="uncompleteTask(task)"></v-checkbox></td>
              <td>{{ task.name }}</td>
              <td>{{ task.description }}</td>
              <td>{{ task.finish_until }}</td>
              <td>{{ task.created_at }}</td>
            </tr>
          </tbody>
        </v-table>

    </v-container>
</template>

<script setup>
  import { computed, ref } from 'vue';

  let created_at = new Date();

  const name = ref('');
  const description = ref('');
  const finish_until = ref('');

  const tasks = [];

  const completedTasks = [];

  const visible = ref(false);
  const warning = ref(false);

  function newTask() {
    warning.value = false;
    if(visible.value) {
        visible.value = false;
    }
    else {
        visible.value = true;
    }
  }

  function addNewTask(name, description, finish_until) {
   if(name != undefined) {
    warning.value = false;
    created_at = new Date();

    tasks.push(
        {
          checked: false,
          name: name,
          description: description,
          finish_until: finish_until,
          created_at: created_at,
        }
    );

    current.value = false;
    current.value = true;
   }
   else {
    warning.value = true;
   }
  }

  const current = ref(true);

  function completeTask(task) {
    tasks.splice(tasks.indexOf(task), 1);
    completedTasks.push(task);

    current.value = false;
    current.value = true;
  }

  function uncompleteTask(task) {
    completedTasks.splice(tasks.indexOf(task), 1);
    tasks.push(task);

    current.value = false;
    current.value = true;
  }
</script>
