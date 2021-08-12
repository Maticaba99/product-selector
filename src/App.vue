<template>
  <div id="app">
    <Callout v-if="errorMessage" type="error" title="Something went wrong">
      <p>
        {{ errorMessage }}
      </p>
    </Callout>
    <div v-if="loaded">
      <ContentTypeSelector
        :element="element"
        :context="context"
        :value.sync="value"
        :updateSize="updateSize"
      />
      <Debug
        :element="element"
        :context="context"
        :value="value"
        @handleDisable="handleDisable"
      />
      <resize-observer @notify="updateSize" />
    </div>
  </div>
</template>

<script>
/*global CustomElement*/
/* eslint-disable no-console */
import Callout from "./components/Callout";
import Debug from "./components/Debug";
import ContentTypeSelector from "./components/ContentTypeSelector";
import Vue from "vue";
import { GlobalEventBus } from "./globalEventBus";

export default {
  name: "app",
  components: {
    Callout,
    Debug,
    ContentTypeSelector
  },
  data: () => ({
    loaded: false,
    errorMessage: "",
    element: {},
    context: {},
    value: null
  }),
  created: function() {
    try {
      CustomElement.init(this.initialize);

      CustomElement.onDisabledChanged(this.handleDisable);
      CustomElement.observeElementChanges([], elementCodename => {
        GlobalEventBus.$emit("onElementChanged", elementCodename[0]);
      });
    } catch (error) {
      this.errorMessage = error;
    }
  },
  methods: {
    getElementValue: function(elementCodename) {
      return new Promise((resolve, reject) => {
        try {
          CustomElement.getElementValue(elementCodename, value => {
            resolve(value);
            /* this.updateSize(); */
          });
        } catch (error) {
          reject(error);
        }
      });
    },
    handleDisable: function(disableState) {
      this.element.disabled = disableState;
      /* this.updateSize(); */
    },
    initialize: async function(element, context) {
      this.element = element;
      this.context = context;
      const previousValueJSON = this.element.value
        ? JSON.parse(this.element.value)
        : null;
      const language = this.context.variant.codename;

      const valueUpdated = await Promise.all(
        previousValueJSON.map(async item => {
          return await fetch(this.element.config.API, {
            method: "post",
            headers: {
              Authorization: `Basic ${this.element.config.API_AUTH}`,
              "Content-Type": "application/json"
            },
            body: `{
    "query": {
        "bool": {
            "must": [
               {
                    "term": {
                        "productfields.unique_id.keyword": "${item.id}"
                    }
                } ,
                 {
                    "term": {
                        "language.keyword": "${language}"
                    }
                } 
                ]
        }
    }
}`
          })
            .then(response => response.json())
            .then(async json => {
              const options = await json.hits.hits.map(product => {
                return {
                  id: product._source.productfields.unique_id + " working",
                  name: product._source.productfields.product_name,
                  dimensions:
                    product._source.productcard &&
                    product._source.productcard.dimensionsin,
                  image:
                    product._source.productcard &&
                    product._source.productcard.featureimage,
                  quantity: 1,
                  price_cad: product._source.productfields.listprice_cad,
                  price_usd: product._source.productfields.listprice_usd
                };
              });
              const selectedPrevious = options.length > 0 && options[0];

              return selectedPrevious;
            });
        })
      );

      this.element.disable
        ? console.log("esta disabled", this.element.value, this.element.disable)
        : console.log("no indabled", valueUpdated, this.element.disable);
      this.value = this.element.value ? JSON.parse(this.element.value) : null;
      this.loaded = true;
      /* this.updateSize(); */
    },
    save: function(value) {
      // Explicitly using == to match both null and undefined
      const toSave = value == null ? null : JSON.stringify(value);
      this.element.value = toSave;
      CustomElement.setValue(toSave);
      /*       this.updateSize(); */
    },
    updateSize() {
      Vue.nextTick(function() {
        const height = Math.max(
          document.body.scrollHeight,
          document.body.offsetHeight,
          document.documentElement.offsetHeight
        );
        CustomElement.setHeight(height);
      });
    }
  },
  watch: {
    value: function(newValue) {
      this.save(newValue);
    }
  }
};
</script>

<style>
html {
  position: relative;
}
#app {
  padding-top: 0.5em;
}
</style>
