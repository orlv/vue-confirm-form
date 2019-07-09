# vue-confirm-form

Vue form component

#### Installation

```
npm i -D vue-confirm-form
```


#### Example

```
<confirm-form :callback="form => myMethod(form)"
              :text="'Show form'"
              :confirm="'Change'"
              :title="'Form'"
              :fields="fields"
              :disabled="disabledFields"
              :default="defaultFields"/>
```

```
import ConfirmForm from 'vue-confirm-form'
```

```
components: {
    ConfirmForm
},

data: function () {
  return {
    fields: {
      text1: '',
      text2: '',
      num: 123,
      checkbox1: { 'val1': 'val1', 'val2': 'val2', 'val3': 'val3' },
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
  myMethod: function () {
    console.log('Form:', form)
  }
}
```
