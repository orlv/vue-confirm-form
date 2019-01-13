<template id="vue-confirm-form-template">
    <div>
        <span class="text-button p-0" :class="{'text-button-green': !busy }" @click="click">{{ buttonText }}</span>
        <div v-if="busy" class="vue-confirm-form" @click="cancel">
            <div class="vue-confirm-form-main" @click.stop>
                <div class="bold center mb-1 font-1_3">{{ title }}</div>
                <div class="vue-confirm-error-message">{{ message }}</div>
                <table>
                    <tbody>
                    <tr v-for="(fieldValue, fieldName) in formFields">
                        <td>{{ fieldName }}</td>
                        <td v-if="disabledFields[fieldName]">{{ form[fieldName] }}</td>
                        <td v-else-if="typeof fieldValue === 'string'">
                            <input v-model="form[fieldName]" type="text" class="" :title="fieldName"/>
                        </td>
                        <td v-else-if="typeof fieldValue === 'number'">
                            <input v-model="form[fieldName]" type="number" class="" :title="fieldName"/>
                        </td>
                        <td v-else-if="Array.isArray(fieldValue)">
                            <div v-if="typeof fieldValue[0] === 'object'">
                                <div v-for="(checkbox, n) in fieldValue">
                                    <input :id="`vue-confirm-form-${fieldName}-${n}`" v-model="form[fieldName]" title=""
                                           class="css-checkbox" type="checkbox"
                                           :value="Object.values(checkbox)[0]">
                                    <label :for="`vue-confirm-form-${fieldName}-${n}`" class="css-label">
                                        {{ Object.keys(checkbox)[0] }}</label>
                                </div>
                            </div>
                            <select v-else v-model="form[fieldName]" :title="fieldName">
                                <option v-for="option in formFields[fieldName]">{{ option }}</option>
                            </select>
                        </td>
                        <td v-else-if="typeof fieldValue === 'boolean'">
                            <input :id="`vue-confirm-form-${fieldName}`" v-model="form[fieldName]" title=""
                                   class="css-checkbox" type="checkbox">
                            <label :for="`vue-confirm-form-${fieldName}`" class="css-label"></label>
                        </td>
                    </tr>
                    <tr>
                        <td colspan="2" class="center p-1 font-1_3">
                            <span class="text-button text-button-green" @click="callConfirm">{{ confirmText }}</span>
                        </td>
                    </tr>
                    </tbody>
                </table>
            </div>
        </div>
    </div>
</template>

<script>
import 'rlv-styles/styles.css'

export default {
  props: ['callback', 'title', 'text', 'confirm', 'fields', 'default', 'disabled'],

  data: function () {
    return {
      busy: false,
      formFields: {},
      form: {},
      message: ''
    }
  },

  computed: {
    disabledFields: function () {
      return typeof this.disabled === 'object' ? this.disabled : {}
    },

    buttonText: function () {
      return this.busy ? 'Cancel' : this.text
    },

    confirmText: function () {
      return this.confirm ? this.confirm : 'OK'
    }
  },

  watch: {
    fields: function () {
      this.extractDefaultForm()
    }
  },

  created () {
    this.extractDefaultForm()
  },

  methods: {
    extractDefaultForm: function () {
      this.formFields = Object.assign({}, this.fields) || {}

      this.form = Object.keys(this.formFields).reduce((acc, field) => {
        const val = this.formFields[field]

        if (Array.isArray(val)) {
          // Checkbox or select
          acc[field] = typeof val[0] === 'object' ? [] : val[0]
        } else {
          acc[field] = val
        }

        return acc
      }, {})

      if (typeof this.default === 'object') {
        Object.keys(this.default).forEach(key => {
          if (key in this.form) {
            this.form[key] = Array.isArray(this.default[key]) ? this.default[key].slice() : this.default[key]
          }
        })
      }
    },

    callConfirm: async function () {
      this.busy = false
      this.message = ''

      Object.keys(this.fields).forEach(field => {
        if ((typeof this.fields[field] === 'number' || (Array.isArray(this.fields[field]) && typeof this.fields[field][0] === 'number')) && typeof this.form[field] === 'string') {
          const val = parseFloat(this.form[field])

          this.form[field] = isNaN(val) ? '' : val
        }
      })

      const res = await this.callback(Object.assign({}, this.form))

      if (typeof res === 'string' && res.length > 0) {
        this.busy = true
        this.message = res
      }
    },

    cancel: function () {
      this.busy = false
      this.extractDefaultForm()
    },

    click: function () {
      if (this.busy) {
        this.cancel()
      } else {
        this.busy = true
      }
    }
  },

  template: '#vue-confirm-form-template'
}
</script>

<style scoped>
    .vue-confirm-form {
        display: flex;
        justify-content: center;
        align-items: center;
        position: fixed;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        background: rgba(249, 249, 249, 0.89);
    }

    .vue-confirm-form-main {
        background-color: #ffffff;
        padding: 2em;
        border-radius: 0.3em;
        display: flex;
        flex-direction: column;
        align-items: center;
        border: 1px solid #e2e2e2;
        max-height: 90vh;
        overflow-y: scroll;
    }

    .vue-confirm-form-main select {
        outline: none;
        font-size: 1em;
        -webkit-appearance: none;
        border: 1px solid #dedede;
        text-align-last: center;
        background-color: #ffffff;
        padding: 0.5em;
    }

    .vue-confirm-error-message {
        font-size: 0.9em;
        color: #c41d25;
    }
</style>
