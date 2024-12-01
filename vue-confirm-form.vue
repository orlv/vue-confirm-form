<template>
    <div>
        <span class="start-button" :class="{'busy': !busy }" @click="click">{{ buttonText }}</span>
        <div v-if="busy" class="vue-confirm-form" @click="cancel">
            <div class="vue-confirm-form-main" @click.stop>
                <div class="confirm-form-copy-paste-buttons">
                    <div class="vue-confirm-clipboard margin-r1" @click="copyToClipboard">copy
                    </div>
                    <div class="vue-confirm-clipboard" @click="pasteFromClipboard">paste
                    </div>
                </div>
                <div class="vue-confirm-title">{{ title }}</div>
                <div class="vue-confirm-error-message">{{ message }}</div>
                <table>
                    <tbody>
                    <tr v-for="(fieldValue, fieldName) in formFields" :key="fieldName"
                        :class="{'validation-failed': fieldName === badField}">
                        <td @click="onSelect(fieldName, form[fieldName])">{{ fieldName }}</td>
                        <td v-if="disabledFields[fieldName]">{{ form[fieldName] }}</td>
                        <td v-else-if="typeof fieldValue === 'string'">
                            <input v-model="form[fieldName]" type="text" class="" :title="fieldName"/>
                        </td>
                        <td v-else-if="typeof fieldValue === 'number'">
                            <input v-model="form[fieldName]" type="number" class="" :title="fieldName"/>
                        </td>
                        <td v-else-if="Array.isArray(fieldValue)">
                            <select v-model="form[fieldName]" :title="fieldName">
                                <option v-for="option in fieldValue" :key="option" :value="option">{{ option }}</option>
                            </select>
                        </td>
                        <td v-else-if="typeof fieldValue === 'object'">
                            <div v-if="typeof form[fieldName] === 'string' || typeof form[fieldName] === 'number'">
                                <select v-model="form[fieldName]" :title="fieldName">
                                    <option v-for="(value, label) in fieldValue"
                                            :key="value"
                                            :value="value">{{ label }}
                                    </option>
                                </select>
                            </div>
                            <div v-for="(checkboxValue, value) in fieldValue" v-else :key="value">
                                <div v-if="form[fieldName].includes(value)"
                                     class="cursor-pointer"
                                     @click="form[fieldName].splice(form[fieldName].findIndex(v => v === value), 1)">
                                    <span>☑</span>
                                    <span>{{ value }}</span>
                                </div>
                                <div v-else
                                     class="cursor-pointer"
                                     @click="form[fieldName].push(value)">
                                    <span>☐</span>
                                    <span>{{ value }}</span>
                                </div>
                            </div>
                        </td>
                        <td v-else-if="typeof fieldValue === 'boolean'">
                            <div class="cursor-pointer" @click="form[fieldName] = !form[fieldName]">
                                <span v-if="form[fieldName]">☑</span>
                                <span v-else>☐</span>
                            </div>
                        </td>
                    </tr>
                    <tr>
                        <td colspan="2" class="confirm-button">
                            <span @click="callConfirm">{{ confirmText }}</span>
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

  emits: ['on-select', 'on-change'],

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

  watch: {
    form: {
      handler (val) {
        const prevForm = this.prevForm

        this.prevForm = Object.assign({}, val)

        for (const key of Object.keys(val)) {
          const newVal = val[key]

          if (prevForm[key] !== newVal) {
            this.$emit('on-change', key, newVal, this.extract())
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

  created () {
  },

  methods: {
    onSelect (field, val) {
      this.$emit('on-select', field, val)
    },

    hookEsc () {
      window.addEventListener('keydown', this.keyDownHandler)
    },

    removeEscHook () {
      window.removeEventListener('keydown', this.keyDownHandler)
    },

    keyDownHandler (e) {
      if (/Esc/.test(e.code)) {
        this.cancel()
      } else if (/Enter/.test(e.code)) {
        this.callConfirm()
      }
    },

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
          : typeof val === 'object' && val !== null
            ? Object.keys(val)
            : val
      }
    },

    extract () {
      return Object.entries(this.fields).reduce((out, [field, inVal]) => {
        const resVal = this.form[field]

        if ((typeof inVal === 'number' || (Array.isArray(inVal) && typeof inVal[0] === 'number')) && typeof resVal === 'string') {
          const num = parseFloat(resVal)

          out[field] = isNaN(num) ? '' : num
        } else if (typeof inVal === 'object' && !Array.isArray(inVal) && Array.isArray(resVal)) {
          out[field] = resVal.reduce((acc, label) => {
            acc[label] = inVal[label]
            return acc
          }, {})
        } else {
          out[field] = resVal
        }

        return out
      }, {})
    },

    async callConfirm () {
      this.busy = false
      this.message = ''
      this.badField = ''
      this.removeEscHook()

      const out = this.extract()
      const res = await this.callback(out)

      if (res) {
        if (typeof res === 'string') {
          this.busy = true
          this.hookEsc()
          this.message = res
        } else if (res.msg || res.field) {
          this.busy = true
          this.hookEsc()
          this.message = res.msg || ''
          this.badField = res.field || ''
        } else {
          this.extractDefaultForm()
        }
      } else {
        this.extractDefaultForm()
      }
    },

    cancel () {
      this.busy = false
      this.loading = false
      this.message = ''
      this.badField = ''
      this.extractDefaultForm()
      this.removeEscHook()
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
            this.hookEsc()
            this.extractDefaultForm(defaultValues)
            this.busy = true
          }
        } else {
          this.hookEsc()
          this.extractDefaultForm()
          this.busy = true
        }
      }
    }
  }
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

.start-button {
    padding: 0;
    cursor: pointer;
    font-weight: 700;
    text-align: center;
    user-select: none;
}

.start-button.busy {
    color: #70ab6c;
}

.start-button.busy:hover {
    color: #409a39;
}

.vue-confirm-clipboard {
    background-color: #f1f1f1;
    color: #000000;
    cursor: pointer;
    width: 4em;
    font-size: 0.9em;
    text-align: center;
}

.vue-confirm-clipboard:hover {
    background-color: #ebebeb;
}

.margin-r1 {
    margin-right: 1em;
}

.vue-confirm-title {
    font-weight: bold;
    text-align: center;
    font-size: 1.3em;
    margin-bottom: 1em;
}

.confirm-button {
    text-align: center;
    padding: 1em;
    font-size: 1.3em;
}

.confirm-button > span {
    padding: 0 0.6em;
    cursor: pointer;
    font-weight: 700;
    text-align: center;
    user-select: none;
    color: #70ab6c;
}

.confirm-button > span:hover {
    color: #409a39;
}

.cursor-pointer {
    cursor: pointer;
}
</style>
