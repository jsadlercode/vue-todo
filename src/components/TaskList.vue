<script setup lang="ts">
import taskItem from './taskitem.vue';
import { ref, nextTick } from 'vue';

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
// --- Modal Logic ---

const taskDialogRef = ref<HTMLDialogElement | null>(null);
const isModalVisible = ref(false);
const taskToEdit = ref<null | { id: number, title: string, description: string }>(null);

const openEditModal = async (taskId: number) => {
    console.log('Opening edit modal for task ID:', taskId);
    const foundTask = tasks.value.find(task => task.id === taskId);
    if (foundTask) {
        taskToEdit.value = { ...foundTask };
        isModalVisible.value = true;
        console.log('Modal visible: ', isModalVisible.value);

        await nextTick();
        if (taskDialogRef.value) {
            console.log('Dialog element found, calling showModal');
            taskDialogRef.value.showModal();
        } else {
            console.error('Dialog element not found');
        }
    }
};

const closeEditModal = () => {
    console.log('Closing edit modal');
    if (taskDialogRef.value && taskDialogRef.value.hasAttribute('open')) {
        taskDialogRef.value.close();
    }
    isModalVisible.value = false;
    taskToEdit.value = null;
    console.log('Modal visible state set to: ', isModalVisible.value);
};

const saveTask = () => {
    if (taskToEdit.value) {
        const taskIndex = tasks.value.findIndex(task => task.id === taskToEdit.value!.id);
        if (taskIndex !== -1) {
            tasks.value[taskIndex] = { ...taskToEdit.value };
        }
        closeEditModal();
    }
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
                    <taskItem :title="item.title" :description="item.description" :id="item.id" draggable="true"
                        @dragstart="startDrag($event, item)" :done="isDoneList(list.id)" @editTask="openEditModal" />
                </div>
            </div>
        </div>
        <!-- Modal -->

        <dialog id="task_model" class="modal" ref="taskDialogRef" v-if="isModalVisible && taskToEdit">
            <div class="modal-box">
                <div class="mb-4">
                    <label for="editTitle" class="block text-sm font-medium text-gray-700">Title</label>
                    <input type="text" id="editTitle" v-model="taskToEdit.title"
                        class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm py-2 px-3 focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm">
                </div>
                <div class="mb-6">
                    <label for="editDescription" class="block text-sm font-medium text-gray-700">Description</label>
                    <textarea id="editDescription" v-model="taskToEdit.description" rows="3"
                        class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm py-2 px-3 focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm"></textarea>
                </div>
                <div class="modal-action">
                    <button @click="closeEditModal" class="btn btn-ghost">Cancel</button>
                    <button @click="saveTask" class="btn btn-primary">Save Changes</button>
                </div>
                <p class="py-4">Press ESC key or click outside to close</p>
            </div>
            <form method="dialog" class="modal-backdrop">
                <button>Close</button>
            </form>
        </dialog>

    </div>
</template>