<template>
  <div>
    <fieldset :id="id" :class="fieldsetClass">
      <legend>{{ label }}</legend>
      <slot></slot>
    </fieldset>
  </div>
</template>

<script>
/**
 * TODO
 *
 * This component does not accept checkbox values directly because
 * the slots are built by the render function (ReleaseSection.vue).
 *
 * This component requires a refactor if the expected scope changes.
 */

import axios from 'axios';

export default {
  name: 'EpkCheckboxComponent',
  props: {
    /**
     * Sets the text of the fieldset legend tag
     */
    'label': {
      type: String
    },
    /**
     * Sets the id of the fieldset element.
     */
    'id' : {
      type: String
    },
    /**
     * The class name provided to the fieldset element.
     */
    'fieldsetClass' : {
      type: String,
      default: 'fieldsetClass'
    },
    /**
     * Data passed from the controller. If present, overrides the selectedValues.
     */
    'value': { } // unused
  },
  data () {
    return {
      /**
        * The records selected by the user.
        */
      selectedValues: [ ],
      /**
        * The records available to the user.
        */
      fieldsetOptions: [ ],
    }
  },
}

/**
 * Register child component globally for use in ReleaseSection.vue
 */
Vue.component('CheckboxComponent', {
  props: {
    /**
     * Sets the text of the label that associates to the checkbox
     */
    'label': {
        type: String
    },
    /**
     * Sets the id of the checkbox element.
     */
    'id' : {
      type: String
    },
    /**
     * Name of the object passed to the controller.
     */
    'name' : {
      type: String,
      required: true
    },
    /**
     * Data passed from the controller. If present, override selected.
     *
     * Received as a string, treated as a boolean.
     */
    'value': { }
  },
  data () {
    return {
      selected: false,
    }
  },
  mounted() {
    /**
     * If value is passed as a property, set selectedValues as a boolean
     *
     * Use the identity operator (===) to cast to boolean  (covers any 
     * string which isn't the empty string evaluating to true).
     */
    if (this.value) this.selected = (this.value === 'true')
  },
  /**
   * Checkboxes don't get submitted to controller if they are falsy (false, 0).
   * However we want to submit a falsy value, so we are submitting via a
   * hidden input field.
   */
  template: `<div>
               <input :name="name" :value="selected" hidden />
               <input :id="id" type="checkbox" v-model="selected"/>
               <label :for="id">{{ label }}</label>
             </div>`
})
</script>
