<template>
    <main style="display: grid; height: 100%; grid-template-rows: auto 1fr;">
        <nav>
            <h3>JSON Playground</h3>
            <div>
                <button @click="generateMapCode">Generate Map Code</button>
                <button @click="reset" class="ml-1rem">Reset</button>
            </div>
        </nav>
        <div class="container">
            <div id="json">
                <div>Input ({{ jsonLength }} items)</div>
                <CodeArea lang="json" v-model="json" placeholder="Enter JSON here..."></CodeArea>
            </div>
            <div id="code">
                <div>Code</div>
                <CodeArea lang="javascript" v-model="code" placeholder="Enter your code here..."></CodeArea>
            </div>
            <div id="output">
                <div>Output ({{ outputLength }} items)</div>
                <CodeArea lang="json" v-model="output" placeholder="See output here..."></CodeArea>
            </div>
        </div>
    </main>
</template>

<script setup>
import { ref, watch, onMounted } from 'vue'
import CodeArea from './CodeArea.vue'

const json = ref('')
const jsonLength = ref(0)
const code = ref('')
const outputLength = ref(0)
const output = ref('')

const generateMapCode = () => {
    if (!confirm('Are you sure you want to generate map code? Generating map code will overwrite your current code.')) {
        return
    }
    const jsonObject = JSON.parse(json.value || '[]')
    const jsonObjectKeys = Object.keys(jsonObject[0]).map(key => {
        return `${key}: item.${key}`
    })
    code.value = `json.map(item => {
    return {
        ${jsonObjectKeys.join(',\n        ')}
    }
})`
}

watch([json, code], () => {
    try {
        // Parse the JSON string into an object
        const jsonObject = JSON.parse(json.value || '{}')

        if (Array.isArray(jsonObject)) {
            jsonLength.value = jsonObject.length
        } else {
            jsonLength.value = Object.keys(jsonObject).length
        }

        const functionBody = 'return ' + code.value
        const userFunction = new Function('json', functionBody)

        // Execute the function and store the results
        const result = userFunction(jsonObject)
        if (typeof result === 'object') {
            // Convert the result to JSON string and format it
            const stringifiedResult = JSON.stringify(result, null, 4)
            if (Array.isArray(result)) {
                outputLength.value = result.length
                output.value =  stringifiedResult
            } else {
                outputLength.value = Object.keys(result).length
                output.value = stringifiedResult
            }
        } else {
            outputLength.value = 1
            output.value = String(result)
        }
    } catch (error) {
        output.value = error.message;
    }
}, { immediate: true })

watch(json, () => {
    localStorage.setItem('JSON-Playground:json', json.value)
})

watch(code, () => {
    localStorage.setItem('JSON-Playground:code', code.value)
})

watch(output, () => {
    localStorage.setItem('JSON-Playground:output', output.value)
})

onMounted(() => {
    json.value = localStorage.getItem('JSON-Playground:json') || ''
    code.value = localStorage.getItem('JSON-Playground:code') || ''
    output.value = localStorage.getItem('JSON-Playground:output') || ''
})
</script>

<style scoped>
nav {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding-bottom: 1rem;
}

.container {
    display: grid;
    grid-template-columns: 1fr 1fr;
    grid-template-rows: 1fr 1fr;
    grid-template-areas: "json code" "output output";
    gap: 1rem;
    height: 100%;
    overflow: auto;
}

#json {
    grid-area: json;
}

#code {
    grid-area: code;
}

#output {
    grid-area: output;
}

:where(#json, #code, #output) {
    display: flex;
    flex-direction: column;
    border: 1px solid #e0e0e0;
    overflow: auto;
}

:where(#json, #code, #output) > div {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 0.5rem;
    background-color: #f5f5f5;
    border-bottom: 1px solid #e0e0e0;
}
</style>
