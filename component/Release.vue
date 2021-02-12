<template>
  <div>
    <form method="post" ref="form" :action="formAction">
      <input
        v-if="resource === 'edit'"
        type="hidden"
        name="_method"
        value="PUT"
      />
      <input type="hidden" name="_token" :value="csrfToken" />

      <!-- ROW -->
      <div class="row">

        <!-- TITLE -->
        <div class="col-md-6">
          <h1 id="language-title" class="heading_grey h3 mt-0 mb-5">
            {{ fullTitle }}
          </h1>
        </div>
        <!-- TITLE -->

         <div class="col-md-6 text-md-right" v-show="showSidebar">
          <button
          @click="toggleLanguage()"
          type="button"
          class="ontario-button ontario-button--primary mr-0 position-relative mb-4"
          id="toggle_button"
        >
          Switch to {{ title == 'English' ? 'French' : 'English' }}
        </button>
        </div>
        <!-- LANGUAGE BUTTON -->


        <!-- LEFT -->
        <div div class="col-lg-8">
          <!-- 
          Previous form submit values take precendence over pre-existing values (an edit).

          If this is within the axios call, the controller does this for us.

          For static components, the value attributes check the following:
            1. if oldData isn't empty, pass the value from oldData by name
            2. if oldData is empty and the edit value exists, pass the value
            3. else, pass null as something must be passed to a prop
         -->

          <!-- Select dropdown that emits event with news type data -->
          <FilterComponent
            :value="{
              selected: oldData['release_type_id'] || selectedNewsType,
            }"
            v-slot="{ filterData, emitData }"
          >
            <!-- News type dropdown -->
            <label for="release_type_id">News Type</label>
            <select
              id="release_type_id"
              name="release_type_id"
              v-model="filterData.selected"
              @change="emitData()"
              class="ontario-input form-control"
            >
              <option
                v-for="news_type in newsTypes"
                :key="news_type.id"
                :value="news_type.id"
              >
                {{ news_type.name }}
              </option>
            </select>
          </FilterComponent>

          <hr />

          <!--
          oldData MUST be sent to the ResultsComponent, the controller does not keep 
          the session data when it performs this axios call.

          Therefore we send back the session data to the controller 
          for previous form submit values.
        -->
          <ResultsComponent
            :endpoint="'/release/get_template_sections/' + releaseId"
            :suppressInitialFetch="true"
            :oldData="oldData"
            class="release"
            v-slot="{ items }"
          >
            <!-- Slot containing release section data -->
            <div
              v-show="item.language == lang || item.language == 'both'"
              v-for="item in items"
              :key="item.label + '_' + item.language"
              class="row"
            >
              <!-- Render function that builds an individual release section label and form element -->
              <ReleaseSection :item="item" :oldData="oldData" :filterData="ministries" class="col-12" />
            </div>
          </ResultsComponent>
        </div>
        <!-- LEFT -->

        <!-- RIGHT -->
        <div
          class="col-xl-3 col-lg-4 offset-xl-1"
          id="create_edit_sidebar"
          v-show="showSidebar"
        >

          <!-- WORKFLOW -->
          <div class="grey_boxes p-3 mb-4 mt-md-2">

            <SelectComponent
              label="Status"
              id="status_type"
              name="release_status_type_id"
              endpoint="/release/get_status_types"
              selectFirstOption
              :filterData="
                release && release.status_type_id
                  ? {
                      status_type_id: release.status_type_id,
                      resource: resource,
                      release_type_id: release.release_type_id,
                    }
                  : { resource: resource }
              "
              :value="
                oldData['release_status_type_id'] ||
                (release ? release.status_type_id : null)
              "
            />

            <label for="release_publish_date">Publish date</label>
            <b-form-datepicker
              id="release_publish_date"
              name="release_publish_date"
              :date-format-options="{ year: 'numeric', month: 'short', day: '2-digit', weekday: 'short' }"
              v-model="publishDate"
              class="mb-4"
            ></b-form-datepicker>

            <label for="release_publish_time">Publish time</label>
            <b-form-timepicker
              id="release_publish_time"
              name="release_publish_time"
              v-model="publishTime"
              class="mb-4"
              locale="en"
            ></b-form-timepicker>

            <!-- DISTRIBUTION -->
            <div v-show="showDistribution">

              <SelectComponent
                label="Distribution"
                id="distribution_type"
                name="release_distribution_id"
                endpoint="/release/get_email_types"
                selectFirstOption
                :value="
                  oldData['release_distribution_id'] ||
                  (release ? release.release_distribution_id : null)
                "
              />
             
              <div v-show="release_distribution_id == '5002'">
                <MultiselectComponent
                  label="Regions"
                  id="release_regions"
                  name="release_regions"
                  endpoint="/release/get_regions"
                  selectPlaceholder="Select region(s)"
                  :value="
                    Object.keys(oldData).length && oldData['release_regions']
                    ? JSON.parse(oldData['release_regions'])
                    : regions
                    ? regions
                    : null
                    "
                />
              </div>

              <div v-show="release_distribution_id == '5002' || release_distribution_id == '5003'">
              
                <SelectComponent
                  label="Language distribution"
                  id="language_type"
                  name="release_language_distribution"
                  endpoint="/release/get_language_distributions"
                  selectFirstOption
                  :filterData=release_status_type_id
                  :value="
                  (oldData['release_language_distribution'] ||
                  (release ? release.release_language_distribution : null))
                "
                />
                
              </div>

            </div>
            <!-- DISTRIBUTION -->

          </div>
          <!-- WORKFLOW -->
          
          <!-- TAGS AND TOPICS -->
          <div class="grey_boxes p-3 mb-4">

            <MultiselectComponent
              label="Topics"
              name="release_topics"
              endpoint="/topic/get_topics"
              selectPlaceholder="Select a topic"
              :value="
                Object.keys(oldData).length && oldData['release_topics']
                  ? JSON.parse(oldData['release_topics'])
                  : topics
                  ? topics
                  : null
              "
            />

            <MultiselectComponent
              label="Tags"
              name="release_tags"
              endpoint="/tag/get_tags"
              searchable
              addTags
              :showLabels="false"
              selectPlaceholder="Search or add a tag"
              :value="
                Object.keys(oldData).length && oldData['release_tags']
                  ? JSON.parse(oldData['release_tags'])
                  : tags
                  ? tags
                  : null
              "
            />

          </div>
          <!-- TAGS AND TOPICS -->

          <div class="text-center mt-3">
            <input type="hidden" name="release_past_news" v-model="pastNews" />
            <button
              @click="submitForm"
              type="button"
              class="ontario-button ontario-button--primary mr-0"
            >
              Submit
            </button>
          </div>

        </div>
        <!-- RIGHT -->

      </div>
      <!-- ROW -->

    </form>
  </div>
