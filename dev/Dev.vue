<template>
  <div id="app">
    <v-select ref="test" :value="selected"        @open="handleDropdownOpened"


              @open-ref="handleRefClick"
              @keyup-enter="handleEnterKeyPress"
              @keyup-backspace="handleBackspacePress"
              @option-selected="handleChange"
              @option-deselected="handleChange"
              @open-create-entity="handleCreateEntityClick"    @search="handleSearch" @input="selected = $event" v-bind="config" />
  </div>
</template>

<script>
import vSelect from '../src/components/Select.vue'
import countries from '../docs/.vuepress/data/countryCodes'

export default {
  components: { vSelect },
  data: () => ({
    selected: null,
    config: {
      options: countries,
    },
    searchQueue: [],
  }),
  methods: {
    handleSearch(value) {
      this.searchQueue.push(value);
      this.$nextTick(this.processSearchQueue);
    },
    processSearchQueue() {
      console.log("test")
    }
  }
}
</script>

<style>
html,
body {
  margin: 0;
  height: 100%;
  font-family: -apple-system, sans-serif;
}

#app {
  height: 100%;
  max-width: 20rem;
  margin: 10rem auto 0;
}

hr {
  border: none;
  border-bottom: 1px solid #cacaca;
  margin-bottom: 1em;
  padding-top: 1em;
  width: 90%;
}
</style>
