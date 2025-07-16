<template>
    <v-container>
        <v-row class="my-4">
          <h1><span class="text-blue">ToDo</span>-List</h1>
          <h2 class="mt-2 ml-16">{{ numberCompletedTasks }}/{{ numberTasks }} tasks completed</h2>
        </v-row>

        <v-btn append-icon="$plus" variant="outlined" class="mt-6 mb-10" @click="newTask">
          Add new Task
        </v-btn>

        <!-- Add new Task form -->
        <v-form v-model="valid" v-if="visible" class="mb-16">
        <v-container>
          <v-row>
            <v-col
              cols="12"
              md="4"
            >
              <v-text-field
                v-model="name"
                label="Name*"
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

            <v-col
              cols="12"
              md="4"
            >
              <v-select
              v-model="priority"
              clearable
              label="Priority"
              :items="['Low', 'Medium', 'High']"
            ></v-select>
            </v-col>
          </v-row>
          <v-row v-if="warning">
            <p class="ml-2 text-red">Please enter a name for the new task.</p>
          </v-row>
          <v-row>
            <v-btn class="mt-2" color="blue" @click="addNewTask(name, description, finish_until, priority)">Add new Task</v-btn>
          </v-row>
        </v-container>
        </v-form>


        <!-- Edit task form -->
        <v-form v-model="valid" v-if="editVisible" class="mb-16">
        <v-container>
          <v-row>
            <v-col
              cols="12"
              md="4"
            >
              <v-text-field
                v-model="name"
                label="Name*"
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

            <v-col
              cols="12"
              md="4"
            >
              <v-select
              v-model="priority"
              clearable
              label="Priority"
              :items="['Low', 'Medium', 'High']"
            ></v-select>
            </v-col>
          </v-row>
          <v-row v-if="warning">
            <p class="ml-2 text-red">Please enter a name for the new task.</p>
          </v-row>
          <v-row>
            <v-btn class="mt-2" color="blue" @click="editTaskPost(taskToChange, name, description, finish_until, priority)">Save changes</v-btn>
          </v-row>
        </v-container>
        </v-form>

        <h2 class="mb-4">Todo</h2>
        <v-data-table :headers="headers" :items="tasks" v-if="current" class="mb-16">
          <template v-slot:item="{ item }">
            <tr>
              <td class="text-black"><v-checkbox @click="completeTask(item)"></v-checkbox></td>
              <td class="text-black">{{ item.name }}</td>
              <td class="text-black">{{ item.description }}</td>
              <td class="text-black">{{ item.finish_until }}</td>
              <td class="text-black">{{ item.priority }}</td>
              <td class="text-black">{{ item.created_at }}</td>
              <td class="text-black"><v-btn icon="$edit" size="small" @click="editTask(item)"></v-btn></td>
              <td class="text-black"><v-btn icon="$delete" size="small" @click="deleteTask(item)"></v-btn></td>
            </tr>
          </template>
        </v-data-table>


        <h2 class="mb-4">Completed</h2>
        <v-data-table :headers="headers" :items="completedTasks" v-if="current" class="mb-16">
          <template v-slot:item="{ item }">
            <tr>
              <td class="text-black"><v-checkbox :model-value="true" @click="uncompleteTask(item)"></v-checkbox></td>
              <td class="text-black">{{ item.name }}</td>
              <td class="text-black">{{ item.description }}</td>
              <td class="text-black">{{ item.finish_until }}</td>
              <td class="text-black">{{ item.priority }}</td>
              <td class="text-black">{{ item.created_at }}</td>
              <td class="text-black"><v-btn icon="$delete" size="small" @click="deleteCompletedTask(item)"></v-btn></td>
            </tr>
          </template>
        </v-data-table>
    </v-container>
</template>

