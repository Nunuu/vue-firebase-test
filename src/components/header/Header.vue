<template>
  <header>
    <div class="filter">
      <span :class="filterClass()" @click="onFilterClick">
        Hosts Dropdown
        <span class="lnr lnr-chevron-down"></span>
      </span>
    </div>
    <div class="dropdown" v-if="shown">
      <input 
        class="search"
        type="text" 
        placeholder="Search Here.."
        @input="onSearch" />
      <div class="sort-options">
        <label>
          <input type="checkbox" @click="reverseSortOrder" />
          Reverse Sort Order
        </label>
      </div>
      <hr/>
      <div class="no-results" v-if="noresults">
        No Results Found.
      </div>
      <div v-else>
        <div class="selectall">
          <label>
            <input 
              type="checkbox" 
              @click="onSelectAll" 
              :checked="isSelectAll" />
            Select All
          </label>
        </div>
        <ul id="hostlist">
          <li 
            v-for="host in hosts"
            :key="host.key" 
            :data-id="host.id" 
            :data-key="host.key">
            <input type="checkbox" @click="onHostCheck" />
            <span class="label" v-html="host.html"></span>
            <button class="delete" @click="onHostDelete">
              <span class="lnr lnr-delete"></span>
            </button>
          </li>
        </ul>
        <button 
          :class="loadMoreClass()" 
          @click="onLoadMore">Show More</button>
      </div>
    </div>
  </header>
</template>

<script>
export default {
  name: 'Header',
  props: ['hosts', 'disableLoadMore'],
  data() { 
    return { 
      shown: false,
      isSelectAll: false,
      checkedHosts: [],
      noresults: true
    }
  },
  watch: {
    hosts(newHosts) {
      if (newHosts.length > 0) {
        this.noresults = false;
      } else {
        this.noresults = true;
      }
    }
  },
  methods: {
    loadMoreClass() {
      return this.disableLoadMore ? 'disabled loadmore' : 'loadmore';
    },
    filterClass() {
      return this.shown ? 'up filter-title' : 'filter-title';
    },
    onFilterClick() {
      this.shown = !this.shown;
      if (!this.shown) {
        console.log('list of checked hosts');
        console.log(this.checkedHosts);
      } else {
        this.$emit('dropdown-opened');
      }
    },
    onHostCheck(event) {
      const liInput = event.currentTarget;
      const li = liInput.parentNode;
      const isChecked = liInput.checked;
      const id = li.getAttribute('data-id');
      if (isChecked) {
        this.checkedHosts.push({
          "id": id,
          "name": li.getElementsByTagName("span")[0].innerText
        });
        if (this.checkedHosts.length == this.hosts.length) {
          this.isSelectAll = true;
        }
      } else {
        this.checkedHosts = this.checkedHosts.filter((host) => {
          return host.id !== id;
        });
        this.isSelectAll = false;
      }
    },
    onHostDelete(event) {
      const key = event.currentTarget.parentNode.getAttribute('data-key');
      this.$emit('delete-host', key);
    },
    onLoadMore() {
      this.isSelectAll = false;
      this.$emit('load-more');
    },
    onSearch(event) {
      const term = event.target.value.toLowerCase();
      this.$emit('term-change', term);

      if (this.hosts.length > 0) {
        const hostItems = document.getElementById('hostlist').getElementsByClassName("label");
        const termLength = term.length;
        for (let hostItem of hostItems) {
          const itemText = hostItem.innerText;
          const strIndex = itemText.toLowerCase().indexOf(term);
          if (strIndex > -1) {
            const matchEnd = strIndex + termLength;
            hostItem.innerHTML = itemText.substring(0, strIndex) + '<strong style="color: #5bc0de;">' + itemText.substring(strIndex, strIndex + termLength) + "</strong>" + itemText.substring(matchEnd);
          } 
        }
      }
    },
    reverseSortOrder(event) {
      this.$emit('reverse-order', event.target.checked);
    },
    onSelectAll(event) {
      this.isSelectAll = event.target.checked;
      const checks = document.getElementById('hostlist').getElementsByTagName("input");
      for (let check of checks) {
        check.checked = this.isSelectAll;
      }
    }
  }
}
</script>

<style lang="sass" scoped>
$leftPadding: 40px

header
  width: 100%
  height: 56px
  background-color: #333
  box-shadow: 0 1px 2px rgba(0,0,0,0.4)
  padding: 0 $leftPadding
  z-index: 111
  position: relative

.filter
  float: left
  margin-top: 10px
  margin-left: 20px
  user-select: none
  position: relative
  .filter-title
    font-size: 18px
    font-weight: 100
    line-height: 1.5em
    text-shadow: 0 1px 0 rgba(0,0,0,0.6)
    cursor: pointer
    color: #f1f1f2
    transition: color 0.3s ease-out
    span
      display: inline-block
      transition: transform 0.3s ease-out
    &:hover
      color: #5bc0de
    &.up
      span
        transform: rotate(-180deg)
  span
    position: relative
    top: 3px
    margin-left: 1em

.dropdown
  position: absolute
  top: 100%
  left: $leftPadding
  background: white
  padding: 2em
  box-shadow: 1px 1px 5px rgba(black, 0.2)
  text-align: center
  min-width: 25em
  max-height: calc(90vh - 56px)
  overflow-y: scroll
  .no-results
    font-size: 1.3em
    padding: 1em 2em
  input.search
    font-size: 1.2em
    margin-bottom: 1em
    width: calc(100% - 2em)
    padding: 0.5em
  .sort-options
    margin-bottom: 1em
    input
      margin-right: 0.5em
    label
      font-size: 1.2em
      display: flex
      align-items: center
      justify-content: center
  hr
    box-shadow: none
    background: none
    border: none
    border-bottom: 1px solid #ccc
    margin: 2em auto 1em
    width: calc(100% - 2em)
  .selectall
    text-align: left
    padding-left: 1.3em
    margin: 1.5em 0 1em
    label
      font-size: 1.3em
      cursor: pointer
    input
      margin-right: 0.5/1.3+0em
  #hostlist
    padding: 0
    margin: 0
    text-align: left
    li
      list-style-type: none
      font-size: 1.3em
      padding: 1em
      transition: background 0.3s ease-out
      input
        margin-right: 0.5em
        position: relative
        top: 0.1em
      &:last-child
        margin-bottom: 0
      button.delete
        margin-left: 2em
        float: right
        transition: color 0.3s ease-out
        padding: 0
        line-height: 1
        span
          font-size: 1.2em
        &:hover
          color: red
      &:hover
        background: #f1f1f2

  button.loadmore
    background: #f1f1f2
    font-size: 1.4em
    margin: 1em auto
    &:hover
      background: #ccc
    &.disabled
      display: none

</style>