<style>
@import '../css/vue-select.css';
</style>

<template>
  <div :dir="dir" class="v-select" :class="stateClasses">
    <slot name="header" v-bind="scope.header" />
    <div
      :id="`vs${uid}__combobox`"
      ref="toggle"
      class="vs__dropdown-toggle"
      role="combobox"
      :aria-expanded="dropdownOpen.toString()"
      :aria-owns="`vs${uid}__listbox`"
      :aria-controls="`vs${uid}__listbox`"
      aria-label="Search for option"
      @mousedown="toggleDropdown($event)"
    >
      <div ref="selectedOptions" class="vs__selected-options">
        <slot
          v-for="option in selectedValue"
          name="selected-option-container"
          :option="normalizeOptionForSlot(option)"
          :deselect="deselect"
          :multiple="multiple"
          :disabled="disabled"
        >
          <span :key="getOptionKey(option)" class="vs__selected">
            <slot
              name="selected-option"
              v-bind="normalizeOptionForSlot(option)"
            >
              {{ getOptionLabel(option) }}
            </slot>
            <button
              v-if="multiple"
              ref="deselectButtons"
              :disabled="disabled"
              type="button"
              class="vs__deselect"
              :title="`Deselect ${getOptionLabel(option)}`"
              :aria-label="`Deselect ${getOptionLabel(option)}`"
              @click="deselect(option)"
            >
              <component :is="childComponents.Deselect" />
            </button>
          </span>
        </slot>

        <slot name="search" v-bind="scope.search">
          <input
            class="vs__search"
            v-bind="scope.search.attributes"
            v-on="scope.search.events"
          />
        </slot>
      </div>

      <div ref="actions" class="vs__actions">
        <button
          v-show="showClearButton"
          ref="clearButton"
          :disabled="disabled"
          type="button"
          class="vs__clear"
          title="Clear Selected"
          aria-label="Clear Selected"
          @click="clearSelection"
        >
          <component :is="childComponents.Deselect" />
        </button>

        <slot name="open-indicator" v-bind="scope.openIndicator">
          <component
            :is="childComponents.OpenIndicator"
            v-if="!noDrop"
            v-bind="scope.openIndicator.attributes"
          />
        </slot>

        <slot name="spinner" v-bind="scope.spinner">
          <div v-show="mutableLoading" class="vs__spinner">Loading...</div>
        </slot>
      </div>
    </div>
    <transition :name="transition">
      <ul
        v-if="dropdownOpen"
        :id="`vs${uid}__listbox`"
        ref="dropdownMenu"
        :key="`vs${uid}__listbox`"
        v-append-to-body
        class="vs__dropdown-menu"
        role="listbox"
        tabindex="-1"
        @mousedown.prevent="onMousedown"
        @mouseup="onMouseUp"
      >
        <slot name="list-header" v-bind="scope.listHeader" />
        <li
          v-for="(option, index) in filteredOptions"
          :id="`vs${uid}__option-${index}`"
          :key="getOptionKey(option)"
          role="option"
          class="vs__dropdown-option"
          :class="{
            'vs__dropdown-option--deselect':
              isOptionDeselectable(option) && index === typeAheadPointer,
            'vs__dropdown-option--selected': isOptionSelected(option),
            'vs__dropdown-option--highlight': index === typeAheadPointer,
            'vs__dropdown-option--disabled': !selectable(option),
          }"
          :aria-selected="index === typeAheadPointer ? true : null"
          @mouseover="selectable(option) ? (typeAheadPointer = index) : null"
          @click.prevent.stop="selectable(option) ? select(option) : null"
        >
          <slot name="option" v-bind="normalizeOptionForSlot(option)">
            {{ getOptionLabel(option) }}
          </slot>
        </li>
        <li v-if="filteredOptions.length === 0" class="vs__no-options">
          <slot name="no-options" v-bind="scope.noOptions">
            Sorry, no matching options.
          </slot>
        </li>
        <slot name="list-footer" v-bind="scope.listFooter" />
      </ul>
      <ul
        v-else
        :id="`vs${uid}__listbox`"
        role="listbox"
        style="display: none; visibility: hidden"
      ></ul>
    </transition>
    <slot name="footer" v-bind="scope.footer" />
  </div>
</template>

<script>
import { defineComponent } from 'vue'
import pointerScroll from '../mixins/pointerScroll.js'
import typeAheadPointer from '../mixins/typeAheadPointer.js'
import ajax from '../mixins/ajax.js'
import childComponents from './childComponents.js'
import appendToBody from '../directives/appendToBody.js'
import sortAndStringify from '../utility/sortAndStringify.js'
import uniqueId from '../utility/uniqueId.js'

