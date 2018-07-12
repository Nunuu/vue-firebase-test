<template>
  <div id="app">
    <Header :hosts="hosts" v-on:dropdown-opened="onDropdownOpened" />
    <Content v-on:add-host="onAddHost" />
  </div>
</template>

<script>
import $ from "jquery"

import Header from './components/header/Header.vue'
import Content from './components/content/Content.vue'

export default {
  name: 'app',
  components: {
    Header,
    Content
  },
  data() {
    return { 
      hosts: []
    };
  },
  methods: {
    onAddHost(newHost) {
      this.hosts.push(newHost);
      console.log('new host added:', this.hosts);
    },
    onDropdownOpened() {
      console.log('call ajax!');
      $.ajax({
        url: "/get-hosts"
      }).done((response) => {
        console.log('response loaded:', response);
        this.hosts = response;
      }).fail((response) => {
        alert('Error loading hosts', response);
      });
    }
  }
}
</script>

<style>
@import './assets/css/normalize.css';
@import './assets/css/main.css';
</style>