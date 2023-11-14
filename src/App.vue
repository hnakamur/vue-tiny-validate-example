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
  console.log('testSumToBe9 lhs=', lhs, ', rhs=', rhs, ', sum=', sum, ', result=', result)
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
const options = reactive({})

const { result } = useValidate(data, rules, options)

const inputted = ref(false)
const handleInput = (index: number) => {
  result.value.equations[String(index)].$touch()
  result.value.$test()
  inputted.value = true
}

const debugDirty = () => {
  console.log('debugDirty, result.$dirty=', result.value.$dirty, ', result.equations.$dirty=', result.value.equations.$dirty)
  for (let i = 0; result.value.equations[String(i)]; i++) {
    console.log('debugDirty, i=', i, ', result.equations[i].$dirty=', result.value.equations[String(i)].$dirty)
  }
}
const message = computed(() => {
  console.log(
    'message inputted=',
    inputted.value,
    ', result.$invalid=',
    result.value.$invalid,
    ', result=',
    result.value
  )
  return inputted.value
    ? result.value.$invalid
      ? 'Some equation(s) are not correct'
      : 'All equations are correct!'
    : 'Input pairs of numbers whose sum is 9.'
})

const touchEquations = () => {
  for (let i = 0; result.value.equations[String(i)]; i++) {
    console.log('touchEquations i=', i, ', result.equations[i]=', result.value.equations[String(i)])
    result.value.equations[String(i)].$touch()
    console.log('touchEquations i=', i, ', result.equations[i].$dirty=', result.value.equations[String(i)].$dirty)
  }
}

const handleAddClick = (index: number) => {
  (rules.equations as any)[String(data.equations.length)] = makeRuleSumMustBe9(
    data.equations.length
  )
  data.equations.splice(index + 1, 0, structuredClone(initialData))
  console.log('handleAddClick calling result.$test()')
  result.value.$test()
  touchEquations()
  debugDirty()
}

const handleDeleteClick = (index: number) => {
  data.equations.splice(index + 1, 1)
  delete (rules.equations as any)[String(data.equations.length)]
  console.log('handleDeleteClick calling result.$test()')
  result.value.$test()
  touchEquations()
  debugDirty()
}
</script>

<template>
  <div>{{ message }}</div>
  <template v-for="(equation, index) in data.equations" v-bind:key="index">
    <div>
      <input v-model="equation.lhs" type="numeric" min="1" v-on:input="handleInput(index)" /> +
      <input v-model="equation.rhs" type="numeric" min="1" v-on:input="handleInput(index)" /> = 9
      <input type="button" value="Add" v-on:click="handleAddClick(index)" />
      <input type="button" value="Delete" v-on:click="handleDeleteClick(index)" />
    </div>
  </template>
</template>
