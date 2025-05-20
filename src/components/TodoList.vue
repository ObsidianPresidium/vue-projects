<script setup lang="ts">
    import { ref, watch } from "vue";
    import type { Ref, CSSProperties } from "vue";
    import ItemCreator from "./ItemCreator.vue"
    type TodoItem = {
        id: number;
        name: string;
        completed: boolean;
    };

    let idIndex = 0;

    const listItemStyle = (isCompleted: boolean): CSSProperties => {
        if (isCompleted) return { textDecoration: "line-through", fontStyle: "italic" }
        return {}
    };

    const todoList: Ref<TodoItem[]> = ref([{
        id: 0,
        name: "Learn Vue",
        completed: false
    }]);

    const addTodoListItem = (name: string) => {
        if (name !== "just-close") todoList.value.push({id: idIndex++, name, completed: false});
    }

    const removeCompleted = () => {
        todoList.value = todoList.value.filter((item) => !item.completed);
    }

    let uncompletionQueue: TodoItem[] = [];

    const queueForUncompletion = (todo: TodoItem) => {
        uncompletionQueue.push(todo);
    };

    const leave = () => {
        uncompletionQueue.forEach(item => todoList.value[todoList.value.indexOf(item)].completed = false);
        uncompletionQueue = [];
    };

    const uncompleteFunction = (todo: TodoItem) => {
        if (todoList.value.filter((item) => item.completed).length === 1) {
            queueForUncompletion(todo);
            shouldFade.value = "fade-quick";
            completeExistState.value = false;
        } else {
            todo.completed = false;
        }
    }

    const track = (event: MouseEvent, targetItem: TodoItem) => {
        elDragTarget = (event.target as HTMLElement);
        initMouseX = mouseX.value;
        if (!dragTargetTranslating) mouseIsDown = true;
        elDeleter.value.style.display = "block";
        const dragTargetRect = elDragTarget.getBoundingClientRect();
        elDeleter.value.style.translate = `${dragTargetRect.x - 200}px ${dragTargetRect.y - 2}px`
        dragTargetItem = targetItem;
    };

    const stopTrack = () => {
        mouseIsDown = false;
        dragTargetTranslating = false;
        elDragTarget.style.translate = "";
        elDeleter.value.style.display = "none";
        if (activateDeletion) {
            todoList.value = todoList.value.filter((item: TodoItem) => item != dragTargetItem);
            activateDeletion = false;
        }
    };

    let shouldFade = ref("false");
    let creatingItem = ref(false);
    let collapsingCompleted = ref(false);
    let completedItemsExist = ref(false);
    let completeExistState = ref(false);

    let mouseX = ref(-1);
    let initMouseX = -1;
    let mouseIsDown = false;
    let elDragTarget: HTMLElement;
    let dragTargetItem: TodoItem;
    let dragTargetTranslating = false;
    let activateDeletion = false;

    let elDeleter = ref();

    watch(todoList, (_todoList) => {
        completedItemsExist.value = _todoList.filter((todoItem: TodoItem) => todoItem.completed).length != 0;
        completeExistState.value = completedItemsExist.value;
    }, {deep: true});

    document.addEventListener("mousemove", (e: MouseEvent) => {
        mouseX.value = e.screenX;
        if (mouseIsDown && elDragTarget) {
            if (mouseX.value - initMouseX < 0) elDragTarget.style.translate = `${Math.max(mouseX.value - initMouseX, -210)}px 0`;
            dragTargetTranslating = true;
        }
    });

    document.addEventListener("mouseup", stopTrack);

    watch(mouseX, (_mouseX) => {
        elDeleter.value.style.opacity = Math.max(Math.min(((mouseX.value - (initMouseX - 200)) / 175) * -1 + 1, 1), 0) / 2;
        activateDeletion = false;
        if (mouseX.value <= initMouseX - 200) {
            elDeleter.value.style.opacity = 1;
            activateDeletion = true;
        }
    });
    
</script>

