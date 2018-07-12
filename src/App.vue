<template>
  <div id="app">
    <Header 
      :hosts="displayedHosts" 
      :disableLoadMore="disableLoadMore"
      @dropdown-opened="onDropdownOpened"
      @delete-host="onDeleteHost"
      @load-more="onLoadMore"
      @term-change="onTermChange"
      @reverse-order="onReverseOrder" />
    <Content 
      @add-host="onAddHost"
      @toggle-error="onToggleError" />
  </div>
</template>

<script>
import axios from 'axios'

import Header from './components/header/Header.vue'
import Content from './components/content/Content.vue'

const ENTRY_LIMIT = 10
const FIREBASE_URL = "https://logdna-js-test.firebaseio.com"

let baseAjaxUrl = FIREBASE_URL

export default {
  name: 'app',
  components: {
    Header,
    Content
  },
  data() {
    return {
      allHosts: [], //stores all the hosts from the db
      currentHosts: [], //stores a current list of hosts, used for search
      displayedHosts: [], //currently displayed hosts
      isLoaded: false,
      hostCount: 0,
      disableLoadMore: true,
      isAscOrder: true,
      isErrorMode: false,
      searchTerm: ''
    }
  },
  methods: {
    onAddHost(newHost) {
      axios.post(`${baseAjaxUrl}/hosts.json`, newHost)
        .then((response) => {
          newHost = {
            "key": response.data.name,
            "id": newHost.id,
            "name": newHost.name,
            "html": newHost.name
          }
          this.currentHosts.push(newHost);
          this.onTermChange(this.searchTerm);
        })
        .catch(error => {
          alert('Error adding new host:', error);
        });
    },
    onDropdownOpened() {
      if (!this.isLoaded) {
        this.loadHosts();
        this.isLoaded = true;
      }
    },
    loadHosts() {
      // NOTE: ideally it should only get the first 10 entries from the server
      // and call additional AJAX requests when needed
      // The test was designed to get all hosts at the beginning so I left it this way
      axios.get(`${baseAjaxUrl}/hosts.json`)
        .then(response => {
          const data = response.data;
          const hostsArr = Object.keys(data)
            .map(key => {
              return {
                "key": key,
                "id": data[key].id,
                "name": data[key].name,
                "html": data[key].name
              }
            });
          this.allHosts = this.currentHosts = hostsArr;
          if (this.allHosts.length > ENTRY_LIMIT) {
            this.currentHosts = hostsArr.slice(0, ENTRY_LIMIT);
            this.hostCount = ENTRY_LIMIT;
            this.disableLoadMore = false;
          }
          this.setDisplayedHosts();
          this.sortHosts();
        })
        .catch(error => {
          alert('Error loading hosts', error);
        });
    },
    sortHosts() {
      this.displayedHosts.sort((a, b) => {
        const aName = a.name.toLowerCase();
        const bName = b.name.toLowerCase();
        return (aName < bName) ? (this.isAscOrder ? -1 : 1) : ((aName > bName) ? (this.isAscOrder ? 1 : -1) : 0)
      });
    },
    onDeleteHost(key) {
      axios.post(`${baseAjaxUrl}/hosts/${key}.json?x-http-method-override=DELETE`)
        .then(() => {
          this.currentHosts = this.currentHosts.filter((host) => {
            return host.key !== key;
          });
          this.setDisplayedHosts();
          this.sortHosts();
        })
        .catch(error => {
          alert('Error removing host:', error);
        });
    },
    onLoadMore() {
      const nextList = this.allHosts.slice(this.hostCount, this.hostCount + ENTRY_LIMIT);
      this.currentHosts = this.currentHosts.concat(nextList);
      this.hostCount += ENTRY_LIMIT;
      if (this.currentHosts.length == this.allHosts.length) {
        this.disableLoadMore = true;
      }
      this.onTermChange(this.searchTerm);
    },
    setDisplayedHosts() {
      this.displayedHosts = this.currentHosts.slice();
    },
    onTermChange(term) {
      this.searchTerm = term;
      if (term === "") {
        for (let currentHost of this.currentHosts) {
          currentHost.html = currentHost.name;
        }
        this.setDisplayedHosts();
      } else {
        let displayedHosts = this.currentHosts.filter(host => {
          return host.name.toLowerCase().indexOf(term) > -1;
        });
        displayedHosts = this.highlightTerm(displayedHosts);
        this.displayedHosts = displayedHosts;
        this.currentHosts = this.highlightTerm(this.currentHosts);
      }
      this.sortHosts();
    },
    onReverseOrder(checked) {
      this.isAscOrder = !checked;
      this.sortHosts();
    },
    onToggleError() {
      this.isErrorMode = !this.isErrorMode;
      this.isLoaded = false;
      this.allHosts = [];
      this.currentHosts = [];
      this.displayedHosts = [];
      this.hostCount = 0;
      this.disableLoadMore = true;
      this.isAscOrder = true;
      if (this.isErrorMode) {
        baseAjaxUrl = "/load-to-nowhere/";
      } else {
        baseAjaxUrl = FIREBASE_URL;
        this.loadHosts();
      }
    },
    highlightTerm(hosts) {
      if (hosts.length > 0) {
        const termLength = this.searchTerm.length;
        for (let host of hosts) {
          const itemText = host.name;
          const strIndex = itemText.toLowerCase().indexOf(this.searchTerm);
          host.html = itemText.substring(0, strIndex) 
            + '<strong style="color: #5bc0de;">' 
            + itemText.substring(strIndex, strIndex + termLength) 
            + "</strong>" 
            + itemText.substring(strIndex + termLength);
        }
      }
      return hosts;
    }
  }
}
</script>

<style>
@import './assets/css/linearicons.css';
@import './assets/css/normalize.css';
@import './assets/css/main.css';
</style>