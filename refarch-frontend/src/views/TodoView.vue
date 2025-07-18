<template>
  <v-container>
    <v-row class="my-4">
      <h1><span class="text-blue">ToDo</span>-List</h1>
      <h2 class="mt-2 ml-16">{{ numberCompletedTasks }}/{{ numberTasks }} tasks completed</h2>
      <v-progress-linear bg-color="blue-lighten-1" color="blue" :model-value="progress"
        v-if="current"></v-progress-linear>
    </v-row>

    <v-row>
      <v-btn append-icon="$plus" variant="outlined" class="mt-6 mb-10 mr-6" @click="newTask">
        Add new task
      </v-btn>

      <v-btn append-icon="$plus" variant="outlined" class="mt-6 mb-10" @click="newCategory">
        Add new category
      </v-btn>
    </v-row>

    <!-- Add new Task form -->
    <v-form v-model="valid" v-if="visible" class="mb-16">
      <v-container>
        <v-row>
          <v-col cols="12" md="4">
            <v-text-field v-model="name" label="Title*" required></v-text-field>
          </v-col>

          <v-col cols="12" md="4">
            <v-text-field v-model="description" label="Description"></v-text-field>
          </v-col>

          <v-col cols="12" md="4">
            <v-date-input v-model="finish_until" label="Finish until" clearable
              display-format="fullDateWithWeekday"></v-date-input>
          </v-col>

          <v-col cols="12" md="4">
            <v-select v-model="priority" clearable label="Priority" :items="['Low', 'Medium', 'High']"></v-select>
          </v-col>

          <v-col cols="12" md="4">
            <v-select v-model="categories" clearable chips label="Categories" :items="allCategoryNames" multiple
              no-data-text="No categories added yet"></v-select>
          </v-col>
        </v-row>
        <v-row v-if="warning">
          <p class="ml-2 text-red">Please enter a title for the new task.</p>
        </v-row>
        <v-row>
          <v-btn class="mt-2" color="blue"
            @click="addNewTask(name, description, finish_until, priority, categories)">Add new Task</v-btn>
        </v-row>
      </v-container>
    </v-form>


    <!-- Edit task form -->
    <v-form v-model="valid" v-if="editVisible" class="mb-16">
      <v-container>
        <v-row>
          <v-col cols="12" md="4">
            <v-text-field v-model="name" label="Title*" required></v-text-field>
          </v-col>

          <v-col cols="12" md="4">
            <v-text-field v-model="description" label="Description"></v-text-field>
          </v-col>

          <v-col cols="12" md="4">
            <v-date-input v-model="finish_until" label="Finish until" clearable
              display-format="fullDateWithWeekday"></v-date-input>
          </v-col>

          <v-col cols="12" md="4">
            <v-select v-model="priority" clearable label="Priority" :items="['Low', 'Medium', 'High']"></v-select>
          </v-col>

          <v-col cols="12" md="4">
            <v-select v-model="categories" clearable chips label="Categories" :items="allCategoryNames" multiple
              no-data-text="No categories added yet"></v-select>
          </v-col>
        </v-row>
        <v-row v-if="editWarning">
          <p class="ml-2 text-red">Please enter a title for the task.</p>
        </v-row>
        <v-row>
          <v-btn class="mt-2" color="blue"
            @click="editTaskPost(taskToChange, name, description, finish_until, priority, categories)">Save
            changes</v-btn>
        </v-row>
      </v-container>
    </v-form>

    <!-- Add new category form -->
    <v-form v-model="valid" v-if="categoryVisible" class="mb-16">
      <v-container>
        <v-row>
          <v-col cols="12" md="4">
            <v-text-field v-model="categoryName" label="Category name*" required></v-text-field>
          </v-col>

          <v-col cols="12" md="4">
            <v-select v-model="bgColor" clearable label="Color"
              :items="['blue', 'red', 'green', 'pink', 'purple', 'yellow', 'orange', 'grey']"></v-select>
          </v-col>
        </v-row>
        <v-row v-if="categoryWarning">
          <p class="ml-2 text-red">Please enter a name for the new category.</p>
        </v-row>
        <v-row>
          <v-btn class="mt-2" color="blue" @click="addNewCategory(categoryName, bgColor)">Add new category</v-btn>
        </v-row>
      </v-container>
    </v-form>


    <!-- Tables -->
    <h2 class="mb-4">Todo</h2>
    <v-data-table :headers="headers" :items="tasks" v-if="current" class="mb-16" no-data-text="No tasks added yet">
      <template v-slot:item="{ item }">
        <tr>
          <td><v-checkbox @click="completeTask(item)"></v-checkbox></td>
          <td>{{ item.name }}</td>
          <td>{{ item.description }}</td>
          <td>{{ item.finish_until }}</td>
          <td>{{ item.priority }}</td>
          <td><span v-for="category in item.categories"
              :class="['mx-2', 'py-1', 'px-2', 'rounded', categoryColor(category), 'bg-' + selectedBgColor]">{{ category
              }}</span></td>
          <td>{{ item.created_at }}</td>
          <td><v-btn icon="$edit" size="small" @click="editTask(item)"></v-btn></td>
          <td><v-btn icon="$delete" size="small" @click="deleteTask(item)"></v-btn></td>
        </tr>
      </template>
    </v-data-table>

    <h2 class="mb-4">Completed</h2>
    <v-data-table :headers="headers" :items="completedTasks" v-if="current" class="mb-16"
      no-data-text="No tasks added yet">
      <template v-slot:item="{ item }">
        <tr>
          <td class="text-black"><v-checkbox :model-value="true" @click="uncompleteTask(item)"></v-checkbox></td>
          <td class="text-black">{{ item.name }}</td>
          <td class="text-black">{{ item.description }}</td>
          <td class="text-black">{{ item.finish_until }}</td>
          <td class="text-black">{{ item.priority }}</td>
          <td class="text-black"><span v-for="category in item.categories"
              :class="['mx-2', 'py-1', 'px-2', 'rounded', categoryColor(category), 'bg-' + selectedBgColor]">{{
                category }}</span></td>
          <td class="text-black">{{ item.created_at }}</td>
          <td class="text-black"><v-btn icon="$edit" size="small" @click="editCompletedTask(item)"></v-btn></td>
          <td class="text-black"><v-btn icon="$delete" size="small" @click="deleteCompletedTask(item)"></v-btn></td>
        </tr>
      </template>
    </v-data-table>

    <div class="pt-10">
      <v-sheet>
        <v-calendar ref="calendar" :events="events" view-mode="month"></v-calendar>
      </v-sheet>
    </div>
  </v-container>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import { useDate } from 'vuetify';