<template>
    <div class="align-left" @dragover="e => e.preventDefault()">
        <Transition :name="shouldFade" @after-leave="leave">
            <div class="collapsed-container" v-if="completeExistState">
                <div class="completed-collapser" @click="collapsingCompleted = !collapsingCompleted" v-if="completedItemsExist">
                    <p>Completed <span :class="collapsingCompleted ? 'arrow-collapsed':'arrow-not-collapsed'">&bigtriangleup;</span></p>
                </div>
                <Transition name="collapse">
                    <div class="completed" v-if="!collapsingCompleted">
                        <ul>
                            <li v-for="todo in todoList.filter((item) => item.completed)" :key="todo.id" @mousedown="event => track(event, todo)" @click="uncompleteFunction(todo)" data-track-x-constraint="100" style="text-decoration: line-through; font-style: italic"><input type="checkbox" checked="true">&nbsp;{{ todo.name }}</li>
                        </ul>
                    </div>
                </Transition>
            </div>
        </Transition>
        <div class="others">
            <ul>
                <li v-for="todo in todoList.filter((item) => !item.completed)" :key="todo.id" @mousedown="event => track(event, todo)" @click="todo.completed = true" data-track-x-constraint="100"><input type="checkbox" v-model="todo.completed">&nbsp;{{ todo.name }}</li>
            </ul>
        </div>
        <div class="buttons">
            <button @click="creatingItem = true">New Item</button>&nbsp;
            <button @click="removeCompleted()">Remove Completed</button>
        </div>
        <Transition name="fade">
            <div class="modal-background" v-if="creatingItem" :style="(creatingItem) ? {}:{pointerEvents: 'none'}"></div>
        </Transition>
        <Transition name="fly">
            <ItemCreator v-if="creatingItem" @item-name="(name) => {addTodoListItem(name); creatingItem = false}" />
        </Transition>
        <div class="delete-item" ref="elDeleter">
            <svg width="800px" height="800px" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
                <path d="M10 12V17" stroke="#ffffff" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
                <path d="M14 12V17" stroke="#ffffff" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
                <path d="M4 7H20" stroke="#ffffff" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
                <path d="M6 10V18C6 19.6569 7.34315 21 9 21H15C16.6569 21 18 19.6569 18 18V10" stroke="#ffffff" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
                <path d="M9 5C9 3.89543 9.89543 3 11 3H13C14.1046 3 15 3.89543 15 5V7H9V5Z" stroke="#ffffff" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
            </svg>
        </div>
    </div>
</template>

<style scoped>
    :root {
        interpolate-size: allow-keywords;
    }

    .align-left {
        display: flex;
        align-items: start;
        justify-content: center;
        flex-direction: column;
    }

    .delete-item {
        display: none;
        background-image: linear-gradient(to right, #ff4857 80%, transparent 100%);
        border-radius: 25% 0 0 25%;
        padding: 0.25rem;
        position: fixed;
        left: 0;
        top: 0;
    }

    .delete-item svg {
        height: 1rem;
        width: 1rem;
    }

    .modal-background {
        position: fixed;
        left: 0;
        top: 0;
        width: 100%;
        height: 100%;
        background: #000a;
        z-index: 1;
    }

    .completed-collapser {
        cursor: pointer;
        color: #aaa;
        user-select: none;
    }

    .completed {
        overflow: hidden;
        display: block;
        margin-bottom: 0.5rem;
        white-space: nowrap;
    }

    ul {
        padding-left: 1rem;
    }

    li {
        cursor: default;
        user-select: none;
    }

    .others ul {
        padding-left: 0;
    }

    .arrow-collapsed,
    .arrow-not-collapsed {
        display: inline-block;
        transform-origin: 50% 60%;
        transition: transform 250ms;
    }

    .arrow-not-collapsed {
        transform: translateY(-10%) rotate(180deg);
    }

    .collapse-enter-active,
    .collapse-leave-active {
        transition: all 300ms ease;
    }

    .collapse-enter-from,
    .collapse-leave-to {
        max-height: 0;
        width: 0;
    }

    .collapse-enter-to,
    .collapse-leave-from {
        max-height: 100%;
        width: 100%;
    }

    .fade-enter-active,
    .fade-leave-active {
        transition: opacity 500ms ease;
    }

    .fade-enter-from,
    .fade-leave-to {
        opacity: 0;
    }

    .fade-quick-enter-active,
    .fade-quick-leave-active {
        transition: opacity 100ms ease;
    }

    .fade-quick-enter-from,
    .fade-quick-leave-to {
        opacity: 0;
    }

    .fly-enter-active {
        transition: transform 500ms ease 100ms;
    }

    .fly-leave-active {
        transition: transform 500ms ease;
    }

    .fly-enter-from,
    .fly-leave-to {
        transform: translateY(-100rem);
    }

    li {
        list-style: none;
    }
</style>