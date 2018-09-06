<template>
  <div>
    <div v-if="wizardInProgress" v-show="asyncState !== 'pending'">
      <keep-alive>
        <component
          ref="currentStep"
          :is="currentStep"
          @updateAsyncState="updateAsyncState"
          :wizard-data="form"
        ></component>
      </keep-alive>

      <div class="progress-bar">
        <div :style="`width: ${progress}%;`"></div>
      </div>

      <!-- Actions -->
      <div class="buttons">
        <button
          @click="goBack"
          v-if="currentStepNumber > 1"
          class="btn-outlined"
        >Back
        </button>

        <button
          @click="nextButtonAction"
          class="btn"
        >{{isLastStep ? 'Complete Order' : 'Next'}}</button>
      </div>
    </div>

    <div v-else>
      <h1 class="title">Thank you!</h1>
      <h2 class="subtitle">
        We look forward to shipping you your first box!
      </h2>

      <p class="text-center">
        <a href="https://vueschool.io" target="_blank" class="btn">Go somewhere cool!</a>
      </p>
    </div>

    <div class="loading-wrapper" v-if="asyncState === 'pending'">
      <div class="loader">
        <img src="/spinner.svg" alt="">
        <p>Please wait, we're hitting our servers!</p>
      </div>
    </div>
  </div>
</template>

<script>
import {postFormToDB} from '../api'
import FormPlanPicker from './FormPlanPicker'
import FormUserDetails from './FormUserDetails'
import FormAddress from './FormAddress'
import FormReviewOrder from './FormReviewOrder'
export default {
  name: 'FormWizard',
  components: {
    FormPlanPicker,
    FormUserDetails,
    FormAddress,
    FormReviewOrder
  },
  data () {
    return {
      currentStepNumber: 1,
      asyncState: null,
      steps: [
        'FormPlanPicker',
        'FormUserDetails',
        'FormAddress',
        'FormReviewOrder'
      ],
      form: {
        plan: null,
        email: null,
        name: null,
        password: null,
        address: null,
        recipient: null,
        chocolate: false,
        otherTreat: false
      }
    }
  },
  computed: {
    isLastStep () {
      return this.currentStepNumber === this.length
    },
    wizardInProgress () {
      return this.currentStepNumber <= this.length
    },
    length () {
      return this.steps.length
    },
    currentStep () {
      return this.steps[this.currentStepNumber - 1]
    },
    progress () {
      return this.currentStepNumber/this.length * 100
    }
  },
  methods: {
    updateAsyncState (state) {
      this.asyncState = state
    },
    submitOrder () {
      this.asyncState = 'pending'
      postFormToDB(this.form)
        .then(() => {
          console.log('form submitted', this.form)
          this.asyncState = 'success'
          this.currentStepNumber++
        })
    },
    nextButtonAction () {
      this.$refs.currentStep.submit()
        .then(stepData => {
          Object.assign(this.form, stepData)
          if (this.isLastStep) {
            this.submitOrder()
          } else {
            this.goNext()
          }
        })
        .catch(error => console.log(error))
    },
    goBack () {
      this.currentStepNumber--
    },
    goNext () {
      this.currentStepNumber++
    }
  }
}
</script>
