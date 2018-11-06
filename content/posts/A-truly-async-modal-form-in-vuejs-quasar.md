+++
date = "2018-11-05T15:20:48+00:00"
draft = true
title = "A truly async modal form in VueJs/Quasar"
slug = "A-truly-async-modal-form-in-vuejs-quasar"

+++

## introduction

In business applications, modal forms are typicall first class citizens. Therefor, it make sense to ensure that calling such a form is as simple as possible.

Modal forms take a rather specific place in the landscape of screen presentationals controls in that they are usually part of a control flow rather than simple screen real estate.
For example, you will typically open an modal form based on a button click, the modal form will allow the user to make kind of selection (e.g. selecting a product) and the program flow will coontinue with the selected 
product when the modal form is closed. So, it is more convenient to model this control flow via code rather than via markup.


## consuming a modal form

To make this idea as clear as possible, look at following markup where the "select-item" component encapsulates a modal form.
A button has an event handler which loads the modal form in an async manner and button event handler can immediately continue with the "result" from the modal form interaction.

```
this.selection = await this.$refs.productModal.open('myref', products)
// do now stuff with the returned result from the modal...
 ```

That makes things very consise and readable.

```

<template>
  <q-page padding>
    <select-item ref='productModal'></select-item>
    <q-btn :disable="!productsLoaded"  @click="selectProduct()">select product</q-btn>
    <pre>{{selection}}</pre>
  </q-page>
</template>

<script>
import SelectItem from './select-item.vue'
export default {
  components: {SelectItem},
  name: 'home',
  data() {
    return {
      selection: {},
    }
  },
  async created() {
    await this.$store.dispatch('products/load')
  },
  computed: {
    productsLoaded() {
      return this.$store.getters['products/loaded']
    }
  },
  methods: {
    async selectProduct () {
      try {
        
        const products = this.$store.getters['products/all']
        this.selection = await this.$refs.productModal.open('myref', products)
        // do stuff with the selected item.
      } catch (error) {
        this.selection = error
      }
    },
    
  }
}
</script>

```

## the select-item component
Now, in order to make the modal form callable with async, we must do inside the modal form component some promise magic.
So, our modal for component has one public method called 'open'. We make this method async and return a promise.
Basically, during the modal form lifetime 3 things can happen:

* the happy flow scenario: the user selects a product
* the cancel scenario: the user clicks cancel
* the error scenario: something bad happens 

These 3 scenario's a nicely handled with the promise.

```
<template>

  <q-modal no-backdrop-dismiss class="row" ref="modal" :content-css="{height: 'auto', width: 'auto', minWidth: $q.platform.is.desktop ? '40vw': '100vw'}">
    <q-card class="col-12">
      <q-toolbar>
        <q-toolbar-title>{{title}}</q-toolbar-title>
      </q-toolbar>
      <q-card-main>
        <q-scroll-area style=" height: 350px;">
        <q-list highlight>
          <q-item @click.native.prevent="selectClicked(item)" v-for="item in items" :key="item.id">
            <q-item-main :label="item.name">
            </q-item-main>
          </q-item>
        </q-list>
        </q-scroll-area>
      </q-card-main>
      <q-card-actions align=end>
        <q-btn color="faded" @click="cancelClicked()">Cancel</q-btn>
      </q-card-actions>
    </q-card>
  </q-modal>

</template>

<script>

export default {
  name: 'SelectItem',
  props: {
    title: {
      type: String,
      required: false,
      default: 'Select an item'
    }
  },
  data() {
    return {
      callerReference: '',
      items: [],
      promise: {}
    }
  },
  methods: {
    async open (callerReference, items) {
      this.items = items
      this.promise = new Promise((resolve, reject) => {
        this.returnDataFromModal = resolve
        this.returnErrorFromModal = reject
      })
      this.callerReference = callerReference
      return await this.$refs.modal
        .show()
        .then(() =>  this.promise)
    },
    cancelClicked() {
      this.$refs.modal.hide()
      this.returnDataFromModal( {
          data: {},
          callerReference: this.callerReference
        })
    },
    selectClicked(item) {
      this.$refs.modal.hide()
      if (item.name === 'errorproduct') {
        this.returnErrorFromModal({
          callerReference: this.callerReference,
          error: 'simulated error: something went wrong while retrieving the item'
      })
      } else {
        this.returnDataFromModal( {
          data: item,
          callerReference: this.callerReference
        })
      }
    }
  }
}
</script>

<style>
</style>

```

## Conclusion

Embrace promises and use the nice ES next feature async await.

Cheers

paul