import { VDateInput } from "vuetify/labs/components";
import { VCalendar } from 'vuetify/labs/VCalendar';

const valid = ref(false);
const date = useDate();
let created_at = date.format(new Date(), 'fullDateTime');

const name = ref('');
const description = ref('');
const finish_until = ref('');
const priority = ref('');

const categoryName = ref('');
const categories = ref([]);
const bgColor = ref('');

const events = ref([]);


const headers: any = [
  { title: 'Done', align: 'start', key: 'done' },
  { title: 'Title', align: 'start', key: 'name' },
  { title: 'Description', align: 'start', key: 'description' },
  { title: 'Finish until', align: 'start', key: 'finish_until' },
  { title: 'Priority', align: 'start', key: 'priority' },
  { title: 'Categories', align: 'start', key: 'categories' },
  { title: 'Created at', align: 'start', key: 'created_at' },
  {}, {},
];

const tasks: any = ref([]);

const completedTasks: any = ref([]);

const visible = ref(false);
const warning = ref(false);

async function newTask() {
  warning.value = false;
  if (visible.value) {
    visible.value = false;
  }
  else {
    editVisible.value = false;
    categoryVisible.value = false;
    await new Promise(f => setTimeout(f, 200));
    visible.value = true;
  }
}

const taskId = ref(1);

