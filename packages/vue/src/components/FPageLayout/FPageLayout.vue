<script setup lang="ts">
import { computed, onMounted, useSlots } from "vue";
import { PageLayout } from "./webcomponent";

const emit = defineEmits<{
    /** Emitted when the layout has been recalculated */
    update: [];
}>();

const { layout } = defineProps<{ layout: string }>();
const slots = useSlots();
const tagName = `ce-page-layout`;

const slotNames = computed((): string[] => {
    return Object.keys(slots);
});

onMounted(() => {
    if (!customElements.get(tagName)) {
        customElements.define(tagName, PageLayout);
    }
});

function onUpdate(): void {
    emit("update");
}
</script>

<template>
    <component :is="tagName" :layout @update="onUpdate">
        <!-- eslint-disable-next-line vue/no-deprecated-slot-attribute -- false positive, this is the native slot attribute -->
        <div v-for="slot of slotNames" :key="slot" :slot>
            <slot :name="slot"></slot>
        </div>
    </component>
</template>
