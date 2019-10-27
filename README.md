# vue-confirm-form

Vue form component

#### Installation

```
npm i -D vue-confirm-form
```


#### Example

```javascript
<confirm-form :callback="form => myMethod(form)"
              :text="'Show form'"
              :confirm="'Change'"
              :title="'Form'"
              :fields="fields"
              :disabled="disabledFields"
              :default="defaultFields"
              @onChange="onChange"/>
```

```javascript
import ConfirmForm from 'vue-confirm-form'
```

```javascript
components: {
  ConfirmForm
},

data () {
  return {
    fields: {
      text1: '',
      text2: '',
      num: 123,
      checkbox1: { 'title1': 'val1', 'title2': 'val2', 'title3': 'val3' },
      select1: ['option1', 'option2', 'option3', 'option4']
    },

    defaultFields: {
      checkbox1: ['2, '3']
      select1: 'option2'
      text2: 'Hello'
    },
    
    disabledFields: {
      text2: true
    }
  }
},

methods: {
  myMethod () {
    console.log('Form:', form)
  },

  formChanged (key, val) {
    console.log(`Changed '${key}':`, val)

    if (key === 'text1') {
      text2 = val
    }
  }
}
```