const finish_date = ref('');
const created_date = ref('');
const cal_date = ref('');

function addNewTask(new_name, new_description, new_finish_until, new_priority, new_categories) {
  if (new_name != '') {
    warning.value = false;
    created_at = new Date();
    created_date.value = date.format(created_at, 'fullDateTime');

    finish_date.value = date.format(new_finish_until, 'fullDateWithWeekday');

    tasks.value.push(
      {
        id: taskId.value,
        checked: false,
        name: new_name,
        description: new_description,
        finish_until: finish_date.value,
        priority: new_priority,
        categories: new_categories,
        created_at: created_date,
      }
    );

    if (new_finish_until != null) {
      cal_date.value = new_finish_until;
    }
    else {
      cal_date.value = created_at;
    }

    categoryColor(categories[0]);

    events.value.push(
      {
        id: taskId.value,
        title: new_name,
        start: cal_date.value,
        end: cal_date.value,
        color: selectedBgColor,
        allDay: true,
      }
    )

    name.value = '';
    description.value = '';
    finish_until.value = '';
    priority.value = '';
    categories.value = [];

    numberTasks.value = tasks.value.length + numberCompletedTasks.value;
    current.value = false;
    current.value = true;

    taskId.value++;
    updateProgressBar();
  }
  else {
    warning.value = true;
  }
}

const current = ref(true);

function completeTask(task) {
  tasks.value.splice(tasks.value.indexOf(task), 1);
  completedTasks.value.push(task);
  completedTasks.value[completedTasks.value.indexOf(task)].checked = true;
  numberCompletedTasks.value = completedTasks.value.length;
  numberTasks.value = tasks.value.length + numberCompletedTasks.value;

  
  current.value = false;
  current.value = true;

  updateProgressBar();
}

function uncompleteTask(task) {
  completedTasks.value.splice(tasks.value.indexOf(task), 1);
  tasks.value.push(task);
  tasks.value[tasks.value.indexOf(task)].checked = false;
  numberCompletedTasks.value = completedTasks.value.length;
  numberTasks.value = tasks.value.length + numberCompletedTasks.value;
  current.value = false;
  current.value = true;

  updateProgressBar();
}

function deleteTask(task) {
  tasks.value.splice(tasks.value.indexOf(task), 1);
  numberTasks.value = tasks.value.length + numberCompletedTasks.value;

  events.value.forEach(event => {
    if(event.id == task.id) {
      events.value.splice(events.value.indexOf(event), 1);
    }
  });

  current.value = false;
  current.value = true;
  updateProgressBar();
}

function deleteCompletedTask(task) {
  completedTasks.value.splice(completedTasks.value.indexOf(task), 1);
  numberCompletedTasks.value = completedTasks.value.length;
  numberTasks.value = tasks.value.length + numberCompletedTasks.value;

  events.value.forEach(event => {
    if(event.id == task.id) {
      events.value.splice(events.value.indexOf(event), 1);
    }
  });

  current.value = false;
  current.value = true;
  updateProgressBar();
}

const numberCompletedTasks = ref(completedTasks.value.length);
const numberTasks = ref(tasks.value.length + numberCompletedTasks.value);
const progress = ref(100);

function updateProgressBar() {
  if (numberTasks.value == 0) {
    progress.value = 100;
  }
  else {
    progress.value = (numberCompletedTasks.value / numberTasks.value) * 100;
  }
}


// Edit Tasks
const editVisible = ref(false);
const editWarning = ref(false);

const taskToChange = ref();

async function editTask(task) {
  editWarning.value = false;
  if (editVisible.value) {
    editVisible.value = false;
  }
  else {
    visible.value = false;
    categoryVisible.value = false;
    await new Promise(f => setTimeout(f, 200));
    editVisible.value = true;
  }

  taskToChange.value = task;

  name.value = task.name;
  description.value = task.description;
  finish_until.value = task.finish_until;
  priority.value = task.priority;
  categories.value = task.categories;
}

