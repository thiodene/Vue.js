<script>
export default {
  name: "ReleaseSection",
  props: {
    item: {
      type: Object,
    },
    filterData: {
      type: Array,
    },
    /**
     * Data passed from the controller on a failed form submit.
     * Optional, must allow either an object or array (due to Laravel).
     * @values json-encoded
     */
    oldData: {
      default: () => [],
    },
  },
  methods: {
    nameAttribute: function (
      lang,
      parentLabel,
      index = null,
      childLabel = null
    ) {
      let x = "";
      // prettier-ignore-start
      if (lang == "both") {
        x = parentLabel;
      } else {
        x = lang + "[" + parentLabel + "]";
      }

      if (index != null) {
        x = x + "[" + index + "]";
      }

      if (childLabel) {
        x = x + "[" + childLabel + "]";
      }
      // prettier-ignore-end
      return x;
    },
    fullLanguageName: (lang) => (lang === "en" ? "English" : "French"),
  },
  render: function (createElement) {
    // Created from parent object's children key
    // i is the index of the child within children, and starts at 0. However named slots start at 1, hence i+1
    const dynamicSlots = this.item.children.reduce(
      (dynamicSlots, child, i) => ({
        ...dynamicSlots,
        ["slot_" + (i + 1)]:
          // Lambda implies a return, see more below for syntax examples
          // https://stackoverflow.com/questions/65111687/build-a-dynamic-number-of-vue-slots-within-a-render-function
          ({ index, record, submitName }) => [
            createElement(
              "label",
              {
                attrs: {
                  for: this.nameAttribute(
                    this.item.language,
                    this.item.label,
                    index,
                    child.label
                  ),
                },
              },
              child.name
            ),
            createElement(
              child.tag,
              {
                attrs: {
                  ...child.attributes,
                  ...{
                    // structure may need to change for ministries
                    label: child.name,
                    id: this.nameAttribute(
                      this.item.language,
                      this.item.label,
                      index,
                      child.label
                    ),
                    name: this.nameAttribute(
                      this.item.language,
                      this.item.label,
                      index,
                      child.label
                    ),
                    /**
                     * Reactivity issue - Vue does not update the Replicator's records array for the form submission.
                     *
                     * However the Vue devtools track it correctly, this is isolated to the render function.
                     *
                     * Instead of submitting as one json-encoded value, submit all replicator values as an array:
                     * (e.g. en[quotes_section][0][quote])
                     */
                    value: record[child.label], // pull data from parent record (Replicator) instead of child record
                  },
                },
                on: {
                  input: function (event) {
                    record[child.name.replace(" ", "_")] = event.target.value;
                  },
                },
              },
              // default slot of the child, only used for textarea (submits with inner HTML instead of value attribute)
              record[child.label] // pull content from parent record (Replicator) instead of child record
            ),
          ],
      }),
      {}
    );

    // Not dynamic, hard coded into component (this example is replicator operations)
    const staticSlots = {
      record_operations: ({ record, index, removeRecord }) => {
        return createElement(
          "button",
          {
            attrs: {
              type: "button",
            },
            on: {
              click: function () {
                removeRecord(index);
              },
            },
          },
          "Remove Record"
        );
      },
      component_operations: ({ record, index, addRecord }) => {
        return createElement(
          "button",
          {
            attrs: {
              type: "button",
            },
            on: {
              click: function () {
                addRecord();
              },
            },
          },
          "Add Record"
        );
      },
    };

    /*
     * Use the default slot when the parent tag is a non-form element (e.g. span, div, results-component etc)
     * If parent tag is a component with defined named slots, use dynamicSlots/staticSlots above
     */
    const defaultSlot = [
      // This reduce leaves flexibilty to add more than one element per child, may not be necessary at this point in time
      this.item.children.reduce((x, child, i) => {
        x.push(
          createElement(
            child.tag,
            {
              attrs: {
                ...child.attributes,
                ...{
                  label: child.name,
                  id: this.nameAttribute(
                    child.language,
                    this.item.label,
                    null,
                    child.label
                  ),
                  name: this.nameAttribute(
                    child.language,
                    this.item.label,
                    null,
                    child.label
                  ),
                  value: child.content,
                  // value: Object.keys(this.oldData).length && this.oldData[child.language] && this.oldData[child.language][child.label] !== null ? this.oldData[child.language][child.label] : child.content,
                },
              },
            },
            ""
          )
        );
        return x;
      }, []),
    ];

    /*
     * This is the return of the rendered component, including nested child elements from above (if applicable).
     */
    return createElement(
      "div",
      [
        // hasLabel is an optional attribute on the top-level element. If value is false, don't render the top-level label.
        // e.g. Name: Quotes Section, Tag: replicator-component
        // e.g. Name: Title,          Tag: input
        this.item.attributes.hasLabel == false
          ? null
          : createElement(
              "label",
              {
                attrs: {
                  for: this.nameAttribute(this.item.language, this.item.label),
                },
              },
              // Add language affix to labels if field is not shared between languages
              this.item.language === "both"
                ? this.item.name
                : `${this.item.name} - ${this.fullLanguageName(
                    this.item.language
                  )}`
            ),
        createElement(
          this.item.tag,
          {
            attrs: {
              ...this.item.attributes,
              ...{
                label: this.item.name,
                name: this.nameAttribute(this.item.language, this.item.label),
                id: this.nameAttribute(this.item.language, this.item.label),
                class: this.item.label,
                value: this.item.content,
                filterData: this.filterData,
                // value: Object.keys(this.oldData).length && this.oldData[this.item.language] && this.oldData[this.item.language][this.item.label] !== null ? this.oldData[this.item.language][this.item.label] : this.item.content,
                language: this.item.language,
                // required: (this.item.mandatory_flag ? true : false)
                // oldData: Object.keys(this.oldData).length && this.oldData[this.item.language] && this.oldData[this.item.language][this.item.label] !== null ? this.oldData[this.item.language][this.item.label] : [],
                oldData:
                  Object.keys(this.oldData).length &&
                  this.oldData[this.item.language] &&
                  this.oldData[this.item.language][this.item.label] !== null
                    ? this.oldData
                    : [],
              },
            },
            scopedSlots: { ...dynamicSlots, ...staticSlots }, // handles any child elements
          },
          // default slot, can be an array
          [defaultSlot]
        ),
      ]
      // text-value of parent, could be used for edit form values?
    );
  },
};
</script>
