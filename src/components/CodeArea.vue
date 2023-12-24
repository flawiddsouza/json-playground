<template>
    <code-mirror
        ref="codeMirrorRef"
        :lang="lang"
        :placeholder="placeholder"
        @input="updateValue"
    ></code-mirror>
</template>

<script setup>
import 'code-mirror-custom-element'
import { onMounted, ref, watch } from 'vue'

const props = defineProps({
    modelValue: {
        type: String,
        required: true
    },
    lang: {
        type: String,
        default: 'javascript'
    },
    placeholder: {
        type: String,
        default: ''
    }
})

const codeMirrorRef = ref(null)
const emit = defineEmits(['update:modelValue'])

watch(
    () => props.modelValue,
    (newValue) => {
        if (codeMirrorRef.value && !codeMirrorRef.value.editor.hasFocus) {
            codeMirrorRef.value.setAttribute('value', newValue)
        }
    },
    { immediate: true }
)

onMounted(() => {
    if (codeMirrorRef.value && props.modelValue) {
        codeMirrorRef.value.setAttribute('value', props.modelValue)
    }
})

const updateValue = (event) => {
    emit('update:modelValue', event.target.value)
}
</script>
