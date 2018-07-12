<template>
  <div class="filter">
    <span class="filter-title" @click="onFilterClick">
      Hosts Dropdown
      <span class="caret"></span>
    </span>
    <ul v-if="shown">
      <li v-for="host in hosts" :data-id="host.id" @click="onHostClick">
        <input type="checkbox" />
        <span class="label">{{ host.name }}</span>
      </li>
    </ul>
  </div>
</template>

<script>
import $ from "jquery";

export default {
  name: 'Dropdown',
  props: ['btnLabel', 'hosts'],
  data() { 
    return { 
      shown: false,
      checkedHosts: []
    }
  },
  methods: {
    onFilterClick(event) {
      this.shown = !this.shown;
      if (this.shown) {
        $.ajax({
          url: "/get-hosts"
        }).done(function(response) {
          // response.sort(compare);
          this.hosts = response;
        }).fail(function(response) {
          alert('Error loading hosts', response);
        });
      } else {
        console.log('list of checked hosts');
        console.log(this.checkedHosts);
      }
    },
    onHostClick(event) {
      const $li = $(event.currentTarget);
      $li.toggleClass('selected');
      const $liInput = $li.find('input');
      const isChecked = !$liInput.prop('checked');
      $liInput.prop('checked', isChecked);

      if (isChecked) {
        console.log('add li');
      } else {
        console.log('remove li');
      }
    }
  }
}
</script>

<style lang="sass" scoped>
.filter
  float: left
  margin-top: 10px
  margin-left: 20px
  user-select: none
  position: relative

.filter-title
  font-size: 18px
  font-weight: 100
  line-height: 34px
  text-shadow: 0 1px 0 rgba(0,0,0,0.6)
  cursor: pointer
  color: #f1f1f2

.caret
  display: inline-block
  width: 0
  height: 0
  margin-left: 2px
  vertical-align: middle
  border-top: 4px dashed
  border-top: 4px solid
  border-right: 4px solid transparent
  border-left: 4px solid transparent

ul
  position: absolute
  top: 2em
  left: 0
  background: white
  padding: 1em
  border: 1px solid black
  li
    list-style-type: none

</style>