<template id="vue-confirm-form-template">
    <div>
        <span class="text-button p-0" :class="{'text-button-green': !busy }" @click="click">{{ buttonText }}</span>
        <div v-if="busy" class="vue-confirm-form" @click="cancel">
            <div class="vue-confirm-form-main" @click.stop>
                <div class="confirm-form-copy-paste-buttons">
                    <div class="button-gray cursor-pointer w4 font-09 center mr-1" @click="copyToClipboard">copy
                    </div>
                    <div class="button-gray cursor-pointer w4 font-09 center" @click="pasteFromClipboard">paste
                    </div>
                </div>
                <div class="bold center font-1_3 mb-1">{{ title }}</div>
                <div class="vue-confirm-error-message">{{ message }}</div>
                <table>
                    <tbody>
                    <tr v-for="(fieldValue, fieldName) in formFields" :key="fieldName"
                        :class="{'validation-failed': fieldName === badField}">
                        <td>{{ fieldName }}</td>
                        <td v-if="disabledFields[fieldName]">{{ form[fieldName] }}</td>
                        <td v-else-if="typeof fieldValue === 'string'">
                            <input v-model="form[fieldName]" type="text" class="" :title="fieldName"/>
                        </td>
                        <td v-else-if="typeof fieldValue === 'number'">
                            <input v-model="form[fieldName]" type="number" class="" :title="fieldName"/>
                        </td>
                        <td v-else-if="Array.isArray(fieldValue)">
                            <select v-model="form[fieldName]" :title="fieldName">
                                <option v-for="option in fieldValue" :key="option">{{ option }}</option>
                            </select>
                        </td>
                        <td v-else-if="typeof fieldValue === 'object'">
                            <div v-for="checkbox in Object.keys(fieldValue)" :key="checkbox">
                                <input :id="`vue-confirm-form-${fieldName}-${checkbox}`"
                                       v-model="form[fieldName]"
                                       title="" class="css-checkbox" type="checkbox"
                                       :value="checkbox">
                                <label :for="`vue-confirm-form-${fieldName}-${checkbox}`" class="css-label">
                                    {{ checkbox }}</label>
                            </div>
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
export default {
  props: ['getDefaultValues', 'callback', 'title', 'text', 'confirm', 'fields', 'default', 'disabled'],

  data () {
    return {
      busy: false,
      formFields: {},
      form: {},
      prevForm: {},
      prevDefault: Object.assign({}, this.default),
      message: '',
      badField: ''
    }
  },

  computed: {
    disabledFields () {
      const disabled = Array.isArray(this.disabled)
        ? this.disabled.filter(field => typeof field === 'string')
        : typeof this.disabled === 'object' ? Object.keys(this.disabled) : []

      return disabled.reduce((res, field) => {
        res[field] = true
        return res
      }, {})
    },

    buttonText () {
      return this.busy ? 'Cancel' : this.loading ? 'Loading' : this.text
    },

    confirmText () {
      return this.confirm ? this.confirm : 'OK'
    }
  },

  created () {
  },

  watch: {
    form: {
      handler (val) {
        const prevForm = this.prevForm

        this.prevForm = Object.assign({}, val)

        for (const key of Object.keys(val)) {
          const newVal = val[key]

          if (prevForm[key] !== newVal) {
            this.$emit('onChange', key, newVal)
            break
          }
        }
      },
      deep: true
    },

    default: {
      handler (val) {
        const prevDefault = this.prevDefault

        this.prevDefault = Object.assign({}, val)

        for (const key of Object.keys(val)) {
          const newVal = val[key]

          if (prevDefault[key] !== newVal) {
            this.setFormValue(key, newVal)
          }
        }
      },
      deep: true
    }
  },

  methods: {
    async copyToClipboard () {
      try {
        if (navigator.clipboard) {
          const textForm = JSON.stringify(this.form)

          await navigator.clipboard.writeText(textForm)
        }
      } catch (e) {
        console.error(`Can't copy to clipboard`, e)
      }
    },

    async pasteFromClipboard () {
      try {
        if (navigator.clipboard) {
          const textForm = await navigator.clipboard.readText()
          const objForm = JSON.parse(textForm)

          Object.keys(objForm).forEach(field => {
            if (field in this.form) {
              this.form[field] = objForm[field]
            }
          })
        }
      } catch (e) {
        console.error(`Can't paste from clipboard`, e)
      }
    },

    extractDefaultForm (defaultFields) {
      this.formFields = Object.assign({}, this.fields || {})

      this.form = this.prevForm = Object.keys(this.formFields).reduce((acc, field) => {
        const val = this.formFields[field]

        if (Array.isArray(val)) {
          acc[field] = val[0]
        } else if (typeof val === 'object') {
          acc[field] = acc[field] ? Object.keys(acc[field]) : []
        } else {
          acc[field] = val
        }

        return acc
      }, {})

      const defaultValues = defaultFields || this.default

      if (defaultValues) {
        Object.keys(defaultValues).forEach(key => {
          this.setFormValue(key, defaultValues[key])
        })
      }
    },

    /**
     * @param {string} key
     * @param {Array|object|string|number} val
     */
    setFormValue (key, val) {
      if (key in this.form) {
        this.form[key] = Array.isArray(val)
          ? val.slice()
          : typeof val === 'object'
            ? Object.keys(val)
            : val
      }
    },

    async callConfirm () {
      this.busy = false
      this.message = ''
      this.badField = ''

      const out = {}

      Object.keys(this.fields).forEach(field => {
        const inVal = this.fields[field]
        const resVal = this.form[field]

        if ((typeof inVal === 'number' || (Array.isArray(inVal) && typeof inVal[0] === 'number')) && typeof resVal === 'string') {
          const num = parseFloat(resVal)

          out[field] = isNaN(num) ? '' : num
        } else if (typeof inVal === 'object' && !Array.isArray(inVal)) {
          out[field] = resVal.reduce((acc, title) => {
            acc[title] = inVal[title]
            console.log(acc)
            return acc
          }, {})
        } else {
          out[field] = resVal
        }
      })

      const res = await this.callback(out)

      console.log('res', res)
      if (res) {
        if (typeof res === 'string') {
          this.busy = true
          this.message = res
        } else if (res.msg || res.field) {
          this.busy = true
          this.message = res.msg || ''
          this.badField = res.field || ''
        }
      }
    },

    cancel () {
      this.busy = false
      this.loading = false
      this.message = ''
      this.badField = ''
      this.extractDefaultForm()
      this.$emit('cancel')
    },

    async click () {
      if (this.loading) {
        this.loading = false
      } else if (this.busy) {
        this.cancel()
      } else {
        if (this.getDefaultValues) {
          this.loading = true
          const defaultValues = await this.getDefaultValues()

          if (defaultValues && this.loading) {
            this.loading = false
            this.extractDefaultForm(defaultValues)
            this.busy = true
          }
        } else {
          this.extractDefaultForm()
          this.busy = true
        }
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
        font-size: 1.1em;
        color: #ff284f;
        margin-bottom: 1em;
    }

    .confirm-form-copy-paste-buttons {
        display: flex;
        flex-direction: row;
        justify-content: flex-end;
        width: 100%;
        margin: -1.5em -3em 0.5em 0;
    }

    .validation-failed {
        outline: 2px solid #ff284f;
    }
</style>
