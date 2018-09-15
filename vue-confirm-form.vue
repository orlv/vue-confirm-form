<template id="vue-confirm-form-template">
    <div>
        <span class="text-button p-0" :class="{'text-button-green': !busy }" @click="click">{{ buttonText }}</span>
        <div class="vue-confirm-form" v-if="busy" @click="cancel">
            <div @click.stop class="vue-confirm-form-main">
                <div class="bold center mb-1 font-1_3">{{ title }}</div>
                <table>
                    <tbody>
                    <tr v-for="(fieldValue, fieldName) in formFields">
                        <td>{{ fieldName }}</td>
                        <td v-if="typeof fieldValue === 'string'">
                            <input v-model="form[fieldName]" type="text" class="" :title="fieldName"/>
                        </td>
                        <td v-else-if="typeof fieldValue === 'number'">
                            <input v-model="form[fieldName]" type="number" class="" :title="fieldName"/>
                        </td>
                        <td v-else-if="Array.isArray(fieldValue)">
                            <div v-if="typeof fieldValue[0] === 'object'">
                                <div v-for="(checkbox, n) in fieldValue">
                                    <input title="" class="css-checkbox" :id="`vue-confirm-form-${fieldName}-${n}`"
                                           type="checkbox" :value="Object.values(checkbox)[0]"
                                           v-model="form[fieldName]">
                                    <label :for="`vue-confirm-form-${fieldName}-${n}`" class="css-label">
                                        {{ Object.keys(checkbox)[0] }}</label>
                                </div>
                            </div>
                            <select v-else v-model="form[fieldName]" :title="fieldName">
                                <option v-for="option in formFields[fieldName]">{{ option }}</option>
                            </select>
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
    template: '#vue-confirm-form-template',

    data: function () {
      return {
        busy: false,
        formFields: {},
        form: {}
      }
    },

    props: ['callback', 'title', 'text', 'confirm', 'fields', 'default'],

    created () {
      this.extractDefaultForm()
    },

    watch: {
      fields: function () {
        this.extractDefaultForm()
      }
    },

    computed: {
      buttonText: function () {
        return this.busy ? 'Cancel' : this.text
      },

      confirmText: function () {
        return this.confirm ? this.confirm : 'OK'
      }
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

      callConfirm: function () {
        this.busy = false
        this.callback(Object.assign({}, this.form))
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
</style>