/**
 * @name VueSelect
 */
export default defineComponent({
  components: { ...childComponents },

  directives: { appendToBody },

  mixins: [pointerScroll, typeAheadPointer, ajax],

  props: {
    value: {},
    components: {
      type: Object,
      default: () => ({}),
    },
    options: {
      type: Array,
      default() {
        return []
      },
    },
    disabled: {
      type: Boolean,
      default: false,
    },
    clearable: {
      type: Boolean,
      default: true,
    },
    deselectFromDropdown: {
      type: Boolean,
      default: false,
    },
    searchable: {
      type: Boolean,
      default: true,
    },
    multiple: {
      type: Boolean,
      default: false,
    },
    placeholder: {
      type: String,
      default: '',
    },
    transition: {
      type: String,
      default: 'vs__fade',
    },
    clearSearchOnSelect: {
      type: Boolean,
      default: true,
    },
    closeOnSelect: {
      type: Boolean,
      default: true,
    },
    label: {
      type: String,
      default: 'label',
    },
    autocomplete: {
      type: String,
      default: 'off',
    },
    reduce: {
      type: Function,
      default: (option) => option,
    },
    selectable: {
      type: Function,
      default: (option) => true,
    },
    getOptionLabel: {
      type: Function,
      default(option) {
        if (typeof option === 'object') {
          if (!option.hasOwnProperty(this.label)) {
            return console.warn(
              `[vue-select warn]: Label key "option.${this.label}" does not` +
              ` exist in options object ${JSON.stringify(option)}.\n` +
              'https://vue-select.org/api/props.html#getoptionlabel'
            )
          }
          return option[this.label]
        }
        return option
      },
    },
    getOptionKey: {
      type: Function,
      default(option) {
        if (typeof option !== 'object') {
          return option
        }

        try {
          return option.hasOwnProperty('id')
            ? option.id
            : sortAndStringify(option)
        } catch (e) {
          const warning =
            `[vue-select warn]: Could not stringify this option ` +
            `to generate unique key. Please provide'getOptionKey' prop ` +
            `to return a unique key for each option.\n` +
            'https://vue-select.org/api/props.html#getoptionkey'
          return console.warn(warning, option, e)
        }
      },
    },
    onTab: {
      type: Function,
      default: function () {
        if (this.selectOnTab && !this.isComposing) {
          this.typeAheadSelect()
        }
      },
    },
    taggable: {
      type: Boolean,
      default: false,
    },
    tabindex: {
      type: Number,
      default: null,
    },
    pushTags: {
      type: Boolean,
      default: false,
    },
    filterable: {
      type: Boolean,
      default: true,
    },
    filterBy: {
      type: Function,
      default(option, label, search) {
        return (
          (label || '')
            .toLocaleLowerCase()
            .indexOf(search.toLocaleLowerCase()) > -1
        )
      },
    },
    filter: {
      type: Function,
      default(options, search) {
        return options.filter((option) => {
          let label = this.getOptionLabel(option)
          if (typeof label === 'number') {
            label = label.toString()
          }
          return this.filterBy(option, label, search)
        })
      },
    },
    createOption: {
      type: Function,
      default(option) {
        return typeof this.optionList[0] === 'object'
          ? { [this.label]: option }
          : option
      },
    },
    resetOnOptionsChange: {
      default: false,
      validator: (value) => ['function', 'boolean'].includes(typeof value),
    },
    clearSearchOnBlur: {
      type: Function,
      default: function ({ clearSearchOnSelect, multiple }) {
        return clearSearchOnSelect && !multiple
      },
    },
    noDrop: {
      type: Boolean,
      default: false,
    },
    inputId: {
      type: String,
    },
    dir: {
      type: String,
      default: 'auto',
    },
    selectOnTab: {
      type: Boolean,
      default: false,
    },
    selectOnKeyCodes: {
      type: Array,
      default: () => [
        13, // enter
      ],
    },
    searchInputQuerySelector: {
      type: String,
      default: '[type=search]',
    },
    mapKeydown: {
      type: Function,
      default: (map, vm) => map,
    },
    appendToBody: {
      type: Boolean,
      default: false,
    },
    calculatePosition: {
      type: Function,
      default(dropdownList, component, { width, top, left }) {
        dropdownList.style.top = top
        dropdownList.style.left = left
        dropdownList.style.width = width
      },
    },
    dropdownShouldOpen: {
      type: Function,
      default({ noDrop, open, mutableLoading }) {
        return noDrop ? false : open && !mutableLoading
      },
    },
    uid: {
      type: [String, Number],
      default: () => uniqueId(),
    },
  },

  data() {
    return {
      search: '',
      open: false,
      isComposing: false,
      pushedTags: [],
      _value: [], // Internal value managed by Vue Select if no `value` prop is passed
    }
  },

  computed: {
    isTrackingValues() {
      return (
        typeof this.value === 'undefined' ||
        this.$props.hasOwnProperty('reduce')
      )
    },
    selectedValue() {
      let value = this.value
      if (this.isTrackingValues) {
        value = this.$data._value
      }

      if (value !== undefined && value !== null && value !== '') {
        return [].concat(value)
      }

      return []
    },
    optionList() {
      return this.options.concat(this.pushTags ? this.pushedTags : [])
    },
    searchEl() {
      return !!this.$slots['search']
        ? this.$refs.selectedOptions.querySelector(
          this.searchInputQuerySelector
        )
        : this.$refs.search
    },
    scope() {
      const listSlot = {
        search: this.search,
        loading: this.loading,
        searching: this.searching,
        filteredOptions: this.filteredOptions,
      }
      return {
        search: {
          attributes: {
            disabled: this.disabled,
            placeholder: this.searchPlaceholder,
            tabindex: this.tabindex,
            readonly: !this.searchable,
            id: this.inputId,
            'aria-autocomplete': 'list',
            'aria-labelledby': `vs${this.uid}__combobox`,
            'aria-controls': `vs${this.uid}__listbox`,
            ref: 'search',
            type: 'search',
            autocomplete: this.autocomplete,
            value: this.search,
            ...(this.dropdownOpen && this.filteredOptions[this.typeAheadPointer]
              ? {
                'aria-activedescendant': `vs${this.uid}__option-${this.typeAheadPointer}`,
              }
              : {}),
          },
          events: {
            compositionstart: () => (this.isComposing = true),
            compositionend: () => (this.isComposing = false),
            keydown: this.onSearchKeyDown,
            keypress: this.onSearchKeyPress,
            blur: this.onSearchBlur,
            focus: this.onSearchFocus,
            input: (e) => (this.search = e.target.value),
          },
        },
        spinner: {
          loading: this.mutableLoading,
        },
        noOptions: {
          search: this.search,
          loading: this.mutableLoading,
          searching: this.searching,
        },
        openIndicator: {
          attributes: {
            ref: 'openIndicator',
            role: 'presentation',
            class: 'vs__open-indicator',
          },
        },
        listHeader: listSlot,
        listFooter: listSlot,
        header: { ...listSlot, deselect: this.deselect },
        footer: { ...listSlot, deselect: this.deselect },
      }
    },
    childComponents() {
      return {
        ...childComponents,
        ...this.components,
      }
    },
    stateClasses() {
      return {
        'vs--open': this.dropdownOpen,
        'vs--single': !this.multiple,
        'vs--multiple': this.multiple,
        'vs--searching': this.searching && !this.noDrop,
        'vs--searchable': this.searchable && !this.noDrop,
        'vs--unsearchable': !this.searchable,
        'vs--loading': this.mutableLoading,
        'vs--disabled': this.disabled,
      }
    },
    searching() {
      return !!this.search
    },
    dropdownOpen() {
      return this.dropdownShouldOpen(this)
    },
    searchPlaceholder() {
      return this.isValueEmpty && this.placeholder
        ? this.placeholder
        : undefined
    },
    filteredOptions() {
      const optionList = [].concat(this.optionList)

      if (!this.filterable && !this.taggable) {
        return optionList
      }

      let options = this.search.length
        ? this.filter(optionList, this.search, this)
        : optionList
      if (this.taggable && this.search.length) {
        const createdOption = this.createOption(this.search)
        if (!this.optionExists(createdOption)) {
          options.unshift(createdOption)
        }
      }
      return options
    },
    isValueEmpty() {
      return this.selectedValue.length === 0
    },
    showClearButton() {
      return (
        !this.multiple && this.clearable && !this.open && !this.isValueEmpty
      )
    },
  },

  watch: {
    options(newOptions, oldOptions) {
      let shouldReset = () =>
        typeof this.resetOnOptionsChange === 'function'
          ? this.resetOnOptionsChange(
            newOptions,
            oldOptions,
            this.selectedValue
          )
          : this.resetOnOptionsChange

      if (!this.taggable && shouldReset()) {
        this.clearSelection()
      }

      if (this.value && this.isTrackingValues) {
        this.setInternalValueFromOptions(this.value)
      }
    },
    value: {
      immediate: true,
      handler(val) {
        if (this.isTrackingValues) {
          this.setInternalValueFromOptions(val)
        }
      },
    },
    multiple() {
      this.clearSelection()
    },
    open(isOpen) {
      this.$emit(isOpen ? 'open' : 'close')
    },
    search(search) {
      if (search.length) {
        this.open = true
      }
    },
  },

  created() {
    this.mutableLoading = this.loading
  },

  mounted() {
    this.$emit('option:created', this.pushTag)
  },

  methods: {
    setInternalValueFromOptions(value) {
      if (Array.isArray(value)) {
        this.$data._value = value.map((val) =>
          this.findOptionFromReducedValue(val)
        )
      } else {
        this.$data._value = this.findOptionFromReducedValue(value)
      }
    },
    select(option) {
      this.$emit('option:selecting', option)
      if (!this.isOptionSelected(option)) {
        if (this.taggable && !this.optionExists(option)) {
          this.$emit('option:created', option)
        }
        if (this.multiple) {
          option = this.selectedValue.concat(option)
        }
        this.updateValue(option)
        this.$emit('option:selected', option)
      } else if (
        this.deselectFromDropdown &&
        (this.clearable || (this.multiple && this.selectedValue.length > 1))
      ) {
        this.deselect(option)
      }
      this.onAfterSelect(option)
    },
    deselect(option) {
      this.$emit('option:deselecting', option)
      this.updateValue(
        this.selectedValue.filter((val) => {
          return !this.optionComparator(val, option)
        })
      )
      this.$emit('option:deselected', option)
    },
    clearSelection() {
      this.updateValue(this.multiple ? [] : null)
    },
    onAfterSelect(option) {
      if (this.closeOnSelect) {
        this.open = !this.open
      }

      if (this.clearSearchOnSelect) {
        this.search = ''
      }
      if (this.noDrop && this.multiple) {
        this.$nextTick(() => this.$refs.search.focus())
      }
    },
    updateValue(value) {
      if (typeof this.value === 'undefined') {
        this.$data._value = value
      }

      if (value !== null) {
        if (Array.isArray(value)) {
          value = value.map((val) => this.reduce(val))
        } else {
          value = this.reduce(value)
        }
      }

      this.$emit('input', value)
    },
    toggleDropdown(event) {
      const targetIsNotSearch = event.target !== this.searchEl
      if (targetIsNotSearch) {
        event.preventDefault()
      }

      const ignoredButtons = [
        ...(this.$refs['deselectButtons'] || []),
        ...([this.$refs['clearButton']] || []),
      ]

      if (
        this.searchEl === undefined ||
        ignoredButtons
          .filter(Boolean)

          .some((ref) => ref.contains(event.target) || ref === event.target)
      ) {
        event.preventDefault()
        return
      }

      if (this.open && targetIsNotSearch) {
        this.searchEl.blur()
      } else if (!this.disabled) {
        this.open = true
        this.searchEl.focus()
      }
    },

    /**
     * Check if the given option is currently selected.
     * @param  {Object|String}  option
     * @return {Boolean}        True when selected | False otherwise
     */
    isOptionSelected(option) {
      return this.selectedValue.some((value) =>
        this.optionComparator(value, option)
      )
    },

    /**
     *  Can the current option be removed via the dropdown?
     */
    isOptionDeselectable(option) {
      return this.isOptionSelected(option) && this.deselectFromDropdown
    },

    /**
     * Determine if two option objects are matching.
     *
     * @param a {Object}
     * @param b {Object}
     * @returns {boolean}
     */
    optionComparator(a, b) {
      return this.getOptionKey(a) === this.getOptionKey(b)
    },

    /**
     * Finds an option from the options
     * where a reduced value matches
     * the passed in value.
     *
     * @param value {Object}
     * @returns {*}
     */
    findOptionFromReducedValue(value) {
      const predicate = (option) =>
        JSON.stringify(this.reduce(option)) === JSON.stringify(value)

      const matches = [...this.options, ...this.pushedTags].filter(predicate)

      if (matches.length === 1) {
        return matches[0]
      }

      /**
       * This second loop is needed to cover an edge case where `taggable` + `reduce`
       * were used in conjunction with a `create-option` that doesn't create a
       * unique reduced value.
       * @see https://github.com/sagalbot/vue-select/issues/1089#issuecomment-597238735
       */
      return (
        matches.find((match) =>
          this.optionComparator(match, this.$data._value)
        ) || value
      )
    },

    /**
     * 'Private' function to close the search options
     * @emits  {search:blur}
     * @returns {void}
     */
    closeSearchOptions() {
      this.open = false
      this.$emit('search:blur')
    },

    /**
     * Delete the value on Delete keypress when there is no
     * text in the search input, & there's tags to delete
     * @return {this.value}
     */
    maybeDeleteValue() {
      if (
        !this.searchEl.value.length &&
        this.selectedValue &&
        this.selectedValue.length &&
        this.clearable
      ) {
        let value = null
        if (this.multiple) {
          value = [
            ...this.selectedValue.slice(0, this.selectedValue.length - 1),
          ]
        }
        this.updateValue(value)
      }
    },

    /**
     * Determine if an option exists
     * within this.optionList array.
     *
     * @param  {Object || String} option
     * @return {boolean}
     */
    optionExists(option) {
      return this.optionList.some((_option) =>
        this.optionComparator(_option, option)
      )
    },

    /**
     * Ensures that options are always
     * passed as objects to scoped slots.
     * @param option
     * @return {*}
     */
    normalizeOptionForSlot(option) {
      return typeof option === 'object' ? option : { [this.label]: option }
    },

    /**
     * If push-tags is true, push the
     * given option to `this.pushedTags`.
     *
     * @param  {Object || String} option
     * @return {void}
     */
    pushTag(option) {
      this.pushedTags.push(option)
    },

    /**
     * If there is any text in the search input, remove it.
     * Otherwise, blur the search input to close the dropdown.
     * @return {void}
     */
    onEscape() {
      if (!this.search.length) {
        this.open = false
      } else {
        this.search = ''
      }
    },

    /**
     * Close the dropdown on blur.
     * @emits  {search:blur}
     * @return {void}
     */
    onSearchBlur() {
      if (this.mousedown && !this.searching) {
        this.mousedown = false
      } else {
        const { clearSearchOnSelect, multiple } = this
        if (this.clearSearchOnBlur({ clearSearchOnSelect, multiple })) {
          this.search = ''
        }
        this.closeSearchOptions()
        return
      }
      // Fixed bug where no-options message could not be closed
      if (this.search.length === 0 && this.options.length === 0) {
        this.closeSearchOptions()
        return
      }
    },

    /**
     * Open the dropdown on focus.
     * @emits  {search:focus}
     * @return {void}
     */
    onSearchFocus() {
      this.open = true
      this.$emit('search:focus')
    },

    /**
     * Event-Handler to help workaround IE11 (probably fixes 10 as well)
     * firing a `blur` event when clicking
     * the dropdown's scrollbar, causing it
     * to collapse abruptly.
     * @see https://github.com/sagalbot/vue-select/issues/106
     * @return {void}
     */
    onMousedown() {
      this.mousedown = true
    },

    /**
     * Event-Handler to help workaround IE11 (probably fixes 10 as well)
     * @see https://github.com/sagalbot/vue-select/issues/106
     * @return {void}
     */
    onMouseUp() {
      this.mousedown = false
    },

    /**
     * Search <input> KeyBoardEvent handler.
     * @param {KeyboardEvent} e
     * @return {Function}
     */
    onSearchKeyDown(e) {
      const preventAndSelect = (e) => {
        e.preventDefault()
        return !this.isComposing && this.typeAheadSelect()
      }

      const defaults = {
        //  backspace
        8: (e) => this.maybeDeleteValue(),
        //  tab
        9: (e) => this.onTab(),
        //  esc
        27: (e) => this.onEscape(),
        //  up.prevent
        38: (e) => {
          e.preventDefault()
          if (!this.open) {
            this.open = true
            return
          }
          return this.typeAheadUp()
        },
        //  down.prevent
        40: (e) => {
          e.preventDefault()
          if (!this.open) {
            this.open = true
            return
          }
          return this.typeAheadDown()
        },
      }

      this.selectOnKeyCodes.forEach(
        (keyCode) => (defaults[keyCode] = preventAndSelect)
      )

      const handlers = this.mapKeydown(defaults, this)

      if (typeof handlers[e.keyCode] === 'function') {
        return handlers[e.keyCode](e)
      }
    },

    /**
     * @todo: Probably want to add a mapKeyPress method just like we have for keydown.
     * @param {KeyboardEvent} e
     */
    onSearchKeyPress(e) {
      if (!this.open && e.keyCode === 32) {
        e.preventDefault()
        this.open = true
      }
    },
  },
})
</script>
