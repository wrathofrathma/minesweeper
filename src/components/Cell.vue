<template>
    <button 
    @click="$emit('click', {row, col})" 
    @contextmenu="emitFlag"
    class="p-2 m-0.5 rounded-sm focus:outline-none h-16 w-16 font-bold"
    :class="classes"
    >
    <span v-if="state===2 && ![0,-1].includes(value)">{{value}}</span>
    </button>
</template>

<script setup lang="ts">
import {defineProps, defineEmit, useContext, computed} from "vue";

const props = defineProps({
    row: {
        type: Number,
        required: true
    },
    col: {
        type: Number,
        required: true
    },
    state: {
        type: Number,
        required: true
    },
    value: {
        type: Number, 
        required: true
    }
})

defineEmit(["flag", "click"]);
const {emit} = useContext();

const emitFlag = (e: any) => {
    emit("flag", {row: props.row, col: props.col})
    e.preventDefault();
}

const classes = computed(() => {
    return {
        // Background based on the state
        // Flagged
        "bg-red-400": props.state === 1,
        "hover:bg-red-500": props.state === 1,
        // Unflagged
        "bg-gray-500": props.state === 0,
        "hover:bg-gray-700": props.state === 0,
        // Exposed, don't need a hover state for this.
        "bg-gray-700": props.state === 2 && props.value !== -1,
        "bg-red-700": props.state === 2 && props.value === -1,
        // Text based on the value
        "text-blue-400": props.value === 1,
        "text-green-400": props.value === 2,
        "text-red-400": props.value === 3,
        "text-purple-400": props.value > 3
    }
})
</script>