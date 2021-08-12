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
      this.value = this.element.value ? JSON.parse(this.element.value) : null;
      await fetch(this.element.config.API, {
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
                        "productfields.unique_id.keyword": "141332-L-000-004"
                    }
                } ,
                 {
                    "term": {
                        "language.keyword": "fr-CA"
                    }
                } 
                ]
        }
    }
}`
      }).then(async response => {
        // eslint-disable-next-line no-console
        console.log(await response.json(), "ehere");
      });

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
