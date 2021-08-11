<template>
  <div>
    <div>
      <toggle-button
        @change="onChangeEventHandler"
        :value="false"
        :labels="{ checked: 'French', unchecked: 'Default' }"
      />
    </div>
    <multiselect
      v-model="selectedTypes"
      placeholder="Select types"
      open-direction="bottom"
      :options="options"
      :multiple="true"
      :loading="isLoading"
      label="sys_name"
      track-by="sys_name"
      :disabled="element.disabled"
      @input="onSelect"
    >
      <template slot="tag" slot-scope="props">
        <span class="multiselect__tag">
          <strong>Display_name:{{ props.option.name }}</strong>
          <strong>Name:{{ props.option.sys_name }}</strong>
          <span
            class="multiselect__tag-icon"
            @click="props.remove(props.option)"
          >
          </span>
        </span>
      </template>
      <template slot="singleLabel" slot-scope="{ option }">
        <strong>Display_name:{{ option.name }}</strong>
        <strong>Name:{{ option.sys_name }}</strong>
      </template>
    </multiselect>
  </div>
</template>

<script>
import Vue from "vue";
import Multiselect from "vue-multiselect";
import { ToggleButton } from "vue-js-toggle-button";

Vue.component("ToggleButton", ToggleButton);

export default {
  components: {
    Multiselect
  },
  data() {
    return {
      selectedTypes: this.value || [],
      options: [],
      isLoading: true,
      xContinuation: "",
      lang: false
    };
  },
  created() {
    this.fetchTypes();
  },
  props: {
    element: {
      type: Object,
      required: true
    },
    context: {
      type: Object,
      required: true
    },
    value: {
      type: Object
    },
    disable: {
      type: Boolean
    }
  },
  methods: {
    limitText(count) {
      return `and ${count} other countries`;
    },
    async fetchTypes() {
      const url = `https://deliver.kontent.ai/${
        this.element.config.projectId
      }/items-feed?${
        this.element.config.filter ? "&" + this.element.config.filter : ""
      }`;

      const urlWithLanguage = `https://deliver.kontent.ai/${
        this.element.config.projectId
      }/items-feed?${
        this.element.config.filter
          ? "&" + this.element.config.filter + `&language=fr-CA`
          : ""
      }`;
      const finalUrl = this.lang ? urlWithLanguage : url;
      // eslint-disable-next-line no-console
      console.log(finalUrl, this.lang);
      if (!this.disable) {
        do {
          await fetch(finalUrl, {
            headers: {
              "Content-Type": "application/json",
              Authorization: `Bearer ${this.element.config.secureAccess}`,
              "x-continuation": this.xContinuation
            }
          })
            .then(response => {
              this.xContinuation = response.headers.get("x-continuation");

              // eslint-disable-next-line no-console
              console.log(this.xContinuation, "xContinuation");
              return response.json();
            })
            .then(json => {
              // Actual language
              const actualLanguage = this.lang ? "en" : "fr-CA";
              json.items.map(type => {
                if (type.system.language === actualLanguage) {
                  // eslint-disable-next-line no-console
                  console.log(type, "data");
                  const res = {
                    name: type.elements.display_name.value,
                    sys_name: type.system.name
                  };

                  return this.options.push(res);
                }
              });

              if (!this.xContinuation) {
                this.isLoading = false;
              }
            });
        } while (this.xContinuation !== null);
      }
    },
    async onChangeEventHandler() {
      this.lang = !this.lang;
      // eslint-disable-next-line no-console
      console.log(this.lang, "updated");
      this.xContinuation = "";
      await this.fetchTypes();
    },
    onSelect: function() {
      this.save(this.selectedTypes);
    },
    save: function(value) {
      this.$emit("update:value", value);
    }
  }
};
</script>

<style src="vue-multiselect/dist/vue-multiselect.min.css"></style>

<style scoped></style>
