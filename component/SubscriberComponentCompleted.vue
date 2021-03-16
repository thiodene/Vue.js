<template>
  <div>
    <div class="row">
      <div class="col-lg-9">
        <h1
          class="heading_grey h3"
          role="heading"
        >{{ $t("main_header") }} {{ application.name }}</h1>
      </div>
    </div>

    <div id="steps" v-html="progress" class="mt-5 mb-5"></div>
    <div id="hidden_steps" v-html="stepHidden" class="sr-only" ref="focus" :tabindex="tabindex"></div>

    <!-- ERROR MESSAGE  -->
    <div v-if="errors.length" role="alert" tabindex="0">
      <div class="col-12 ontario-alert ontario-alert--error">
        <div class="ontario-alert__header">
          <div class="ontario-alert__header-icon">
            <img :src="styles_repo_url + '/assets/ontario-icon-alert-error.png'" alt="Error" />
          </div>
          <h2 class="ontario-alert__header-title ontario-h4">Please Fix the Following:</h2>
        </div>
        <div class="ontario-alert__body">
          <ul>
            <li v-for="(error,index) in errors" :key="index">{{ error }}</li>
          </ul>
        </div>
      </div>
    </div>

    <!-- STEP 1 -->
    <section v-if="currentStep == 1">
      <div class="grey_boxes p-4 mb-5 mt-4">
        <div class="row">
          <div class="col-12">
            
            <h2 class="h4" tabindex="0" aria-label="You are currently at step 1 of 5, Email Signup.">Email Signup</h2>

            <div class="ontario-form-group">
              <label class="ontario-label" for="email">
                Email
                <span class="ontario-label__flag">(required)</span>
              </label>
              <input v-model="form.email" type="text" class="ontario-input" name="email" id="email" />
              <label class="ontario-label" for="confirm-email">
                Confirm Email
                <span class="ontario-label__flag">(required)</span>
              </label>
              <input
                v-model="form.emailConfirm"
                type="text"
                class="ontario-input"
                name="confirm-email"
                id="confirm-email"
              />
            </div>
            <div class="ontario-form-group mt-4">
              <fieldset class="ontario-fieldset">
                <legend class="ontario-label" for="identify-media-group-label">
                  Are you a member of the media?
                  <span class="ontario-label__flag">(required)</span>
                </legend>
                <div class="ontario-radios">
                  <div class="ontario-radios__item">
                    <input
                      class="ontario-radios__input"
                      id="identify-media-radio-1"
                      name="radio-buttons"
                      type="radio"
                      value="1"
                      v-model="form.media"
                    />
                    <label
                      class="ontario-label ontario-radios__label"
                      for="identify-media-radio-1"
                    >Yes</label>
                  </div>
                  <div class="ontario-radios__item">
                    <input
                      class="ontario-radios__input"
                      id="identify-media-radio-2"
                      name="radio-buttons"
                      type="radio"
                      value="0"
                      v-model="form.media"
                    />
                    <label
                      class="ontario-label ontario-radios__label"
                      for="identify-media-radio-2"
                    >No</label>
                  </div>
                </div>
              </fieldset>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- STEP 2 -->
    <!-- only shows when media is selected in step 1 -->
    <section v-if="currentStep == 2">
      <div class="grey_boxes p-4 mb-5 mt-4">
        <div class="row">
          <div class="col-12">
            <h2 class="h4">Subscriber Information</h2>
            <div class="ontario-form-group">
              <label class="ontario-label" for="media-organization">
                Organization
                <span class="ontario-label__flag">(required)</span>
              </label>
              <input
                v-model="form.organization"
                type="type"
                class="ontario-input"
                name="media-organization"
                id="media-organization"
              />
            </div>
            <fieldset>
              <legend class="sr-only">media regions</legend>

              <label class="ontario-label" for="media-region-dropdown">
                Region
                <span class="ontario-label__flag">(required)</span>
              </label>

              <select
                class="ontario-input ontario-dropdown"
                v-model="form.mediaRegion"
                id="media-region-dropdown"
                size="1"
                name="media-region-dropdown"
              >
                <option value="Select" selected>Select</option>
                <option
                  v-for="region in regions"
                  :key="region.label"
                  :value="region.value"
                >{{region.label}}</option>
              </select>
            </fieldset>
            <div class="ontario-form-group">
              <label class="ontario-label" for="media-first-name">
                First Name
                <span class="ontario-label__flag">(optional)</span>
              </label>
              <input
                v-model="form.firstName"
                type="text"
                class="ontario-input"
                name="media-first-name"
                id="media-first-name"
              />

              <label class="ontario-label" for="media-last-name">
                Last Name
                <span class="ontario-label__flag">(optional)</span>
              </label>
              <input
                v-model="form.lastName"
                type="text"
                class="ontario-input"
                name="media-last-name"
                id="media-last-name"
              />

              <label class="ontario-label" for="media-phone-number">
                Phone Number
                <span class="ontario-label__flag">(optional)</span>
              </label>
              <input
                v-model="form.phoneNumber"
                type="text"
                class="ontario-input"
                name="media-phone-number"
                id="media-phone-number"
              />
            </div>
            <div class="ontario-form-group mt-4">
              <fieldset class="ontario-fieldset">
                <legend class="ontario-label">
                  Media Type(s)
                  <span class="ontario-label__flag">(required)</span>
                </legend>
                <div class="ontario-checkboxes mt-3 mb-0">
                  <div v-for="type in media_types" :key="type.id" class="ontario-checkboxes__item">
                    <input
                      v-model="form.mediaTypes"
                      type="checkbox"
                      :value="type.id"
                      :id="type.id"
                      :name="type.id"
                      :true-value="type.id"
                      false-value="no"
                      class="ontario-checkboxes__input"
                    />
                    <label class="ontario-checkboxes__label" :for="type.id">{{ type.name }}</label>
                  </div>
                </div>
              </fieldset>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- STEP 3 -->
    <!-- only shows if media is NOT selected in step 1 -->
    <section v-if="currentStep == 3">
      <div class="grey_boxes p-4 mb-5 mt-4">
        <div class="row">
          <div class="col-12">
            <h2 class="h4">Subscriber Information (optional)</h2>

            <div class="ontario-form-group">
              <div v-for="demographic in demographics" :key="demographic.id">
                <fieldset v-if="demographic.input_type == 'radio'" class="ontario-fieldset mt-4">
                  <legend class="ontario-label" tabindex="0">
                    {{ demographic.label }}
                    <span class="ontario-label__flag">(optional)</span>
                  </legend>
                  <div class="ontario-radios mb-0">
                    <div
                      v-for="child in demographic.children"
                      :key="child.id"
                      class="ontario-radios__item"
                    >
                      <input
                        v-model="form.demographics[demographic.name]"
                        :type="child.input_type"
                        v-bind:value="child.id"
                        :true-value="child.id"
                        false-value="no"
                        class="ontario-radios__input"
                        :id="child.id"
                      />
                      <label
                        class="ontario-radios__label"
                        :for="child.id"
                      >{{ child.name }}</label>
                    </div>
                  </div>
                </fieldset>

                <fieldset v-else-if="demographic.input_type == 'select'" class="ontario-fieldset">
                  <legend class="sr-only">regions</legend>
                  <label class="ontario-label" for="demographic-dropdown">
                    {{ demographic.label }}
                    <span class="ontario-label__flag">(optional)</span>
                  </label>

                  <select
                    class="ontario-input ontario-dropdown mb-4"
                    v-model="form.demographics[demographic.name]"
                    id="demographic-dropdown"
                    size="1"
                    name="demographic-dropdown"
                  >
                    
                    <option
                      v-for="child in demographic.children"
                      :key="child.id"
                      :value="child.id"
                    >{{child.name}}</option>
                  </select>
                </fieldset>
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- STEP 4 -->
    <section v-if="currentStep == 4">
        <div class="mt-2 mb-2">
          <div class="grey_boxes p-4 mb-5 mt-4 col-12">
            <h2 class="h4">Customize Subscriptions</h2>
            <p>Want to subscribe to all news? Press ‘Next’ to continue.</p>
            <p>Click ‘Add Customized Subscription’ to customize your email alerts by news types, ministries, topics and regions.</p>
      
            <button
              class="ontario-button ontario-button--primary mt-0 mr-0 mb-4"
             
              v-on:click="isHidden = false"
            >+ Add Customized Subscription</button>
         

            <!-- ERRORS -->
              <div v-if="addSubErrors.length" class="col-12 ontario-alert ontario-alert--error" tabindex="-2">
                <div class="ontario-alert__header">
                  <div class="ontario-alert__header-icon">
                    <img :src="styles_repo_url + '/assets/ontario-icon-alert-error.png'" alt="Error" />
                  </div>
                  <h2 class="ontario-alert__header-title ontario-h4">Please Fix the Following:</h2>
                </div>
                <div class="ontario-alert__body">
                  <ul style="color:red">
                    <li v-for="(error,index) in addSubErrors" :key="index">{{ error }}</li>
                  </ul>
                </div>
              </div>
              <!-- END ERRORS -->

          
            <div v-if="!isHidden" >
              <!-- <h3 class="h5">Add Customized Subscription</h3> -->

              <span v-for="category in categories" :key="category.value" class="row ml-0 mr-0">
                <!-- <MultiselectComponent
                  class="mb-4 col-8"
                  :id="category.label"
                  :label="category.label"
                  :for="category.label"
                  :name="category.label"
                  :options="category.tags"
                  :showLabels="false"
                  searchable
                  selectPlaceholder="select option"
                /> -->

                <label class="ontario-label" :for="category.label">
                  {{category.label}}
                </label>
                <select
                  class="ontario-input ontario-dropdown"
                  v-model="form.tags[category.label]"
                  :id="category.label"
                  size="1"
                  :name="category.label"
                >
                  <option
                    v-for="tag in category.tags"
                    :key="tag.label"
                    :value="[tag.value]"
                  >{{tag.label}}</option>
                </select>
              </span>

              <button
                class="ontario-button ontario-button--primary mb-4 d-block"
                @click="formAddSubscription">Confirm
              </button>
            </div>
          </div> <!--grey_boxes -->


          <div v-if="form.categories.length > 0" class="table-responsive">
           <h2 class="h4">Your Subscriptions</h2>
            <b-table-simple hover bordered responsive role="table">
              <b-thead head-variant="dark">
                <b-tr>
                  <b-th
                    class="customizeSubscription-criteria-width"
                    v-for="(category,index) in form.categories"
                    :key="index"
                  >{{category}}</b-th>
                  <b-th class="customizeSubscription-actions-width">Actions</b-th>
                </b-tr>
              </b-thead>

              <b-tbody>
                <b-tr tabindex="0" v-for="(distribution,index) in distributions" :key="index">
                  <b-td v-for="(category,index) in distribution" :key="index">
                    <span v-for="(tag,index) in category" :key="index">
                      <p class="m-0 mt-2">{{tag.label}}</p>
                    </span>
                  </b-td>
                  <b-td class="text-center">
                    <img
                      @click.prevent="removeSubscription(index)"
                      :src="styles_repo_url + '/assets/ontario-icon-delete.png'"
                      alt="Delete"
                      class="actions"
                      data-toggle="tooltip"
                      data-placement="top"
                      title="Delete Subscription"
                      data-trigger="hover"
                      tabindex="0"
                    />
                  </b-td>
                </b-tr>
                <b-tr tabindex="0" v-if="subscribeAll == true">
                  <b-td colspan="4">
                    <div>
                      <p class="mb-0">
                        You are subscribed to
                        <span class="font-weight-bold">All News</span>
                      </p>
                    </div>
                  </b-td>
                  <b-td class="text-center">
                    <img
                      @click.prevent="removeSubscription(index)"
                      :src="styles_repo_url + '/assets/ontario-icon-delete-blue.png'"
                      alt="Delete"
                      class="actions"
                      data-toggle="tooltip"
                      data-placement="top"
                      title="Delete Subscription"
                    />
                  </b-td>
                </b-tr>
             </b-tbody>
            </b-table-simple>
          </div>
          <div v-else-if="subscribeAll">
            <p>You are subscribed to <span class="font-weight-bold">All News</span></p>
          </div>

        <!-- MODAL for adding a subscription -->

        <!-- <b-modal id="add-subscription" no-close-on-backdrop centered hide-footer>
          <template #modal-title>Add Customized Subscription</template> -->
          <!-- ERRORS -->
          <!-- <div v-if="addSubErrors.length" class="col-12 ontario-alert ontario-alert--error">
            <div class="ontario-alert__header">
              <div class="ontario-alert__header-icon">
                <img :src="styles_repo_url + '/assets/ontario-icon-alert-error.png'" alt="Error" />
              </div>
              <h2 class="ontario-alert__header-title ontario-h4">Please Fix the Following:</h2>
            </div>
            <div class="ontario-alert__body">
              <ul style="color:red">
                <li v-for="(error,index) in addSubErrors" :key="index">{{ error }}</li>
              </ul>
            </div>
          </div> -->
          <!-- END ERRORS -->

          <!-- <span v-for="category in categories" :key="category.value">
            <MultiselectComponent
              class="mb-4"
              :label="category.label"
              :name="category.label"
              :options="category.tags"
              :showLabels="false"
              searchable
              selectPlaceholder="select option"
            />
          </span>
          <button
            class="ontario-button ontario-button--primary mb-0"
            @click="formAddSubscription"
          >Confirm</button>
        </b-modal> -->
      </div>
    </section>

    <!-- STEP 5 -->
    <section v-if="currentStep == 5">
      <div class="grey_boxes p-4 mb-5 mt-4">
        <div class="row">
          <div class="col-12">
            <h2 class="h4">Confirmation</h2>
          </div>
          <div class="col-12">
            <table class="table table-hover table-bordered" role="table">
              <thead>
                <tr tabindex="0">
                  <th colspan="2">Email for Subscription</th>
                </tr>
              </thead>
              <tbody>
                <tr tabindex="0">
                  <td class="table-options">Email</td>
                  <td class="table-response">{{ form.email }}</td>
                </tr>
                <tr tabindex="0">
                  <td>Are you a member of the media?</td>
                  <td v-if="form.media == 1">Yes</td>
                  <td v-else>No</td>
                </tr>
              </tbody>
            </table>
            <table v-if="form.media == 1" class="table table-hover table-bordered" role="table">
              <thead>
                <tr tabindex="0">
                  <th colspan="2">Subscriber Information</th>
                </tr>
              </thead>
              <tbody>
                <tr tabindex="0">
                  <td class="table-options">Organization</td>
                  <td class="table-response">{{ form.organization }}</td>
                </tr>
                <tr tabindex="0">
                  <td class="table-options">Region</td>
                  <td class="table-response">
                    <span v-for="(name,index) in getDemoName(form.mediaRegion)" :key="index">
                      <div style="display:inline-block;">{{ name }}</div>
                    </span>
                  </td>
                </tr>
                <tr tabindex="0">
                  <td class="table-options">First Name</td>
                  <td class="table-response" v-if="form.firstName">{{ form.firstName }}</td>
                  <td class="table-response" v-else>N/A</td>
                </tr>
                <tr tabindex="0"> 
                  <td class="table-options">Last Name</td>
                  <td class="table-response" v-if="form.lastName">{{ form.lastName }}</td>
                  <td class="table-response" v-else>N/A</td>
                </tr>
                <tr tabindex="0">
                  <td class="table-options">Phone Number</td>
                  <td class="table-response" v-if="form.phoneNumber">{{ form.phoneNumber }}</td>
                  <td class="table-response" v-else>N/A</td>
                </tr>
                <tr tabindex="0">
                  <td class="table-options">Media Type</td>
                  <td class="table-response">
                    <ul class="mb-0 confirmTable-ulpadding">
                      <span v-for="type in selectedMediaTypes" :key="type.id">
                        <li>{{ type.name }}</li>
                      </span>
                    </ul>
                  </td>
                </tr>
              </tbody>
            </table>

            <table v-else class="table table-hover table-bordered" role="table">
              <thead>
                <tr tabindex="0">
                  <th colspan="2">Subscriber Information</th>
                </tr>
              </thead>
              <tbody>
                <tr tabindex="0" v-for="demo in demographics" :key="demo.id">
                  <td class="table-options">{{ demo.name }}</td>

                  <td class="table-response">
                    <p class="mb-0" v-if="getDemoName(form.demographics[demo.name]).length">
                      <span
                        v-for="(name,index) in getDemoName(form.demographics[demo.name])"
                        :key="index"
                      >{{ name }}</span>
                    </p>
                    <p class="mb-0" v-else>N/A</p>
                  </td>
                </tr>
              </tbody>
            </table>

            <div v-if="subscribeAll">
              <table class="table table-hover table-bordered" role="table">
                <thead>
                  <tr tabindex="0">
                    <th colspan="2">Customize Subscriptions</th>
                  </tr>
                </thead>
                <tbody>
                  <tr>
                    <td colspan="2">
                      You are subscribed to
                      <span class="font-weight-bold">All News</span>
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>
            <div v-else>
              <b-table-simple hover bordered responsive role="table">
                <b-thead>
                  <b-tr tabindex="0">
                    <b-th colspan="4">Customize Subscriptions</b-th>
                  </b-tr>
                </b-thead>

                <b-tbody>
                  <b-tr tabindex="0">
                    <b-td
                      v-for="(category,index) in form.categories"
                      :key="index"
                      class="bg_dkteal text-white"
                    >{{category}}</b-td>
                  </b-tr>
                  <b-tr tabindex="0" v-for="(distribution,index) in distributions" :key="index">
                    <b-td
                      v-for="(category,index) in distribution"
                      :key="index"
                      class="customizeSubscription-confirm-width"
                    >
                      <span v-for="(tag,index) in category" :key="index">
                        {{tag.label}}
                        <br />
                      </span>
                    </b-td>
                  </b-tr>
                </b-tbody>
              </b-table-simple>
            </div>
          </div>
        </div>
      </div>
    </section>
    
    <!-- STEP 6 -->
    <section v-if="currentStep == 6">
      <div class="grey_boxes p-4 mb-5 mt-4">
        <div class="row">
          <div class="col-12">
            <h2 class="h4">Informed Consent</h2>
            <p>
              <span v-html="application.noc" tabindex="0"></span>
            </p>
          </div>
        </div>
      </div>
    </section>

    <!-- STEP 7 -->
    <div v-if="currentStep == 7">
      <section>
        <div class="grey_boxes p-4 mb-5 mt-4">
          <div class="row">
            <!-- IF ERROR ON SUBMIT -->
            <div v-if="submitError == true" class="col-12">
              <h2 class="h4" ref="focus" :tabindex="tabindex">Error</h2>
              <p>An error has occurred. Please contact <a href="mailto:newsroomhelp@ontario.ca">newsroomhelp@ontario.ca</a>.</p>
            </div>
            <!-- IF UPDATE -->
            <div v-else-if="update == true" class="col-12">
              <h2 class="h4" ref="focus" :tabindex="tabindex">Changes to your Subscription</h2>
              <p>Your subscription preferences have been updated.</p>
            </div>
            <!-- NEW SUBSCRIBER -->
            <div v-else class="col-12">
              <h2 class="h4" ref="focus" :tabindex="tabindex">Confirm your Subscription</h2>
              <p>We have sent an email with a confirmation link to {{form.email}}. Please check your mailbox and click the confirmation link to finish subscribing.</p>
            </div>

          </div>
        </div>
      </section>
    </div>

    <!-- Do not show buttons on last step -->
    <div v-else class="mb-5">
      <button
        class="ontario-button ontario-button--secondary"
        v-if="currentStep != 1"
        @click.prevent="backStep"
        @click="scrollToTop();setFocus()"
      >Back</button>

      <button
        class="ontario-button ontario-button--primary"
        v-if="currentStep != totalSteps"
        @click.prevent="nextStep"
        @click="scrollToTop();setFocus()"
      >Next</button>

      <button
        class="ontario-button ontario-button--primary"
        v-if="currentStep == totalSteps"
        @click.prevent="formSubmit"
        @click="scrollToTop();setFocus()"
      >Confirm Subscription</button>
    </div>

    <!-- USED FOR TESTING (REMOVE LATER) -->
    <!-- <br />
    <br />
    <strong>Output:</strong>
    <pre>
      {{output}}
    </pre> -->
  </div>
