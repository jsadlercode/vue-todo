<script setup lang="ts">
import taskItem from './taskitem.vue';
import { ref, nextTick } from 'vue';

interface Task {
    id: number;
    title: string;
    description: string;
    listId: number;
}
interface List {
    id: number;
    name: string;
}

const lists = ref<List[]>([
    { id: 1, name: 'To Do' },
    { id: 2, name: 'Doing' },
    { id: 3, name: 'Done' },
]);

const tasks = ref<Task[]>([
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

const lastId = ref(tasks.value.length > 0 ? Math.max(...tasks.value.map(t => t.id)) : 0);

const addNewTask = () => {
    if (newTaskTitle.value.trim() === '') {
        return;
    }
    lastId.value++;
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

const onDropOnList = (event: DragEvent, targetListId: number) => {
    console.log('dropped on list container', targetListId);
    event.preventDefault();
    if (event.dataTransfer) {
        const itemIDStr = event.dataTransfer.getData('itemID');
        if (!itemIDStr) return;

        const draggedItemId = parseInt(itemIDStr);
        const tasksArray = tasks.value;
        const draggedItemIndex = tasksArray.findIndex(task => task.id === draggedItemId);

        if (draggedItemIndex !== -1) {
            // Check if the drop target is actually a task item child; if so, let taskItem handle it.
            // This can happen if dragleave from item fires late or drop on padding.
            if ((event.target as HTMLElement).closest('.drag-el > div')) { // .drag-el is parent, div is card
                console.log("Dropped on list, but likely over an item. Item should handle.");
                return;
            }

            const [draggedItem] = tasksArray.splice(draggedItemIndex, 1); // Remove from old position
            draggedItem.listId = targetListId; // Update listId

            // Find the index of the last task belonging to targetListId to append after it
            let lastIndexOfTargetList = -1;
            for (let i = tasksArray.length - 1; i >= 0; i--) {
                if (tasksArray[i].listId === targetListId) {
                    lastIndexOfTargetList = i;
                    break;
                }
            }

            if (lastIndexOfTargetList !== -1) {
                tasksArray.splice(lastIndexOfTargetList + 1, 0, draggedItem);
            } else {
                // If the target list is empty, just add it.
                // It will be the first (and only) item for that listId.
                tasksArray.push(draggedItem);
            }
            console.log('Item moved to end of list:', draggedItem);
        }
    }
};

const handleDropOnTaskItem = (payload: { draggedItemId: number; targetItemId: number; position: 'top' | 'bottom' }) => {
    const { draggedItemId, targetItemId, position } = payload;
    console.log(`Drop on item: dragged ${draggedItemId} onto ${targetItemId} at ${position}`);

    const tasksArray = tasks.value;
    const draggedItemIndex = tasksArray.findIndex(t => t.id === draggedItemId);
    const targetItemIndex = tasksArray.findIndex(t => t.id === targetItemId);

    if (draggedItemIndex === -1 || targetItemIndex === -1) {
        console.error('Dragged or target item not found in tasks array');
        return;
    }

    // 1. Remove the dragged item from its current position
    const [draggedItem] = tasksArray.splice(draggedItemIndex, 1);

    // 2. Update its listId to that of the target item's list
    // Need to find the target item again as its index might have changed after splice
    const currentTargetItem = tasks.value.find(t => t.id === targetItemId);
    if (!currentTargetItem) { // Should not happen
        tasksArray.splice(draggedItemIndex, 0, draggedItem); // Put it back
        return;
    }
    draggedItem.listId = currentTargetItem.listId;


    // 3. Find the new index of the target item (it might have shifted if dragged item was before it)
    const newTargetItemIndex = tasksArray.findIndex(t => t.id === targetItemId);

    // 4. Insert the dragged item at the correct new position
    if (position === 'top') {
        tasksArray.splice(newTargetItemIndex, 0, draggedItem);
    } else { // 'bottom'
        tasksArray.splice(newTargetItemIndex + 1, 0, draggedItem);
    }
    console.log('Tasks reordered:', tasks.value);
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

// End of Modal Logic
const deleteTask = (taskIdToDelete: number) => {
    console.log('Attempting to delete task with ID:', taskIdToDelete);
    const indexToDelete = tasks.value.findIndex(task => task.id === taskIdToDelete);

    if (indexToDelete !== -1) {
        tasks.value.splice(indexToDelete, 1);
        console.log('Task deleted successfully:', taskIdToDelete);
    } else {
        console.warn('Task not found for deletion:', taskIdToDelete);
    }
};


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
                class="drop-zone min-h-100 flex-1 max-w-96 bg-base-200 p-4 rounded-lg"
                @drop="onDropOnList($event, list.id)" @dragenter.prevent @dragover.prevent>
                <h2 class="text-center">{{ list.name }}</h2>
                <div class="space-y-3">
                    <div v-for="item, index in getTasksForList(list.id)" :key="item.id" :item="item"
                        class="drag-el p-2">
                        <taskItem 
                        :title="item.title" 
                        :description="item.description" 
                        :index="index" 
                        :id="item.id"
                        draggable="true" 
                        @dragstart="startDrag($event, item)" 
                        :done="isDoneList(list.id)"
                        @editTask="openEditModal" 
                        @deleteTask="deleteTask"
                        @dropOnItem="handleDropOnTaskItem" />
                    </div>
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