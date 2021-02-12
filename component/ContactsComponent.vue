<template>
  <div>
    <SelectComponent :options="selectOptions" id="contacts" name="contacts_select" />

    <wysiwyg-component
      ref="wysiwyg"
      type="contacts"
      :id="id"
      :name="name"
      :value="value"
      suppressInitialDisplay
    ></wysiwyg-component>
  </div>
</template>

<script>
import { WYSIWYGComponent, EventBus, SelectComponent } from "vue-components-library"

import axios from "axios"

export default {
  name: "ContactsComponent",
  props: {
    /**
     * Passed as the id for the wysiwyg editor.
     */
    id: {
      type: String,
    },
    /**
     * Passed as the name for the wysiwyg editor.
     */
    name: {
      type: String,
    },
    /**
     * Used as parameter in axios call to contacts endpoint.
     */
    language: {
      type: String,
      default: "en",
    },
    'filterData' : {
      type: Array
    },
    /**
     * Data passed from the controller.
     */
    value: {
      default: "",
    },
  },
  components: {
    WYSIWYGComponent: WYSIWYGComponent,
    SelectComponent: SelectComponent,
  },
  data() {
    return {
      contacts: [],
      selectOptions: [],
    }
  },
  watch: {
    // Generate selectOptions from contacts
    contacts: function () {
      const selectOptions = this.contacts.map(contact => {
        const option = {}
        option.label = `${contact.first_name} ${contact.last_name}`
        option.value = contact.email
        return option
      })
      this.selectOptions = selectOptions
    },
    filterData: {
      handler: 'fetchResults',
      deep: true
      // this.fetchResults()
    }
  },
  methods: {
    fetchResults: function () {
      axios
        .get("/contact/get_contacts_by_ministry", {
          params: {
            ministry_ids: this.filterData,
            lang: this.language
          }
        })
        .then(response => {
          this.contacts = response.data
        })
    },
    // Creating stringified html using the contact's values to send to ckeditor.
    appendContact: function (value) {
      let contact = this.contacts.find(c => c.email == value)

      // extra spacing will be interpreted in result, but ckeditor strips this out
      let ckeditorValue = `<strong>${contact.first_name} ${contact.last_name}</strong><br>
                          ${contact.title}<br>
                          <a href="mailto:${contact.email}">${contact.email}</a><br>
                          <a href="tel:${contact.phone_number}">${contact.phone_number}</a><br>`

      // Append contact to WYSIWYG
      if (!this.$refs.wysiwyg.editorData) this.$refs.wysiwyg.editorData = ckeditorValue
      else this.$refs.wysiwyg.editorData += ckeditorValue
      this.$refs.wysiwyg.editorVisible = true
    },
  },
  mounted() {
    EventBus.$on('contacts-change', (value) => {
      if (value) {
        this.appendContact(value)
      }
    })
  },
}
</script>