</template>

<script>
// Generic
import {
  ReplicatorComponent,
  WYSIWYGComponent,
  FilterComponent,
  ResultsComponent,
  MultiselectComponent,
  SelectComponent,
  EventBus,
} from "vue-components-library";
// Contrib
import ReleaseSection from "./ReleaseSection";
import FieldsetComponent from "./FieldsetComponent";
import EpkCheckboxComponent from "./EpkCheckboxComponent";
import ContactsComponent from "./ContactsComponent.vue";
// Third-party
import { BModal, VBModal, BVModalPlugin } from "bootstrap-vue";

export default {
  props: {
    /**
     * Array of select options for News Types.
     *
     * Exists before the axios call.
     */
    newsTypes: {
      type: Array,
      required: true,
    },
    /**
     * Selected news type.
     *
     * Exists before the axios call.
     */
    selectedNewsType: {
      type: Number,
      default: null,
    },
    /**
     * Resource ('Create' or 'Edit').
     */
    resource: {
      type: String,
      required: true,
    },
    /**
     * Existing edit values.
     */
    release: {
      type: Object,
    },
    /**
     * Existing selected tags.
     *
     * ReleaseApi keywords is not in a format that is useful here
     * (semicolon-delimited labels).
     */
    tags: {
      type: Array,
    },
    /**
     * Existing selected topics.
     *
     * ReleaseApi keywords is not in a format that is useful here
     * (semicolon-delimited labels).
     */
    topics: {
      type: Array,
    },
    /**
     * Existing selected topics.
     *
     * ReleaseApi keywords is not in a format that is useful here
     * (semicolon-delimited labels).
     */
    regions: {
      type: Array,
      default: null,
    },

    role: {
      type: Number,
    },
    /**
     * Data passed from the controller on a failed form submit.
     *
     * Optional, must allow either an object or array (due to Laravel).
     */
    oldData: {
      default: () => [],
    },
  },
  components: {
    ReplicatorComponent,
    ResultsComponent,
    FilterComponent,
    ReleaseSection,
    WYSIWYGComponent,
    SelectComponent,
    FieldsetComponent,
    MultiselectComponent,
    EpkCheckboxComponent,
    ContactsComponent,
  },
  data() {
    return {
      releaseId: this.release ? this.release.id : 0,
      lang: "en",
      title: "English",
      showSidebar: false,
      showDistribution: false,
      showRegions:false,
      release_status_type_id: null,
      release_distribution_id: null,
      release_language_distribution: null,
      csrfToken: document.querySelector('meta[name="csrf-token"]').content,
      leadMinistry: null,
      partnerMinistries: [],
      publishDate:
        this.oldData["release_publish_date"] ??
        (this.release && this.release.release_date_time
          ? this.release.release_date_time.split(" ")[0]
          : null),
      publishTime:
        this.oldData["release_publish_time"] ??
        (this.release && this.release.release_date_time
          ? this.release.release_date_time.split(" ")[1]
          : null),
      pastNews: false
    };
  },
  methods: {
    IsPastOneDayFromNow: function () {
      if(this.publishDate && this.publishTime) 
      {
        var now = new Date().getTime();
        var oneDayFromNow = now - (1000 * 60 * 60 * 24) //Set in the past

        var dateOneDayFromNow = new Date(oneDayFromNow),
        [year, month, day] = this.publishDate.split("-"),
        [hours, minutes] = this.publishTime.split(":"),
        formattedPublishDate = new Date(
          year,
          month - 1,
          day,
          hours,
          minutes
        );
        if(formattedPublishDate < dateOneDayFromNow) // And if my role is just regular contributor
        {
          return true
          // this.release_distribution_id = 5001
          // this.showDistribution = false
          // this.$bvModal
          // .msgBoxConfirm(
          //   "You are not allow to distribute emails past 24 hours. Please contact your SA"
          // )
          // .then((confirmation) => {
          //   if (confirmation)
          //    this.release_distribution_id = 5001;
          //    this.showDistribution = false;
          // });
        }
        else
        {
          return false
          // this.showDistribution = true
        }
      }
      
    },
    toggleLanguage: function () {
      this.lang == "en" ? (this.lang = "fr") : (this.lang = "en");
      this.title == "English"
        ? (this.title = "French")
        : (this.title = "English");
    },
    /**
     * Displays warning modal before form submit when publishing, if:
     * - publish time is less than the current time
     * - deleting the release
     */
    submitForm: function () {
      // Rely on backend validation, will prevent form submission if set to published
      if (!this.publishTime || !this.publishDate) {
        this.$refs.form.submit();
        return;
      }

      // Build Date objects (cannot compare strings)
      var currentDate = new Date(),
        [year, month, day] = this.publishDate.split("-"),
        [hours, minutes] = this.publishTime.split(":"),
        formattedPublishDate = new Date(
          year,
          month - 1,
          day,
          hours,
          minutes
        );
      // Published, published (English only), published (French only)
      if (
        [3002, 3003, 3004].includes(this.release_status_type_id) &&
        formattedPublishDate <= currentDate
      ) {
        const h = this.$createElement
        
        if (this.IsPastOneDayFromNow()) //This is where you check the role id to say if it is admin go to else condition
        {
          var message = h('p', 'You are about to publish immediately, email distribution will not be sent since it has been more than 24 hours.')
          // Boolean to tell controller that this should reset distribution to none and language to null
          this.pastNews = true
        }
        else 
        {
          var message = h('p', 'You are about to publish immediately, would you want to proceed?')
          if(this.release_status_type_id == "3002" && this.release_language_distribution != "7001") 
          {
            var message2 = h('strong', 'Please note: You are about to send an email in one language!')
          }
          this.pastNews = false
        }
          const messageVNode = h('p', { class: ['text-left'] }, [message, message2])
          this.$bvModal
            .msgBoxConfirm([messageVNode])
            .then((confirmation) => {
              if (confirmation) this.$refs.form.submit();
            });
        // Deleted
      } else if (this.release_status_type_id == 3006) {
        this.$bvModal
          .msgBoxConfirm("Are you sure you want to delete?")
          .then((confirmation) => {
            if (confirmation) this.$refs.form.submit();
          });
      } else {
        this.$refs.form.submit();
      }
    },
  },
  computed: {
    fullTitle: function () {
      // prettier-ignore
      return `${this.resource.charAt(0).toUpperCase()}${this.resource.slice(1)} - ${this.title}`
    },
    formAction: function () {
      return `/release${this.releaseId ? `/${this.releaseId}` : ""}`;
    },
    ministries: function () {
      let ministries = []

      if (this.leadMinistry) {
        ministries = ministries.concat(parseInt(this.leadMinistry));
      }
      if (this.partnerMinistries.length) {
        ministries = ministries.concat(this.partnerMinistries)
      }
      
      // return only unique values in the array to avoid duplicate values (reference Set datatype in es6)
      return [...new Set(ministries)];
    },
  },
  created() {
    
    // Listen to news type dropdown event to also toggle sidebar
    EventBus.$on("filter-data", (payload) => this.showSidebar = (payload.selected ? true : false));
    // Listen to status type dropdown event for form submission logic
    EventBus.$on("status_type-change", (payload) => {
      this.release_status_type_id = payload
      if(this.release_status_type_id == '3002' || this.release_status_type_id == '3003' || this.release_status_type_id == '3004')
      {
        this.showDistribution = true
        // this.checkDate();
      }
      if(this.release_status_type_id == '3001') 
      {
        this.showDistribution = false
      }
      
    });

    // Listen to status type dropdown event for form submission logic
    EventBus.$on( "distribution_type-change", (payload) => {
        this.release_distribution_id = payload
    });

    // Listen to status type dropdown event for form submission logic
    EventBus.$on( "language_type-change", (payload) => {
        this.release_language_distribution = payload
    });
  },
  mounted() {
    EventBus.$on('ministry_lead-change', (value) => {
      this.leadMinistry = value
    })

    EventBus.$on('ministry_partner-change', (value) => {
      this.partnerMinistries = value
    })
  },
};
</script>
