<template>
  <div class="content">
    <div class="buttons-wrapper">
      <button type="button" class="blue" @click="onAddHost">
        Add a Random Host
      </button>
      <button type="button" class="orange" @click="onToggleError">
        Toggle Error Mode
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Content',
  methods: {
    onAddHost() {
      const seed = (Math.random().toString(16)+'000000000').substr(2,8);
      const randomHost = {
        id: 'random-host-' + seed, 
        name: 'A Random Host (' + seed + ')'
      };
      this.$emit('add-host', randomHost);
    },
    onToggleError() {
      this.$emit('toggle-error');
    }
  }
}
</script>

<style lang="sass" scoped>
$blue: #5bc0de
$orange: #f0ad4e
$darkenPerc: 10%

.content
  padding: 4em 10%

.buttons-wrapper
  display: flex
  justify-content: center

button
  color: white
  font-size: 1.8em
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