</template>


<script>
import { MultiselectComponent, EventBus } from "vue-components-library";
import VueI18n from 'vue-i18n';
export default {
  props: {
    application: {
      type: Object
    },
    categories: {
      type: Array
    },
    demographics: {
      type: Array
    },
    regions: {
      type: Array
    },
    media_types: {
      type: Array
    },
    tags: {
      type: Array
    },
    language_type_id: {
      type: Number
    },
    manage: {
      type: Object
    },
    tabindex: {
      type: Number,
      required: false,
      default: -1
    }
  },
  components: {
    MultiselectComponent: MultiselectComponent
  },
  data() {
    return {
      application_id: this.application.id,
      styles_repo_url: process.env.MIX_STYLES_REPO,
      errors: [],
      addSubErrors: [],
      submitError: false,
      currentStep: 1,
      progress: this.progressbar(1),
      stepHidden: this.hiddenStep(1),
      totalSteps: 6,
      subscribeAll: false,
      update: false,
      subscriberId: null,
      output: "",
      subs: [],
      distributions: [],
      isHidden: true,
      form: {
        categories: [],
        email: null,
        emailConfirm: "",
        organization: "",
        firstName: "",
        lastName: "",
        phoneNumber: "",
        mediaTypes: [],
        tags: {},
        demographics: {},
        media: false,
        mediaRegion: []
      }
    };
  },
  mounted() {
    console.log(this.language_type_id); 
    // change other locale
    if (this.language_type_id == 4001) {
      this.$root.$i18n.locale = 'en'
    } else {
      this.$root.$i18n.locale = 'fr'
    }
    // setting default demographics
    for (var i = 0; i < this.demographics.length; i++) {
      Vue.set(this.form.demographics, this.demographics[i].name, []);
    }
    // adding multiple subscriptions from modal
    for (var i = 0; i < this.categories.length; i++) {
      const cat = this.categories[i].label;
      EventBus.$on(
        this.categories[i].label + "-change",
        payload => (this.form.tags[cat] = payload)
      );
    }
    // check if this is an edit of existing subscriber
    // pre-populate form data
    if (this.manage !== null) {
      // set update flag to true
      this.update = true;
      // set subscriber id 
      this.subscriberId = this.manage.subscriber.id;
      let firstSubId = Object.values(this.manage.subscriptions[0])[0][0];
      // populate customized subscriptions
      // currently shows subscriptions POST cartesian function
      // not sure how to un-cartesian
      for (var i = 0; i < this.manage.subscriptions.length; i++) {
        // if the first subscription id is 1, then the subscriber
        // is subscribed to All
        if (firstSubId === 1) {
          // set subscribe all to true
          this.subscribeAll = true;
          // clear subs
          this.subs = [];
        } else {
          // console.log(this.manage.subscriptions[i]); 
          this.form.tags = this.manage.subscriptions[i];
          this.formAddSubscription();
          // this.subscriptions(this.manage.subscriptions[i]);
          // this.subs.push(this.manage.subscriptions[i]);
        }
      }
      // populate email field
      this.form.email = this.manage.subscriber.email;
      this.form.emailConfirm = this.manage.subscriber.email;
      // populate media contact info
      this.form.firstName = this.manage.subscriber.first_name;
      this.form.lastName = this.manage.subscriber.last_name;
      this.form.organization = this.manage.subscriber.organization;
      this.form.phoneNumber = this.manage.subscriber.phone_number;
      if (this.manage.media_types.length > 0) {
        this.form.media = '1';
        let chosenMediaTypes = [];
        for (var i = 0; i < this.manage.media_types.length; i++) {
          chosenMediaTypes.push(this.manage.media_types[i]["id"]);
        }
        // set media types
        this.form.mediaTypes = chosenMediaTypes;
        // set region from demographics
        this.form.mediaRegion = this.manage.demographics.Region;
      } else {
        this.form.media = '0';
      }
      
      if (this.manage.demographics != null) {
        // populate demographics if not media
        this.form.demographics = this.manage.demographics;
      }
    }
  },
  methods: {
    scrollToTop() {
      window.scrollTo(0, 0);
    },
    setFocus() {
      // Focuses on REF in the html H2 element
      this.$refs.focus.focus(); // moves focus on REF element
    },
    checkForm: function() {
      this.errors = [];
      if (this.form.email) {
        if (this.form.emailConfirm !== this.form.email) {
          window.scrollTo(0, 0);
          this.errors.push("Emails do not match.");
        } else if (!this.validEmail(this.form.email)) {
          window.scrollTo(0, 0);
          this.errors.push("Valid email required.");
        } else if (!this.form.media) {
          window.scrollTo(0, 0);
          this.errors.push("Please identify yourself as a member of the public or the media.");
        } else {
          return true;
        }
      } else {
        window.scrollTo(0, 0);
        this.errors.push("Email required.");
      }
    },
    validEmail: function(email) {
      var re = /^(([^<>()[\]\\.,;:\s@"]+(\.[^<>()[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/;
      return re.test(email);
    },
    nextStep: function() {
      if (this.checkForm()) {
        //clear errors
        this.errors = [];
        // if user identified as NOT Media
        if (this.currentStep == 1 && this.audience == 3003) {
          this.currentStep = 3;
          this.progress = this.progressbar(this.currentStep);
          this.stepHidden = this.hiddenStep(this.currentStep);
        } else if (this.currentStep == 2) {
          // form validation for Media Contact Info
          // phone number format validation
          if (this.form.phoneNumber) {
            var re = /^\(?([0-9]{3})\)?[-. ]?([0-9]{3})[-. ]?([0-9]{4})$/;
            var validNumber = re.test(this.form.phoneNumber);
            if (!validNumber) {
              window.scrollTo(0, 0);
              this.errors.push("Please enter a valid phone number, ex. (555-555-5555).");
            } else {
              this.errors = [];
            }
          }
          if (!this.form.organization) {
            window.scrollTo(0, 0);
            this.errors.push("Organization required.");
          }
          if (this.form.mediaRegion.length == 0) {
            window.scrollTo(0, 0);
            this.errors.push("Region required.");
          }
          if (this.form.mediaTypes.length == 0) {
            window.scrollTo(0, 0);
            this.errors.push("Please select at least one <strong>Media Type</strong>.");
          }
          if (this.errors.length == 0) {
            this.currentStep = 4;
            this.progress = this.progressbar(this.currentStep);
            this.stepHidden = this.hiddenStep(this.currentStep);
          }
        }
        // logic for determining if user did NOT select any customizations
        else if (this.currentStep == 4 && this.subscribeAll == false) {
          if (this.subs.length === 0) {
            //setFocus();
            this.confirmSubscribeAll();
            window.scrollTo(0, 0);
          } else {
            this.currentStep++;
            this.progress = this.progressbar(this.currentStep);
            this.stepHidden = this.hiddenStep(this.currentStep);
          }
        } else {
          this.currentStep++;
          this.progress = this.progressbar(this.currentStep);
          this.stepHidden = this.hiddenStep(this.currentStep);
        }
      } else {
        return;
      }
    },
    backStep: function() {
      // if user identified as NOT Media
      if (this.currentStep == 3 && this.audience == 3003) {
        this.currentStep = 1;
        this.progress = this.progressbar(this.currentStep);
        this.stepHidden = this.hiddenStep(this.currentStep);
        // if user IS media and clicks back from step 4
      } else if (this.currentStep == 4 && this.audience == 3002) {
        this.currentStep = 2;
        this.progress = this.progressbar(this.currentStep);
        this.stepHidden = this.hiddenStep(this.currentStep);
      } else {
        this.currentStep--;
        this.progress = this.progressbar(this.currentStep);
        this.stepHidden = this.hiddenStep(this.currentStep);
      }
    },
    removeSubscription: function(index) {
      this.distributions.splice(index, 1);
      this.subs.splice(index, 1);
    },
    hiddenStep: function(step) {
      const spokenStep = [  "You are currently at step 1 of 6, Email&nbsp;Signup",
                            "You are currently at step 2 of 6, Subscriber Information",
                            "You are currently at step 2 of 6, Subscriber Information", // Not a mistake!
                            "You are currently at step 3 of 6, Customize Subscriptions",
                            "You are currently at step 4 of 6, Review Subscriptions",
                            "You are currently at step 5 of 6, Informed&nbsp;Consent",
                            "You are currently at step 6 of 6, Confirmation"];
      
      if (step <= spokenStep.length)
        return spokenStep[step-1];
      else
        return false;
    },
    progressbar: function(step) {
      const maxsteps = 6; // Define your number of steps
      var progressstep = this.hiddenStep(step).split(",")[1].trim();
      // Remove the Step 3 and adjust Steps above
      if (step > 2)
        step += -1;
      var ul = '<ul class="progressbar">';
      var width = (100 / maxsteps).toFixed(3);
      for (var i = 1; i < maxsteps + 1; i++) {
        if (i < step) {
          if (i > 2)
          {
            ul += '<li class="done" ' + 
                  'style="width:' + 
                  width + 
                  '%">' + 
                  this.hiddenStep(i+1).split(",")[1].trim().split(" ").join("<br \>") + 
                  "</li>";
          }
          else
          {
            ul += '<li class="done" ' + 
                  'style="width:' + 
                  width + 
                  '%">' + 
                  this.hiddenStep(i).split(",")[1].trim().split(" ").join("<br \>") +  
                  "</li>";
          }
        } 
        else if (i == step) 
        {
          ul += '<li class="current" ' +
                'style="width:' +
                width +
                '%">' +
                progressstep.split(" ").join("<br \>") +
                "<br \> (Current)</li>";
        } 
        else 
        {
          if (i > 2)
          {
            ul += '<li style="width:' + 
                  width + 
                  '%">' + 
                  this.hiddenStep(i+1).split(",")[1].trim().split(" ").join("<br \>") + 
                  "</li>";
          }
          else
          {
            ul += '<li style="width:' + 
                  width + 
                  '%">' + 
                  this.hiddenStep(i).trim().split(",")[1].trim().split(" ").join("<br \>") + 
                  "</li>";
          }
        }
      }
      ul.concat("</ul>");
      return ul;
    },
    confirmSubscribeAll() {
      this.setFocus();
      this.$bvModal
        .msgBoxConfirm("Are you sure you would like to subscribe to all news?", {
          title: "Please Confirm",
          size: "sm",
          buttonSize: "sm",
          okVariant: "danger",
          okTitle: "YES",
          cancelTitle: "NO",
          hideHeaderClose: false,
          centered: true
        })
        .then(value => {
          if (value == true) {
            this.subscribeAll = true;
            this.currentStep++;
            this.progress = this.progressbar(this.currentStep);
            this.stepHidden = this.hiddenStep(this.currentStep);
            this.setFocus();
            setFocus();
            this.window.scrollTo(0, 0);
            this.window.scrollTo(0, 0);
            this.setFocus();
          }
        })
        .catch(err => {
          // An error occurred
        });
        //window.scrollTo(0, 0);
        setFocus();
    },
    formSubmit: function() {
      // if subscribe all is selected send tag id 1
      this.progress = this.progressbar(this.currentStep+1);
      if (this.subscribeAll == true) {
        this.subs = [1];
      }
      let self = this;
      axios
        .post("/subscribe", {
          application_id: this.application_id,
          subscriber_id: this.subscriberId,
          language_type_id: this.language_type_id,
          email: this.form.email,
          organization: this.form.organization,
          firstName: this.form.firstName,
          lastName: this.form.lastName,
          phoneNumber: this.form.phoneNumber,
          mediaTypes: this.form.mediaTypes,
          mediaRegion: this.form.mediaRegion,
          media: this.form.media,
          subs: this.subs,
          audience: this.audience,
          demographics: this.form.demographics,
          update: this.update
        })
        .then(function(response) {
          self.output = response.data;
          self.currentStep++;
          //self.progress = self.progressbar(self.currentStep);
        })
        .catch(function(error) {
          self.submitError = true; 
          self.currentStep++;
          //self.progress = self.progressbar(self.currentStep);
        });
    },
    formAddSubscription: function() {
      if (this.categories.length !== Object.keys(this.form.tags).length) {
        //this.addSubErrors = [];
        //this.addSubErrors.push("Please fill all options");
        this.errors = [];
        this.errors.push("Please fill in all options.");
        this.isHidden = false;
        window.scrollTo(0, 0); // Should scroll to where the Error message is...
      } else {
        this.subs.push(this.form.tags);
        this.subscriptions(this.form.tags);
        this.form.tags = {};
        this.$bvModal.hide("add-subscription");
        this.subscribeAll = false;
        this.addSubErrors = [];
        this.isHidden = true;
      }
    },
    subscriptions: function(tags) {
      var result = [];
      // set categories for 'customize subscriptions' summary 
      // using Object.keys sometime rearranges the order of tags, why???!!!
      // adding sort() fixes, but may need a better solution
      this.form.categories = Object.keys(tags).sort();
      for (const [key, value] of Object.entries(tags).sort()) {
        const tagsList = this.tags;
        var tag = [];
        value.forEach(function(element) {
          tag.push(tagsList.find(item => item.value == element));
        });
        result.push(tag);
      }
      this.distributions.push(result);
    },
    getDemoName: function(id) {
      const demographicsList = this.demographics;
      var name = [];
      for (var i = 0; i < demographicsList.length; i++) {
        let children = demographicsList[i].children;
        for (var j = 0; j < children.length; j++) {
          if (typeof id === "object") {
            id.forEach(function(item) {
              if (children[j].id == item) {
                name.push(children[j].name);
              }
            });
          } else if (children[j].id == id) {
            name.push(children[j].name);
          }
        }
      }
      return name;
    }
  },
  computed: {
    audience: function() {
      if (this.form.media == 0) {
        return 3003;
      } else {
        return 3002;
      }
    },
    lang: function() {
      if (this.language_type_id == 4001) {
        return "_en";
      } else {
        return "_fr";
      }
    },
    selectedMediaTypes: function() {
      const media_types = this.media_types;
      const type_ids = this.form.mediaTypes;
      var types = [];
      type_ids.forEach(function(element) {
        types.push(media_types.find(item => item.id == element));
      });
      return types;
    }
  }
};
</script>
