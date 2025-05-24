<script setup lang="ts">
defineProps({
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
    }
})
</script>

<template>
    <div class="card bg-base-100 w-80 card-xs shadow-sm" :class="{ 'opacity-30': done }" draggable="true">
        <button class="btn btn-xs btn-square btn-outline absolute top-2 right-2 z-10">x</button>
        <div class="card-body bg-primary rounded-lg" :class="{ 'bg-secondary': done }">
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