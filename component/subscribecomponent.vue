<template>
  <div>
    <!-- FAQ -->
    <div style="float:right;">
      <a href='#' @click.prevent="$bvModal.show('show-faq')">FAQ</a>
    </div>
    <!-- Modal for FAQ -->
    <b-modal id="show-faq" size="lg" scrollabel centered hide-footer>
      <template #modal-title>FAQ</template>
      <h3>Subscription FAQ</h3>
      <div v-for="faq in application.faq" :key="faq.Question">
        <p>
          <strong>Question:</strong>
          {{ faq.Question }}
          <br />
          <strong>Answer:</strong>
          {{ faq.Answer }}
        </p>
      </div>
    </b-modal>

    <h1>{{ application.name_en }}</h1>
    <h2>Current step: {{ currentStep }}</h2>
    <div id="steps" v-html="progress"></div>

    <!-- ERROR MESSAGE  -->
    <p v-if="errors.length">
    <b>Please correct the following error(s):</b>
      <ul style="color:red">
        <li v-for="(error,index) in errors" :key="index">{{ error }}</li>
      </ul>
    </p>


    <!-- STEP 1 -->
    <section v-if="currentStep == 1">
      <hr />
      
      <label>Email</label>
      <input v-model="form.email" type="email" class="ontario-input form-control" />
      <br />
      <label>Confirm Email</label>
      <input v-model="form.emailConfirm" type="email" class="ontario-input form-control" />
      <br />
      <p>Are you member of the media?</p>
      <input type="radio" v-bind:value="true" v-model="form.media" />
      <label for="media">Yes</label>
      <br />
      <input type="radio" v-bind:value="false" v-model="form.media" />
      <label for="media">No</label>
      <hr />
    </section>


    <!-- STEP 2 -->
    <!-- only shows when media is selected in step 1 -->
    <section v-if="currentStep == 2">
      <hr />
      <h3>Media Contact Information</h3>
      <label>Organization</label>
      <input v-model="form.organization" type="text" class="ontario-input form-control" />

      <label>Region</label>
      <fieldset class="ontario-fieldset">
        <select class="ontario-input form-control" v-model="form.mediaRegion">
          <option disabled value="">Please select one</option>
          <option
          v-for="region in regions"
          :key="region.value"
          :value="region.value"
          >{{region.label}}</option>
      
        </select>
      </fieldset>
      <label>First Name</label>
      <input v-model="form.firstName" type="text" class="ontario-input form-control" />

      <label>Last Name</label>
      <input v-model="form.lastName" type="text" class="ontario-input form-control" />

      <label>Phone Number</label>
      <input v-model="form.phoneNumber" type="text" class="ontario-input form-control" />
      
      <label>Media Type</label>
      <fieldset class="ontario-fieldset">
        <div class="ontario-checkboxes mt-3">
          <div 
          v-for="type in media_types"
          :key="type.id"
          class="ontario-checkboxes__item">
            <input
              v-model="form.mediaTypes"
              type="checkbox"
              v-bind:value="type.id"
              :true-value="type.id"
              false-value="no"
              class="ontario-checkboxes__input"
            />
            <label class="ontario-checkboxes__label">{{ type.name }}</label>
          </div>
        </div>
      </fieldset>

      <hr />
    </section>
    
    <!-- STEP 3 -->
    <!-- only shows if media is NOT selected in step 1 -->
    <section v-if="currentStep == 3">
      <hr />
      <div class="ontario-form-group">
        <h3>Subscriber Profile (optional)</h3>

        <div v-for="demographic in demographics" :key="demographic.id">
          
          <h4>{{ demographic.name_en }}</h4>

            <fieldset v-if="demographic.input_type == 'checkbox'" class="ontario-fieldset">
              <div class="ontario-checkboxes mt-3">
                <div 
                v-for="child in demographic.children"
                :key="child.id"
                class="ontario-checkboxes__item">
                  <input
                    v-model="form.demographics[demographic.name_en]"
                    :type="child.input_type"
                    v-bind:value="child.id"
                    :true-value="child.id"
                    false-value="no"
                    class="ontario-checkboxes__input"
                  />
                  <label class="ontario-checkboxes__label">{{ child.name_en }}</label>
                </div>
              </div>
            </fieldset>
         
            <fieldset v-else-if="demographic.input_type == 'radio'" class="ontario-fieldset">
              <div class="ontario-radioboxes mt-3">
                <div 
                v-for="child in demographic.children"
                :key="child.id"
                class="ontario-radioboxes__item">
                  <input
                    v-model="form.demographics[demographic.name_en]"
                    :type="child.input_type"
                    :value="child.id"
                    class="ontario-radioboxes__input"
                  />
                  <label class="ontario-radioboxes__label">{{ child.name_en }}</label>
                </div>
              </div>
            </fieldset>

            <fieldset v-else-if="demographic.input_type == 'select'" class="ontario-fieldset">
              <select class="ontario-input form-control" v-model="form.demographics[demographic.name_en]">
                <option disabled value="">Please select one</option>
                <option
                v-for="child in demographic.children"
                :key="child.id"
                :value="child.id"
                >{{child.name_en}}</option>
           
              </select>
            </fieldset>

        </div>
      </div>
      <hr />
    </section>

    <!-- STEP 4 -->
    <section v-if="currentStep == 4">
      <hr />
      <button
        class="ontario-button ontario-button--primary mt-0 text-white"
        @click.prevent="$bvModal.show('add-subscription')"
      >+ Add Subscription</button>
      <br />
      <br />

      <div v-if="subscribeAll == true">
        <p>You are subscribed to All News</p>
      </div>

      <div>
        <b-table-simple hover small responsive>
      
          <b-thead head-variant="dark">
            <b-tr>
              <b-th v-for="(category,index) in categories" :key="index">{{category.label}}</b-th>
              <b-th>Actions</b-th>
            </b-tr>
          </b-thead>

          <b-tbody>
            <b-tr v-for="(distribution,index) in distributions" :key="index">
              <b-td v-for="(category,index) in distribution" :key="index">
                <span v-for="(tag,index) in category" :key="index">
                  {{tag.label}}
                  <br />
                </span>
              </b-td>
              <b-td><img @click.prevent="removeSubscription(index)" src="https://dev-styles-dcpp.ontarionewsroom.com/assets/ontario-icon-delete-blue.png" alt="Delete" style="width:25px; height:25px;"></b-td>
            </b-tr>
          </b-tbody>
          
        </b-table-simple>
      </div>

      <!-- MODAL for adding a subscription -->
      <b-modal id="add-subscription" size="lg" no-close-on-backdrop centered hide-footer>
        <template #modal-title>Add Subscription</template>
        <span v-for="category in categories" :key="category.value">
          <MultiselectComponent
            :label="category.label"
            :name="category.label"
            :options="category.tags"
            searchable
            selectPlaceholder="select option"
          />
        </span>
        <button
          class="ontario-button ontario-button--primary mt-0 text-white"
          @click="formAddSubscription"
        >Confirm</button>
      </b-modal>

      <hr />
    </section>

    <!-- STEP 5 -->
    <section v-if="currentStep == 5">
      <hr />
      <h3>Confirmation</h3>
      <div>
        <h4>Email for Subscription</h4>
        <ul>
          <li>Email: {{ form.email }}</li>
          <li v-if="form.media">Media: Yes</li>
          <li v-else>Media: No</li> 
        </ul>
      </div>

      <div v-if="form.media">
        <h4>Media Contact Information</h4>
        <ul>
          <li>Organization: {{ form.organization }}</li>
          <li>Region: <span v-for="(name,index) in getDemoName(form.mediaRegion)" :key="index"><div style="display:inline-block;">{{ name }}</div></span></li>
          <li>First Name: {{ form.firstName }}</li>
          <li>Last Name: {{ form.lastName }}</li>
          <li>Phone Number: {{ form.phoneNumber }}</li>
          <li>Media type: <span v-for="type in selectedMediaTypes" :key="type.id">{{ type.name }}, </span></li>
        </ul>
      </div>
      <div v-else>
        <h4>Subscriber Profile</h4>
        <ul>
          <li v-for="demo in demographics" :key="demo.id">{{ demo.name_en }}: <span v-for="(name,index) in getDemoName(form.demographics[demo.name_en])" :key="index"><div style="display:inline-block;">{{ name }}</div></span></li>
        </ul>
      </div>


      <div v-if="subscribeAll">
        <h4>You are subscribed to All News</h4>
      </div>
      <div v-else>
        <b-table-simple small responsive bordered striped outlined>
      
          <b-thead>
            <b-tr>
              <b-th v-for="(category,index) in categories" :key="index">{{category.label}}</b-th>
            </b-tr>
          </b-thead>

          <b-tbody>
            <b-tr v-for="(distribution,index) in distributions" :key="index">
              <b-td v-for="(category,index) in distribution" :key="index">
                <span v-for="(tag,index) in category" :key="index">
                  {{tag.label}}
                  <br />
                </span>
              </b-td>
            </b-tr>
          </b-tbody>
          
        </b-table-simple>
      </div>

      <hr />
    </section>

    <!-- STEP 6 -->
    <section v-if="currentStep == 6">
      <hr />
      <h3>Informed Consent</h3>
      <p><span v-html="application.noc"></span></p>
      <hr />
    </section>

    <button
      class="ontario-button ontario-button--primary mt-0 text-white"
      v-if="currentStep != 1"
      @click.prevent="backStep"
      @click="scrollToTop()"
    >Back</button>

    <button
      class="ontario-button ontario-button--primary mt-0 text-white"
      v-if="currentStep != totalSteps"
      @click.prevent="nextStep"
      @click="scrollToTop()"
    >Next</button>

    <button
      class="ontario-button ontario-button--primary mt-0 text-white"
      v-if="currentStep == totalSteps"
      @click.prevent="formSubmit"
    >Submit</button>
    <br />
    <br />
    <br />
    <br />
    <strong>Output:</strong>
    <pre>
        {{output}}
    </pre>
  </div>
