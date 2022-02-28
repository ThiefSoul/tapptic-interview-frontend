<template>
  <form class="row" @submit="onSubmit">
    <div class="col-12 mb-3">
      <label for="email" class="form-label">E-mail*</label>
      <input type="text" class="form-control" id="email" name="email" :class="{'is-invalid': hasFieldError('email')}" v-model="email" @blur="onFieldBlur('email')"/>
      <div class="invalid-feedback">
        {{ fieldServerErrorMessage('email') }}
      </div>
      <div class="invalid-feedback" v-for="error of v$.email.$errors" :key="error.$uid">
        {{ error.$message }}
      </div>
    </div>

    <div class="col-12 mb-3">
      <label for="passwordField" class="form-label">Password*</label>
      <input type="password" class="form-control" id="passwordField" name="password" :class="{'is-invalid': hasFieldError('password')}" v-model="password"  @blur="onFieldBlur('password')"/>
      <div class="invalid-feedback">
        {{ fieldServerErrorMessage('password') }}
      </div>
      <div class="invalid-feedback" v-for="error of v$.password.$errors" :key="error.$uid">
        {{ error.$message }}
      </div>
    </div>

    <div class="col-12 mb-3">
      <label for="descriptionField" class="form-label">Description</label>
      <textarea class="form-control" id="descriptionField" name="description" :class="{'is-invalid': hasFieldError('description')}" v-model="description"  @blur="onFieldBlur('description')"></textarea>
      <div class="invalid-feedback">
        {{ fieldServerErrorMessage('description') }}
      </div>
      <div class="invalid-feedback" v-for="error of v$.description.$errors" :key="error.$uid">
        {{ error.$message }}
      </div>
      <div>{{description.length}} / 50</div>
    </div>

    <div class="col-12 mb-3">
      <button class="btn btn-primary" type="submit" :disabled="isSubmitDisabled">Submit form</button>
    </div>

    <div class="col-12">
      <div v-if="generalError" class="alert alert-danger">
        {{ generalError }}
      </div>
    </div>
  </form>
</template>

<script>
import useVuelidate from '@vuelidate/core'
import { required, email, maxLength, minLength, helpers } from '@vuelidate/validators'

const hasLowercase = value => /^(.*[a-z].*)$/.test(value)
const hasUppercase = value => /^(.*[A-Z].*)$/.test(value)
const hasNumber = value => /^(.*[0-9].*)$/.test(value)

const API_URL = 'https://tapptic-interview.free.beeceptor.com'
// const API_URL = 'https://tapptic-interview-general-error.free.beeceptor.com'

const tryJsonParse = text => {
  try {
    return JSON.parse(text)
  } catch {
    return null
  }
}

export default {
  setup() {
    return { v$: useVuelidate() }
  },
  data() {
    return {
      email: '',
      password: '',
      description: '',

      isSubmitDisabled: false,
      serverErrors: null,
      generalError: null
    }
  },
  validations () {
    return {
      email: { required, email },
      password: {
        required,
        minLength: minLength(6),
        hasUppercase: helpers.withMessage('Value must contain at least one uppercase', hasUppercase),
        hasLowercase: helpers.withMessage('Value must contain at least one lowercase', hasLowercase),
        hasNumber: helpers.withMessage('Value must contain at least one digit', hasNumber),
      },
      description: { maxLength: maxLength(50) }
    }
  },
  methods: {
    hasFieldError(field) {
      return this.v$[field].$error || !!this.serverErrors?.[field]
    },
    fieldServerErrorMessage(field) {
      return this.serverErrors?.[field]
    },
    onFieldBlur(field) {
      this.v$[field].$touch()
      delete(this.serverErrors?.[field])
    },

    async onSubmit(e) {
      e.preventDefault()

      this.isSubmitDisabled = true

      await this.submit()

      this.isSubmitDisabled = false
    },
    async submit () {
      this.serverErrors = null
      this.generalError = null

      try {
        const response = await this.callRandomEndpoint()
        await this.handleResponse(response)
      } catch (error) {
        this.generalError = error.message
      }
    },
    async handleResponse (response) {
      if (response.status === 200) return

      const text = await response.text()
      const json = tryJsonParse(text)

      if (json?.errors) {
        this.serverErrors = json.errors
      } else {
        this.generalError = text
      }
    },
    callRandomEndpoint() {
      const endpoint = `${API_URL}/${this.getRandomEndpoint()}`
      return fetch(endpoint, {
        method: 'POST',
        body: JSON.stringify({
          email: this.email,
          password: this.password,
          description: this.description,
        })
      })
    },
    getRandomEndpoint() {
      const fields = ['email', 'password', 'description', 'no-error']
      return fields[Math.round(Math.random() * (fields.length - 1))]
    }
  }
}
</script>
