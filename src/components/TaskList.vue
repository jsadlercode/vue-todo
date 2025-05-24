<script setup lang="ts">
import taskItem from './taskitem.vue';
import { ref } from 'vue';

const lists = ref([
    { id: 1, name: 'To Do' },
    { id: 2, name: 'Doing' },
    { id: 3, name: 'Done' },
]);

const tasks = ref([
    {
        id: 1,
        title: 'Task 1',
        description: 'Description for Task 1',
        listId: 1,
    },
    {
        id: 2,
        title: 'Task 2',
        description: 'Description for Task 2',
        listId: 2,
    },
    {
        id: 3,
        title: 'Task 3',
        description: 'Description for Task 3',
        listId: 1,
    },
    {
        id: 4,
        title: 'Task 4',
        description: 'Description for Task 4',
        listId: 3,
    },
    {
        id: 5,
        title: 'Task 5',
        description: 'Description for Task 5',
        listId: 2,
    },
    {
        id: 6,
        title: 'Task 6',
        description: 'Description for Task 6',
        listId: 1,
    },
]);

const newTaskTitle = ref('');

const getTasksForList = (listIdValue: number) => {
    return tasks.value.filter(task => task.listId === listIdValue);
};

const startDrag = (event: DragEvent, item) => {
    console.log('dragging', item);
    if (event.dataTransfer) {
        event.dataTransfer.dropEffect = 'move';
        event.dataTransfer.effectAllowed = 'move';
        event.dataTransfer.setData('itemID', item.id.toString());
    }
};

const addNewTask = () => {
    if (newTaskTitle.value.trim() === '') {
        return;
    }
    const newTask = {
        id: tasks.value.length + 1,
        title: newTaskTitle.value,
        description: 'New task description',
        listId: 1, // Default to 'To Do' list
    };
    tasks.value.push(newTask);
    newTaskTitle.value = ''; // Clear input after adding
    console.log('Added new task:', newTask);
}

const onDrop = (event: DragEvent, targetListId: number) => {
    console.log('dropped on list', targetListId);
    event.preventDefault();
    if (event.dataTransfer) {
        const itemID = event.dataTransfer.getData('itemID');
        const item = tasks.value.find(task => task.id === parseInt(itemID));
        if (item) {
            item.listId = targetListId;
            console.log('Dropped', item);
        }
    }

};

const isDoneList = (listId: number) => {
    return listId === 3;
}

</script>

<template>
    <div>

        <div class="mb-4 ">
            <form action="" class="flex gap-2" @submit.prevent="() => addNewTask()">
                <input type="text" placeholder="New Task Title..." v-model="newTaskTitle" class="input" />
                <button class="btn btn-success">Add</button>
            </form>

        </div>
        <div class="flex gap-2">
            <div v-for="list in lists" :key="list.id"
                class="drop-zone min-h-100 flex-1 max-w-96 bg-base-200 p-4 rounded-lg" @drop="onDrop($event, list.id)"
                @dragenter.prevent @dragover.prevent>
                <h2 class="text-center">{{ list.name }}</h2>
                <div v-for="item in getTasksForList(list.id)" :key="item.id" class="drag-el p-2">
                    <taskItem :title="item.title" :description="item.description" draggable="true"
                        @dragstart="startDrag($event, item)" :done="isDoneList(list.id)" />
                </div>

            </div>

        </div>
    </div>
</template>