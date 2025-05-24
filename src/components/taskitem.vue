<script setup lang="ts">
const props = defineProps({
    id: {
        type: Number,
        required: true
    },
    title: {
        type: String,
        default: 'Card Title'
    },
    description: {
        type: String,
        default: 'A card component has a figure, a body part, and inside body there are title and actions parts'
    },
    done: {
        type: Boolean,
        default: false
    },
    index: {
        type: Number,
        required: true
    }
})

const emit = defineEmits(['editTask', 'deleteTask']);

const onEditClick = () => {
    emit('editTask', props.id);
    console.log('Emitting editTask with id:', props.id);
};

const onDeleteClick = () => {
    emit('deleteTask', props.id);
    console.log('Emitting deleteTask with id:', props.id);
};

</script>

<template>
    <div class="card bg-base-100 w-80 card-xs shadow-sm" :class="{ 'opacity-30': done }" draggable="true">
        <button class="btn btn-xs btn-square btn-primary absolute top-2 right-9 z-10" @click="onEditClick">
            <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5"
                stroke="currentColor" class="size-6">
                <path stroke-linecap="round" stroke-linejoin="round"
                    d="m16.862 4.487 1.687-1.688a1.875 1.875 0 1 1 2.652 2.652L10.582 16.07a4.5 4.5 0 0 1-1.897 1.13L6 18l.8-2.685a4.5 4.5 0 0 1 1.13-1.897l8.932-8.931Zm0 0L19.5 7.125M18 14v4.75A2.25 2.25 0 0 1 15.75 21H5.25A2.25 2.25 0 0 1 3 18.75V8.25A2.25 2.25 0 0 1 5.25 6H10" />
            </svg>
        </button>

        <button class="btn btn-xs btn-square btn-primary absolute top-2 right-2 z-10" @click="onDeleteClick">X</button>
        <div class="card-body bg-primary rounded-lg">
            <h2 class="card-title" :class="{ 'line-through': done }">
                {{ title }}
                <div class="badge badge-xs badge-secondary">!</div>
            </h2>

            <p>{{ description }}</p>
            <div class="card-actions justify-end">
                <div class="badge badge-xs badge-outline">Fashion</div>
                <div class="badge badge-xs badge-outline">Products</div>
            </div>
        </div>
    </div>
</template>