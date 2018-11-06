+++
date = "2018-11-05T12:20:48+00:00"
draft = false
title = "A pure client side minimal elastic search solution with VueJs/Quasar"
slug = "a-pure-client-side-elastic-search-solution-with-vuejs"

+++

## introduction
Although seriously simplified thanks to docker, setting up elastic search is a tedious task and requires quite a lot of plumbing in order to integrate it with a single page application.

An example of a such an elastic search can be found here: [elastic search sample](http://blog.opinionatedapps.com/ItemsJs-Vue-Demo/)

Under a specific set of conditions, elastic search functionality can be provided in a much simpler way, purely client side, in the browser.

In this blog post, I'll demonstrate this with a VueJs Single page application with the brilliant [Quasar framework](https://quasar-framework.org/).

## moving the meet of the elastic search to the browser

In the solution i'm proposing, I'm completely eliminating all server side elastic search functionality. For this blog post, it's even indifferent where the data we present in the typeical elastic search manner, come from.

Modern single page application or PWA's (progressive web apps) often retrieve data from the server in an unconvential manner via web workers or via socket communication. 

So, this post doesn't focus on that specific retrieval process, but purely on the client side elastic search presentation part.

The above link ([elastic search sample](http://blog.opinionatedapps.com/ItemsJs-Vue-Demo/)) I provided is in a fact a client side elastic search. 
So let's explorer how we can build such a search screen with VueJs

## the VueJs Search screen

The amount of markup and code needed is surprisingly low.
Basically the screen has a left part where the elastic search magic happens and right part where the search results are presented.
The left part is managed via a vuejs component called items-js-facets. We'll explorer that in a minute.
The components takes as input some elastic search config data and return via an event (searchResultUpdated)the search results.
The right part is basic vuejs stuff for rendering the search results.
```language-javascript
<template>
  <q-page padding>
    <div class="row">
      <div class="col-3">
        <items-js-facets :rows="jsonData" :configuration="configuration" 
            @searchResultUpdated="searchResultUpdated">
        </items-js-facets>
      </div>
      <div class="col-1"></div>
      <div class="col-7">
        <q-list>
          <q-item v-for="item of items" :key="item.name">
            <q-item-side><img style="width: 100px;" v-bind:src="item.image">
            </q-item-side>
            <q-item-main :label="item.name" :sublabel="item.description">
              <q-item-tile>
                <br>
                <q-chip small tag v-for="tag in item.tags"
                     :key="tag.key">{{ tag }} </q-chip>
              </q-item-tile>
            </q-item-main>
            <q-item-side right>{{item.year[0]}}</q-item-side>
          </q-item>
        </q-list>
      </div>
    </div>
  </q-page>
</template>
<style>
</style>
<script>

import rawJson from './imdb.json'
import ItemsJsFacets from './ItemsJsFacets.vue'
export default {
  components:{ItemsJsFacets},
  created() {
    this.jsonData = rawJson
     this.jsonData.forEach(e => {
        e.year = [e.year.toString()]
      })
  },
  data() {
    var configuration = {
      searchableFields: ['title', 'tags', 'actors'],
      sortings: {
        name_asc: {
          field: 'name', order: 'asc' }
       },
      aggregations: {
        tags: { title: 'Tags', size: 200 },
        actors: { title: 'Actors', size: 10 },
        genres: { title: 'Genres', size: 10 },
        country: { title: 'Country', size:20 },
        year: { title: 'Year', size:100 }
      }
    }
    return {
      jsonData: [],
      configuration,
      items: []

    }
  },
  methods: 
  {
    searchResultUpdated(d) {
      this.items = d
    }
  }
}
</script>
```

## introducing items-js

Let's get a bit closer now to the elastic search magic. I'm using an amazing library called ItemsJs, which is doing all the heavy lifting of generating facets and executing the search in a simple json document.

[ItemsJs on github](https://github.com/itemsapi/itemsjs)

## this items-js-facets component
The integration of ItemsJs in our vueJs is again very simple. I'm only adding a nice VueJs sauce which gives an elegant presentation of the facets. 
```language-javascript
<template>
  <div>
    <q-search inverted color='primary' v-model="query" @clear="reset()" placeholder="Type your search here..." clearable />
     <p class="text-right">Search performed in {{ searchResult.timings.search }} ms, facets in {{ searchResult.timings.facets }} ms</p> -->
    <br>
    <div>
      <span v-for="facet in searchResult.data.aggregations" :key="facet.name">
        <span v-for="bucket in filters[facet.name]" :key="bucket.key">
          <q-chip closable @hide="unselectBucket(facet.name,bucket)" color="primary">
            {{bucket}}
          </q-chip>
        </span>
      </span>
    </div>
    <div>
      <q-list dense v-for="facet2 in searchResult.data.aggregations" :key="facet2.name">
        <q-collapsible group="mygroup" :header-class="filters[facet2.name].length > 0?'text-bold text-italic':''" :label="getFacetLabel(facet2)">
          <q-item v-for="bucket in facet2.buckets" :key="bucket.key">
            <q-checkbox v-model="filters[facet2.name]" :val="bucket.key" :label="getBucketLabel(bucket)">
            </q-checkbox>
          </q-item>
        </q-collapsible>
      </q-list>
    </div>
  </div>
</template>
<style>
</style>
<script>


import itemsjs from 'itemsjs'
export default {
  props:['rows','configuration'],
  created() {
    this.itemsJsInstance = itemsjs(this.rows, this.configuration)
  },
  data() {
   var filters = {};
    Object.keys(this.configuration.aggregations).map(function(v) {
      filters[v] = [];
    })

    return {
      query: '',
      filters,
      itemsJsInstance: {}
    }
  },
  methods: {
    getFacetLabel(facet) {
      return facet.title
    },
    unselectBucket(facetName, tagBucket) {
      this.filters[facetName] = this.filters[facetName].filter(item => item !== tagBucket)
    },
    getBucketLabel(bucket) {
      return bucket.key + ' (' + bucket.doc_count + ')'
    },
    reset() {
      var filters = {}
      Object.keys(this.optionList.aggregations).map(function(v) {
        filters[v] = []
      })
      this.filters = filters
      this.query = ''
    },
  },
    computed: {
    searchResult() {
      let result =  this.itemsJsInstance.search({
        per_page:100,
        query: this.query,
        filters: this.filters
      })
      this.$emit('searchResultUpdated', result.data.items)
      return result
    }
  }
}
</script>
```
## will I need ever again the full blown server side elastic search ?

Yes you will !
The client side approach only works with a limited set of data (a few thousand records probably) but is of course extremly fast.

## where are the sources?

[github](https://github.com/paulvanbladel/ItemsJs-Vue-Demo)

[sample](http://blog.opinionatedapps.com/ItemsJs-Vue-Demo/)

Cheers

paul