<template>
  <button type="button" :class=btnStyle >
    {{btnLabel}}
  </button>
</template>

<script>
export default {
  name: 'Button',
  props: ['btnLabel', 'btnStyle']
}
</script>

<style lang="sass" scoped>
$blue: #5bc0de
$orange: #f0ad4e
$darkenPerc: 10%

button
  color: white
  font-size: 1.8em
  padding: 1em
  padding: 10px 16px
  border-radius: 6px
  cursor: pointer
  transition: background 0.3s ease-out
  line-height: 1.5em
  margin-right: 5px
  &:last-child
    margin-right: 0

.blue
  background-color: $blue
  border: 1px solid darken($blue, $darkenPerc)
  &:hover
    background-color: darken($blue, $darkenPerc)

.orange
  background-color: $orange
  border: 1px solid darken($orange, $darkenPerc)
  &:hover
    background-color: darken($orange, $darkenPerc)

</style>