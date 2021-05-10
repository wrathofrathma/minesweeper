<template>
    <button 
    @click="$emit('click', {row, col})" 
    @contextmenu="emitFlag"
    class="lg:p-2 p-1 m-0.5 rounded-sm focus:outline-none font-bold text-xs lg:text-md xl:text-lg"
    :class="classes"
    :style="style"
    >
    <span v-if="state===2 && ![0,-1].includes(value)">{{value}}</span>
    </button>
</template>

<script setup lang="ts">
import {defineProps, defineEmit, useContext, computed} from "vue";
import lodash from "lodash";

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
    },
    width: {
        type: Number,
        required: true,
    },
    height: {
        type: Number,
        required: true,
    },
    viewport: {
        type: Object,
        required: true
    }
})

defineEmit(["flag", "click"]);
const {emit} = useContext();

const emitFlag = (e: any) => {
    emit("flag", {row: props.row, col: props.col})
    e.preventDefault();
}

const style = computed(() => {
    if (props.viewport.height > props.viewport.width) {
        // Restrict by width
        return {
            width: `${lodash.floor(90 / props.width)}vw`,
            height: `${lodash.floor(90 / props.width)}vw`,
        }
    } else {
        // Restrict by height
        return {
            width: `${lodash.floor(90 / props.height)}vh`,
            height: `${lodash.floor(90 / props.height)}vh`,
        }
    }
    return {};
})

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
        "text-purple-400": props.value > 3,
    }
})
</script>