function editTaskPost(task, name, description, finish_until, priority, categories) {
  if (name != '') {
    finish_date.value = date.format(finish_until, 'fullDateWithWeekday');

    if (task.checked) {
      editWarning.value = false;

      completedTasks.value[completedTasks.value.indexOf(task)].name = name;
      completedTasks.value[completedTasks.value.indexOf(task)].description = description;
      completedTasks.value[completedTasks.value.indexOf(task)].finish_until = finish_date.value;
      completedTasks.value[completedTasks.value.indexOf(task)].priority = priority;
      completedTasks.value[completedTasks.value.indexOf(task)].categories = categories;

      events.value.forEach(event => {
        if (finish_until != null) {
          cal_date.value = finish_until;
        }
        else {
          cal_date.value = completedTasks.value[completedTasks.value.indexOf(task)].created_at;
        }

        categoryColor(categories[0]);

        if(task.id == event.id) {
          events.value[events.value.indexOf(event)].title = name;
          events.value[events.value.indexOf(event)].start = cal_date.value;
          events.value[events.value.indexOf(event)].end = cal_date.value;
          events.value[events.value.indexOf(event)].color = selectedBgColor;
        }
      });
    }
    else {
      editWarning.value = false;

      tasks.value[tasks.value.indexOf(task)].name = name;
      tasks.value[tasks.value.indexOf(task)].description = description;
      tasks.value[tasks.value.indexOf(task)].finish_until = finish_date.value;
      tasks.value[tasks.value.indexOf(task)].priority = priority;
      tasks.value[tasks.value.indexOf(task)].categories = categories;

      events.value.forEach(event => {
        if (finish_until != null) {
          cal_date.value = finish_until;
        }
        else {
          cal_date.value = tasks.value[tasks.value.indexOf(task)].created_at;
        }

        categoryColor(categories[0]);

        if(task.id == event.id) {
          events.value[events.value.indexOf(event)].title = name;
          events.value[events.value.indexOf(event)].start = cal_date.value;
          events.value[events.value.indexOf(event)].end = cal_date.value;
          events.value[events.value.indexOf(event)].color = selectedBgColor;
        }
      });
    }

    name.value = '';
    description.value = '';
    finish_until.value = '';
    priority.value = '';
    categories.value = '';

    current.value = false;
    current.value = true;
  }
  else {
    editWarning.value = true;
  }
}

//Edit completed Task

async function editCompletedTask(task) {
  editWarning.value = false;
  if (editVisible.value) {
    editVisible.value = false;
  }
  else {
    visible.value = false;
    categoryVisible.value = false;
    await new Promise(f => setTimeout(f, 200));
    editVisible.value = true;
  }

  taskToChange.value = task;
}


// Categories
const categoryVisible = ref(false);
const categoryWarning = ref(false);

const allCategories = ref([]);
const allCategoryNames = ref([]);

async function newCategory() {
  categoryWarning.value = false;
  if (categoryVisible.value) {
    categoryVisible.value = false;
  }
  else {
    visible.value = false;
    editVisible.value = false;
    await new Promise(f => setTimeout(f, 200));
    categoryVisible.value = true;
  }
}

function addNewCategory(name, color) {
  if (name != '') {
    categoryWarning.value = false;

    allCategories.value.push(
      {
        name: name,
        bgColor: color,
      },
    );

    allCategoryNames.value.push(name);

    categoryName.value = '';
    bgColor.value = '';
  }
  else {
    categoryWarning.value = true;
  }
}

const selectedBgColor = ref('');

function categoryColor(selectedCategory) {
  allCategories.value.forEach(category => {
    if (category.name == selectedCategory) {
      selectedBgColor.value = category.bgColor;
    }
  });
}
</script>