</template>

<script>
import { MultiselectComponent, EventBus } from "vue-components-library";

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
    }
  },
  components: {
    MultiselectComponent: MultiselectComponent
  },
  data() {
    return {
      application_id: this.application.id,
      errors: [],
      currentStep: 1,
      progress: this.progressbar(1),
      totalSteps: 6,
      subscribeAll: false,
      output: "",
      subs: [],
      distributions: [],
      form: {
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
        mediaRegion: [],
      },
    };
  },
  mounted() {
    // setting default demographics 
    for (var i = 0; i < this.demographics.length; i++) {
      Vue.set(this.form.demographics, this.demographics[i].name_en,[])
    }

    // adding multiple subscriptions from modal
    for (var i = 0; i < this.categories.length; i++) {
      const cat = this.categories[i].label;
      EventBus.$on(
        this.categories[i].label + "-change",
        payload => (this.form.tags[cat] = payload)
      );
    }
  },
  methods: {
    scrollToTop() {
        window.scrollTo(0,0);
    },
    checkForm: function () {
      if (this.form.email) {
        if (this.form.emailConfirm !== this.form.email) {
          this.errors.push('Emails do not match.');
        } else {
          return true;
        }
      }

      if (!this.form.email) {
        this.errors.push('Email required.');
      }

    },
    validEmail: function (email) {
      var re = /^(([^<>()[\]\\.,;:\s@"]+(\.[^<>()[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/;
      return re.test(email);
    },
    nextStep: function() {
      if(this.checkForm()) {
        //clear errors
        this.errors = [];

        // if user identified as NOT Media 
        if (this.currentStep == 1 && this.audience == 3003) {
          this.currentStep = 3;
          this.progress = this.progressbar(this.currentStep);
        }
        else if (this.currentStep == 2) {
          this.currentStep = 4; 
          this.progress = this.progressbar(this.currentStep);
        }
        // logic for determining if user did NOT select any customizations
        else if (this.currentStep == 4 && this.subscribeAll == false) {
          if (this.subs.length === 0) {
            this.confirmSubscribeAll();
          } else {
            this.currentStep++;
            this.progress = this.progressbar(this.currentStep);
          }
        } else {
          this.currentStep++;
          this.progress = this.progressbar(this.currentStep);
        }
      } else {
        return
      }
    },
    backStep: function() {
      //this.progress = this.progressbar(this.currentStep - 1);
      // if user identified as NOT Media 
      if (this.currentStep == 3 && this.audience == 3003) {
        this.currentStep = 1;
        this.progress = this.progressbar(this.currentStep);
      // if user IS media and clicks back from step 4
      } else if (this.currentStep == 4 && this.audience == 3002) {
        this.currentStep = 2;
        this.progress = this.progressbar(this.currentStep);
      } else {
        this.currentStep--;
        this.progress = this.progressbar(this.currentStep);
      }
    },
    removeSubscription: function(index) {
      this.distributions.splice(index, 1); 
      this.subs.splice(index,1);
    },
    demographicChange: function(e) {
      console.log(e)
    },
    progressbar: function(step) {
      const maxsteps = 6; // Define your number of steps
      var ul = '<ul class="progressbar">';
      var width = (100 / maxsteps).toFixed(3);
      for(var i = 1; i < maxsteps + 1; i++) {
          if (i < step){
            ul += '<li class="done" ' + 'style="width:' + width + '%">' + 'Step ' + i + '</li>';
          } else if (i == step) {
            ul += '<li class="current" ' + 'style="width:' + width + '%">' + 'Step ' + i + ' (Current)</li>';
          } else {
            ul += '<li style="width:' + width + '%">' + 'Step ' + i + '</li>';
          }
      }
      ul.concat('</ul>');
      return ul;

    },
    confirmSubscribeAll() {
      this.$bvModal
        .msgBoxConfirm("Are you sure you would like to subscribe to all news", {
          title: "Please Confirm",
          size: "sm",
          buttonSize: "sm",
          okVariant: "danger",
          okTitle: "YES",
          cancelTitle: "NO",
          footerClass: "p-2",
          hideHeaderClose: false,
          centered: true
        })
        .then(value => {
          if (value == true) {
            this.subscribeAll = true;
            this.currentStep++;
            this.progress = this.progressbar(this.currentStep);
          }
        })
        .catch(err => {
          // An error occurred
        });
    },
    formSubmit: function() {
      // if subscribe all is selected send tag id 1
      if (this.subscribeAll == true) {
        this.subs = [1];
      }
      
      let self = this;
      axios
        .post("/subscribe", {
          application_id: this.application_id,
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
          demographics: this.form.demographics
        })
        .then(function(response) {
          self.output = response.data;
        })
        .catch(function(error) {
          self.output = response.error;
        });
    },
    formAddSubscription: function() {
      if (this.categories.length !== Object.keys(this.form.tags).length) {
        alert('fill out everything please'); 
      } else {
        this.subs.push(this.form.tags);
        this.subscriptions(this.form.tags);
        this.form.tags = {};
        this.$bvModal.hide("add-subscription");
        this.subscribeAll = false;
      }
    },
    subscriptions: function(tags) {
      
      var result = [];

      for (const [key, value] of Object.entries(tags)) {
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
      // console.log(typeof id)
      const demographicsList = this.demographics;
      // var region_id = this.form.demographics.Region;
      var name = [];
      
      for (var i = 0; i < demographicsList.length; i++) {
        let children = demographicsList[i].children
        for (var j = 0; j < children.length; j++) {
          // console.log(children[j].name_en)
          if (typeof id === 'object') {
            id.forEach(function(item) {
              if (children[j].id == item) {
                name.push(children[j].name_en)
              }
            })
          }
          else if (children[j].id == id) {
            name.push(children[j].name_en)
          }
        }
      }

      return name; 

    },
  },
  watch: {
    // 'form.demographics': function (newValue, oldValue) {
    //   console.log(newValue); 
    // }
  },
  computed: {
    audience: function() {
      if (this.form.media == false) {
        return 3003;
      } else {
        return 3002;
      }
    },
    selectedMediaTypes: function() {
      const media_types = this.media_types
      const type_ids = this.form.mediaTypes
      var types = [];

      type_ids.forEach(function(element) {
        types.push(media_types.find(item => item.id == element));
      });

      return types; 
    },
  }
};
</script>
