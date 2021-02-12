<template>
  <div>
    <fieldset :id="id" :class="fieldsetClass">
      <legend>{{ legend }}</legend>
      <div v-for="option in fieldsetOptions" :key="option.value">
        <label>{{ option.label }}</label>
        <input type="radio" :name="name" v-model="selectedOption" :value="option.value">
      </div>
    </fieldset>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'FieldsetComponent',
  props: {
    /**
     * Sets the text of the fieldset legend tag
     */
    'legend': {
      type: String
    },
    /**
     * Web route that pulls data to populate the fieldset form elements, following the format:
     * e.g. [{'label': 'MCU', 'value': 1},{'label': 'MLTSD', 'value': 2}]
     *
     * Optional, must have either endpoint or options.
     */
    'endpoint': {
      type: String
    },
    /**
     * Alternative way of populating the fieldset form elements, following the same format:
     * e.g. [{'label': 'MCU', 'value': 1},{'label': 'MLTSD', 'value': 2}]
     *
     * Optional, must have either endpoint or options.
     */
    'options': {
      type: Array
    },
    /**
     * The class name provided to the fieldset element.
     */
    'fieldsetClass' : {
      type: String,
      default: 'fieldsetClass'
    },
    /**
     * Sets the id of the select element.
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
     * Data passed from the controller. If present, overrides the selectedOption.
     */
    'value': { 
      type: String
    }
  },
  data () {
      return {
        selectedOption: null,
        fieldsetOptions: [],
      }
  },
  methods: {
    fetchResults(endpoint, filterData = null) {
      axios
        .get(endpoint, {
          params: {
            filterData: filterData
          }
        })
        .then(response => {
          this.selectOptions = response.data
        })
    },
  },
  mounted() {
    // If options are passed as a property do not fetch options from an endpoint.
    if (this.options) {
      this.fieldsetOptions = this.options
    } else {
      this.fetchResults(this.endpoint)
    }

    // If value is passed as a property, set selectedOption.
    if (this.value) {
      this.selectedOption = this.value
    }
  }
}
</script>