<script setup lang="ts">
  import { ref } from 'vue';
  import { useDate } from 'vuetify';

  const valid = ref(false);
  const date = useDate();
  let created_at = date.format(new Date(), 'fullDateTime');

  const name = ref('');
  const description = ref('');
  const finish_until = ref('');
  const priority = ref('');


  const headers: any = [
  { title: 'Done', align: 'start', key: 'done' },
  { title: 'Name', align: 'start', key: 'name' },
  { title: 'Description', align: 'start', key: 'description' },
  { title: 'Finish until', align: 'start', key: 'finish_until' },
  { title: 'Priority', align: 'start', key: 'priority' },
  { title: 'Created at', align: 'start', key: 'created_at' },
  {}, {},
  ];

  const tasks: any = ref([]);

  const completedTasks: any = ref([]);

  const visible = ref(false);
  const warning = ref(false);

  async function newTask() {
    warning.value = false;
    if(visible.value) {
        visible.value = false;
    }
    else {
      editVisible.value = false;
      await new Promise(f => setTimeout(f, 200));
      visible.value = true;
    }
  }

  function addNewTask(name, description, finish_until, priority) {
   if(name != '') {
    warning.value = false;
    created_at = date.format(new Date(), 'fullDateTime');

    tasks.value.push(
        {
          checked: false,
          name: name,
          description: description,
          finish_until: finish_until,
          priority: priority,
          created_at: created_at,
        }
    );

    numberTasks.value = tasks.value.length + numberCompletedTasks.value;
    current.value = false;
    current.value = true;
   }
   else {
    warning.value = true;
   }
  }

  const current = ref(true);

  function completeTask(task) {
    tasks.value.splice(tasks.value.indexOf(task), 1);
    completedTasks.value.push(task);
    numberCompletedTasks.value = completedTasks.value.length;
    numberTasks.value = tasks.value.length + numberCompletedTasks.value;
    current.value = false;
    current.value = true;
  }

  function uncompleteTask(task) {
    completedTasks.value.splice(tasks.value.indexOf(task), 1);
    tasks.value.push(task);
    numberCompletedTasks.value = completedTasks.value.length;
    numberTasks.value = tasks.value.length + numberCompletedTasks.value;
    current.value = false;
    current.value = true;
  }

  function deleteTask(task) {
    tasks.value.splice(tasks.value.indexOf(task), 1);
    numberTasks.value = tasks.value.length + numberCompletedTasks.value;
    current.value = false;
    current.value = true;
  }

  function deleteCompletedTask(task) {
    completedTasks.value.splice(completedTasks.value.indexOf(task), 1);
    numberCompletedTasks.value = completedTasks.value.length;
    numberTasks.value = tasks.value.length + numberCompletedTasks.value;
    current.value = false;
    current.value = true;
  }

  const numberCompletedTasks = ref(completedTasks.value.length);
  const numberTasks = ref(tasks.value.length + numberCompletedTasks.value);


  // Edit Tasks
  const editVisible = ref(false);
  const editWarning = ref(false);

  const taskToChange = ref();

   async function editTask(task) {
    editWarning.value = false;
    if(editVisible.value) {
      editVisible.value = false;
    }
    else {
      visible.value = false;
      await new Promise(f => setTimeout(f, 200));
      editVisible.value = true;
    }

    taskToChange.value = task;
  }

  function editTaskPost(task, name, description, finish_until, priority) {
    if(name != '') {
      editWarning.value = false;

      tasks.value[tasks.value.indexOf(task)].name = name;
      tasks.value[tasks.value.indexOf(task)].description = description;
      tasks.value[tasks.value.indexOf(task)].finish_until = finish_until;
      tasks.value[tasks.value.indexOf(task)].priority = priority;
      
      current.value = false;
      current.value = true;
   }
   else {
      editWarning.value = true;
   }
  }

  //Edit completed Task
  const completedTaskToChange = ref();

  async function editCompletedTask(task) {
    editWarning.value = false;
    if(editVisible.value) {
      editVisible.value = false;
    }
    else {
      visible.value = false;
      await new Promise(f => setTimeout(f, 200));
      editVisible.value = true;
    }

    completedTaskToChange.value = task;
  }

  function editCompletedTaskPost(task, name, description, finish_until, priority) {
    if(name != '') {
      editWarning.value = false;

      completedTasks.value[completedTasks.value.indexOf(task)].name = name;
      completedTasks.value[completedTasks.value.indexOf(task)].description = description;
      completedTasks.value[completedTasks.value.indexOf(task)].finish_until = finish_until;
      completedTasks.value[completedTasks.value.indexOf(task)].priority = priority;

      current.value = false;
      current.value = true;
    }
    else {
      editWarning.value = true;
    }
  }
</script>
