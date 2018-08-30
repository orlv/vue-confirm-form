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
      checkbox1: [{ 'title 1': '1' }, { 'title 2': '2' }, { 'title 3': '3' }]
      select1: ['option1', 'option2', 'option3', 'option4']
    },

    defaultFields: {
      checkbox1: ['2, '3']
      select1: 'option2'
      text2: 'Hello'
    }
  }
},

methods: {
  myMethod: function () {
    console.log('Form:', form)
  }
}
```
