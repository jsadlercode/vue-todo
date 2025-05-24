<script setup lang="ts">
import taskItem from './taskitem.vue';
import { ref } from 'vue';

const tasks = ref([
    {
        id: 1,
        title: 'Task 1',
        description: 'Description for Task 1',
        list: 1,
    },
    {
        id: 2,
        title: 'Task 2',
        description: 'Description for Task 2',
        list: 2,
    },
    {
        id: 3,
        title: 'Task 3',
        description: 'Description for Task 3',
        list: 1,
    },
    {
        id: 4,
        title: 'Task 4',
        description: 'Description for Task 4',
        list: 3,
    },
    {
        id: 5,
        title: 'Task 5',
        description: 'Description for Task 5',
        list: 2,
    },
    {
        id: 6,
        title: 'Task 6',
        description: 'Description for Task 6',
        list: 1,
    },
]);

const getList = (list) => {
    return tasks.value.filter(task => task.list === list);
};

const startDrag = (event, item) => {
    console.log('dragging', item);
    event.dataTransfer.dropEffect = 'move';
    event.dataTransfer.effectAllowed = 'move';
    event.dataTransfer.setData('itemID', item.id);
};

const onDrop = (event, list) => {
    console.log('dropped on list', list);
    event.preventDefault();
    const itemID = event.dataTransfer.getData('itemID');
    const item = tasks.value.find(task => task.id === parseInt(itemID));
    if (item) {
        item.list = list;
        console.log('Dropped', item);
    }
};

</script>

<template>
    <div>
        <h2>Tasks</h2>
        <div class="flex gap-2">
            <div class="drop-zone min-h-100 flex-1 max-w-96 bg-base-200 p-4 rounded-lg" 
            @drop="onDrop($event, 1)"
            @dragenter.prevent
            @dragover.prevent>
            <h2 class="text-center">To Do</h2>
                <div v-for="item in getList(1)" :key="item.id" class="drag-e p-2">
                    <taskItem 
                    :title="item.title" 
                    :description="item.description" 
                    draggable="true" 
                    @dragstart="startDrag($event, item)" />
                </div>

            </div>
            <div class="drop-zone min-h-100 flex-1 max-w-96 bg-base-200 p-4 rounded-lg" 
            @drop="onDrop($event, 2)"
            @dragenter.prevent
            @dragover.prevent>
            <h2 class="text-center">Doing</h2>
                <div v-for="item in getList(2)" :key="item.id" class="drag-el p-2">
                    <taskItem 
                    :title="item.title" 
                    :description="item.description" 
                    draggable="true" 
                    @dragstart="startDrag($event, item)"/>
                </div>
            </div>
            <div class="drop-zone min-h-100 flex-1 max-w-96 bg-base-200 p-4 rounded-lg" 
            @drop="onDrop($event, 3)"
            @dragenter.prevent
            @dragover.prevent>
            <h2 class="text-center">Done</h2>
                <div v-for="item in getList(3)" :key="item.id" class="drag-e p-2">
                    <taskItem 
                    :title="item.title" 
                    :description="item.description" 
                    draggable="true" 
                    @dragstart="startDrag($event, item)"
                    :done="true" />
                </div>

            </div>
        </div>
    </div>
</template>