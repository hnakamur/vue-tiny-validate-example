<script setup lang="ts">
import { reactive, ref, computed } from 'vue'
import useValidate from 'vue-tiny-validate'

const initialData = { lhs: '', rhs: '' }

const data = reactive({
  equations: [structuredClone(initialData)]
})

const testSumToBe9 = (value: any) => {
  const lhs = value.lhs
  const rhs = value.rhs
  const sum = Number(lhs) + Number(rhs)
  const result = sum === 9
  return result
}

const makeRuleSumMustBe9 = (i: number) => ({
  name: `sumMustBe9-${i}`,
  test: testSumToBe9,
  message: 'please input two numbers whose sum is 9'
})

const rules = reactive({
  equations: {
    0: makeRuleSumMustBe9(0)
  }
})
const options = reactive({ autoTouch: true })

const { result } = useValidate(data, rules, options)

const inputted = ref(false)
const handleInput = () => {
  result.value.$test()
  inputted.value = true
}

const message = computed(() =>
  inputted.value
    ? result.value.$invalid
      ? 'Some equation(s) are not correct'
      : 'All equations are correct!'
    : 'Input pairs of numbers whose sum is 9.'
)

const handleAddClick = (index: number) => {
  (rules.equations as any)[String(data.equations.length)] = makeRuleSumMustBe9(data.equations.length)
  data.equations.splice(index + 1, 0, structuredClone(initialData))
}

const handleDeleteClick = (index: number) => {
  data.equations.splice(index + 1, 1)
  delete (rules.equations as any)[String(data.equations.length)]
}
</script>

<template>
  <div>{{ message }}</div>
  <template v-for="(equation, index) in data.equations" v-bind:key="index">
    <div>
      <input v-model="equation.lhs" type="numeric" min="1" v-on:input="handleInput" /> +
      <input v-model="equation.rhs" type="numeric" min="1" v-on:input="handleInput" /> = 9
      <input type="button" value="Add" v-on:click="handleAddClick(index)" />
      <input type="button" value="Delete" v-on:click="handleDeleteClick(index)" />
    </div>
  </template>
</template>
