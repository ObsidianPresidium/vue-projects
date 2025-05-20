<script setup lang="ts">
    import { ref, onMounted } from "vue";
    import type { Ref } from "vue";
    let itemName = ref();
    let itemNameInput = ref<HTMLInputElement | null>(null);

    const emit = defineEmits(["item-name"])

    let create = () => {
        emit("item-name", itemName);
    };

    onMounted(() => {
        (itemNameInput as Ref<HTMLInputElement>).value.focus();
    });
</script>

<template>
    <div class="container">
        <div class="background" @click="emit('item-name', 'just-close')"></div>
        <div class="modal">
                <h1>New item</h1>
                <div class="table">
                    <p>Item Name</p>
                    <input type="text" placeholder="Name" v-model="itemName" ref="itemNameInput">
                </div>
                <button @click="create">Create Item</button>
                <button class="close" @click="emit('item-name', 'just-close')">
                    <svg viewBox="-0.5 0 25 25" fill="none" xmlns="http://www.w3.org/2000/svg">
                        <path d="M3 21.32L21 3.32001" stroke="#000000" stroke-width="3" stroke-linecap="round" stroke-linejoin="round"/>
                        <path d="M3 3.32001L21 21.32" stroke="#000000" stroke-width="3" stroke-linecap="round" stroke-linejoin="round"/>
                    </svg>
                </button>
            </div>
    </div>
</template>

<style scoped>
    .container {
        position: fixed;
        width: 100%;
        height: 100%;
        left: 0;
        top: 0;
        z-index: 2;
    }

    .background {
        position: fixed;
        left: 0;
        top: 0;
        width: 100%;
        height: 100%;
        z-index: 3;
    }

    .modal {
        position: absolute;
        padding: 2rem;
        border: 0.3rem solid black;
        border-radius: 1rem;
        background-color: white;
        z-index: 5;
        left: 50%;
        top: 50%;
        translate: -50% -50%;
    }

    .table {
        margin: 2rem 0;
        display: grid;
        grid-template-columns: repeat(2, 1fr);
    }

    .close {
        position: absolute;
        right: 0;
        top: 0;
        border-radius: 50%;
        margin: 1rem;
        padding: 1rem;
        border: 0.15rem solid black;
        width: 1rem;
        height: 1rem;
        display: flex;
        align-items: center;
        justify-content: center;
    }
    svg {
        position: absolute;
        width: 1rem;
        height: 1rem;
        pointer-events: none;
        z-index: 6;
    }
</style>