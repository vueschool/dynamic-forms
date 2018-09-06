<template>
  <div>
    <h1 class="title">Create Account</h1>

    <h2 class="subtitle">
      Create an account or log in to order your liquid gold subscription
    </h2>

    <form v-if="!loggedIn" class="form">
      <div class="form-group">
        <label class="form-label" for="email">Email</label>
        <input type="text" v-model="$v.form.email.$model" :disabled="emailCheckedInDB" placeholder="your@email.com" class="form-control" id="email">
        <div v-if="emailCheckedInDB" class="error info">
          <a href="#" @click="reset">Not you?</a>
        </div>
        <div v-if="$v.form.email.$error && !$v.form.email.required" class="error">email is required</div>
        <div v-if="$v.form.email.$error && !$v.form.email.email" class="error">email is invalid</div>
      </div>


      <div v-if="emailCheckedInDB" class="form-group">
        <label class="form-label" for="password">Password</label>
        <input v-model="$v.form.password.$model" type="password" placeholder="Super Secret Password" class="form-control" id="password">
        <div v-if="$v.form.password.$error && !$v.form.password.required" class="error">password is required</div>
        <div v-if="$v.form.password.$error && !$v.form.password.correct" class="error">password is invalid - try again</div>
      </div>

      <div v-if="emailCheckedInDB && !existingUser" class="form-group">
        <label class="form-label" for="name">Name</label>
        <input v-model="$v.form.name.$model" type="text" placeholder="What should we call you?" class="form-control" id="name">
        <div v-if="$v.form.name.$error" class="error">name is required</div>
      </div>
    </form>
    <div v-else class="text-center">
      Successfully logged in! <a href="#" @click="reset">Not {{form.name}}?</a>
    </div>
  </div>
</template>

<script>
  import {authenticateUser, checkIfUserExistsInDB} from "../api";
  import {required, email} from 'vuelidate/lib/validators'
  export default {
    data () {
      return {
        form: {
          email: null,
          password: null,
          name: null,
        },
        emailCheckedInDB: false,
        existingUser: false,
        wrongPassword: false
      }
    },
    computed: {
      loggedIn () {
        return this.existingUser && this.form.name
      }
    },
    validations: {
      form: {
        email: {
          required,
          email
        },
        password: {
          required,
          correct () {
            return !this.wrongPassword
          }
        },
        name: {
          required
        }
      }
    },
    methods: {
      checkIfUserExists () {
        if (!this.$v.form.email.$invalid) {
          this.$emit('updateAsyncState', 'pending')
          return checkIfUserExistsInDB(this.form.email)
            .then(() => {
              // USER EXISTS
              this.existingUser = true
              this.emailCheckedInDB = true
              this.$emit('updateAsyncState', 'success')
            })
            .catch(() => {
              // USER DOESN'T EXIST
              this.existingUser = false
              this.emailCheckedInDB = true
              this.$emit('updateAsyncState', 'success')
            })
        } else {
          return Promise.reject('email is invalid')
        }
      },
      login () {
        this.wrongPassword = false
        if (!this.$v.form.password.$invalid) {
          this.$emit('updateAsyncState', 'pending')
          return authenticateUser(this.form.email, this.form.password)
            .then(user => {
              // LOGGED IN
              this.form.name = user.name
              this.$emit('updateAsyncState', 'success')
            })
            .catch(() => {
              // WRONG PASSWORD?
              this.wrongPassword = true
              this.$emit('updateAsyncState', 'success')
            })
        } else {
          return Promise.reject('password is invalid')
        }
      },
      reset () {
        this.form.email = null
        this.form.password = null
        this.form.name = null
        this.emailCheckedInDB = false
        this.wrongPassword = false
        this.existingUser = false
        this.$v.$reset()
      },
      submit () {
        let job
        if (!this.emailCheckedInDB) {
          this.$v.form.email.$touch()
          job = this.checkIfUserExists()
        } else {
          if (this.existingUser && !this.loggedIn) {
            this.$v.form.password.$touch()
            job = this.login()
          } else {
            // new user is typing info manually
            this.$v.$touch()
            job = Promise.resolve()
          }
        }

        return new Promise((resolve, reject) => {
          job.then(() => {
            if (!this.$v.$invalid) {
              resolve({
                email: this.form.email,
                password: this.form.password,
                name: this.form.name
              })
            } else {
              reject('data is not valid yet')
            }
          })
            .catch(error => reject(error))
        })

        // this.$emit('update', {
        //   data: {
        //     email: this.form.email,
        //     password: this.form.password,
        //     name: this.form.name
        //   },
        //   valid: !this.$v.$invalid
        // })
      }
    }
  }
</script>

<style scoped>

</style>