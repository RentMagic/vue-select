# vue-select ![Current Release](https://img.shields.io/github/release/sagalbot/vue-select.svg?style=flat-square) ![Release Date](https://img.shields.io/github/release-date/sagalbot/vue-select?style=flat-square) ![Bundle Size](https://flat.badgen.net/bundlephobia/min/vue-select) ![Monthly Downloads](https://img.shields.io/npm/dm/vue-select.svg?style=flat-square) [![Coverage Status](https://coveralls.io/repos/github/sagalbot/vue-select/badge.svg?branch=master)](https://coveralls.io/github/sagalbot/vue-select?branch=master) ![MIT License](https://img.shields.io/github/license/sagalbot/vue-select.svg?style=flat-square)

> **Everything you wish the HTML `<select>` element could do, wrapped up into a lightweight, zero
> dependency, extensible Vue component.**

**This package is a modified version of vue-select updated to Vue 3 in its most basic form with minor changes we needed for our project.**

Vue Select is a feature-rich select/dropdown/typeahead component. It provides a default
template that fits most use cases for a filterable select dropdown. The component is designed to be as
lightweight as possible, while maintaining high standards for accessibility,
developer experience, and customization.

- Tagging
- Filtering / Searching
- Vuex Support
- AJAX Support
- SSR Support
- Accessible
- ~20kb Total / ~5kb CSS / ~15kb JS
- Select Single/Multiple Options
- Customizable with slots and SCSS variables
- Zero dependencies

## Documentation

Complete documentation and examples available at https://vue-select.org.

- **[API Documentation](https://vue-select.org)**

## Sponsors :tada:

It takes a lot of effort to maintain this project. If it has saved you development time, please consider [sponsoring the project](https://github.com/sponsors/sagalbot)
with GitHub sponsors!

Huge thanks to the [sponsors](https://github.com/sponsors/sagalbot) and [contributors](https://github.com/sagalbot/vue-select/graphs/contributors) that make Vue Select possible!

## Get started

**Vue 3 only**

Install:

```bash
yarn add @rentmagic/vue-select
```


The component itself does not include any CSS. You'll need to include it separately in your Component.vue:
```vue
<style>
@import "vue-select/dist/vue-select.css";
</style>
```

Example usage in a Vue 3 component:

```vue
<template>
  <div>
    <v-select :options="['Option 1', 'Option 2', 'Option 3']"></v-select>
  </div>
</template>

<script>
import VueSelect from "@rentmagic/vue-select";

export default {
  components: {
    'v-select': VueSelect
  }
};
</script>

<style>
@import "vue-select/dist/vue-select.css";
</style>
```


## License

[MIT](https://github.com/sagalbot/vue-select/blob/master/LICENSE.md)
