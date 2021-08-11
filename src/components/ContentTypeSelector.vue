<template>
  <div>
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
import Multiselect from "vue-multiselect";

export default {
  components: {
    Multiselect
  },
  data() {
    return {
      selectedTypes: this.value || [],
      options: [],
      isLoading: true,
      xContinuation: ""
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
      // eslint-disable-next-line no-console
      console.log(this.context);
      if (!this.disable) {
        const language = this.context.variant.codename;
        const url = `https://deliver.kontent.ai/${
          this.element.config.projectId
        }/items-feed?${
          this.element.config.filter
            ? "&" + this.element.config.filter + `&language=${language}`
            : ""
        }`;
        do {
          await fetch(url, {
            headers: {
              "Content-Type": "application/json",
              Authorization: `Bearer ${this.element.config.secureAccess}`,
              "x-continuation": this.xContinuation
            }
          })
            .then(response => {
              this.xContinuation = response.headers.get("x-continuation");
              return response.json();
            })
            .then(json => {
              // eslint-disable-next-line no-console
              console.log(json, "check");

              json.items.map(type => {
                const res = {
                  name: type.elements.display_name.value,
                  sys_name: type.system.name
                };
                // eslint-disable-next-line no-console
                console.log(res);
                return this.options.push(res);
              });
              // eslint-disable-next-line no-console
              console.log(this.options[2]);

              if (!this.xContinuation) {
                this.isLoading = false;
              }
            });
        } while (this.xContinuation !== null);
      }
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
