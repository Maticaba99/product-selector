<template>
  <div>
    <div class="textSelect">Select language</div>
    <select class="selectLang" v-model="lang">
      <option v-for="(language, index) in languages" :key="index">
        {{ language }}
      </option>
    </select>
    <div class="textSelect">Select Product</div>
    <multiselect
      v-model="selectedTypes"
      :key="option"
      placeholder="Select products"
      open-direction="bottom"
      :hide-selected="true"
      :options="options"
      :option-height="104"
      :multiple="true"
      :loading="isLoading"
      :internal-search="false"
      :close-on-select="false"
      label="id"
      track-by="id"
      :disabled="element.disabled"
      @input="onSelect"
      @onChange="onSelect"
      @search-change="fetchTypes"
      :clear-on-select="false"
      :options-limit="300"
      :limit-text="limitText"
      :custom-label="customLabel"
      :show-labels="false"
    >
      <template slot="tag" slot-scope="props">
        <div class="container selectedproduct">
          <div class="imageContainer">
            <img
              class="option__image"
              :src="props.option.image"
              :alt="props.option.name"
            />
            <div class="events">
              <span class="remove" @click="props.remove(props.option)">❌</span>
              <span
                class="add"
                @click="props.option.quantity = props.option.quantity - 1"
                >➖</span
              >
              <span
                class="add"
                @click="props.option.quantity = props.option.quantity + 1"
                >➕</span
              >
              <span
                class="remove"
                @click="selectedTypes.push([]) && selectedTypes.pop()"
                >✔</span
              >
            </div>
          </div>
          <span class="option__desc">
            <span class="option__title"
              ><strong>{{ props.option.name }}</strong></span
            >
            <span class="option__title">{{ props.option.id }}</span>
            <span class="option__title"
              ><strong>Qty: </strong> {{ props.option.quantity }}</span
            >
            <span class="option__title"
              ><strong>CAD: </strong> {{ props.option.price_cad }}</span
            >
            <span class="option__title"
              ><strong>USD: </strong> {{ props.option.price_usd }}</span
            >
          </span>
        </div>
      </template>
      <template slot="option" slot-scope="props"
        ><img
          class="option__image"
          :src="props.option.image"
          :alt="props.option.name"
        />
        <div class="option__desc">
          <span class="option__title"
            ><strong>{{ props.option.name }}</strong></span
          ><span class="option__small">{{ props.option.id }}</span>
          <span class="option__small">{{ props.option.dimensions }}</span>
          <span class="option__small">CAD:{{ props.option.price_cad }}</span>
          <span class="option__small">USD:{{ props.option.price_usd }}</span>
        </div>
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
      lang: "",
      languages: ["en", "en-CA", "fr-CA"]
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
    updateSize: {
      type: Function
    }
  },
  methods: {
    async fetchTypes(query) {
      const firstUpdateValue = this.element.config.QUERY.replace(
        "##query##",
        query
      );
      const langUpdateValue = firstUpdateValue.replace(
        "##language##",
        this.lang
      );
      this.isLoading = true;
      await fetch(this.element.config.API, {
        method: "post",
        headers: {
          Authorization: `Basic ${this.element.config.API_AUTH}`,
          "Content-Type": "application/json"
        },
        body: langUpdateValue
      })
        .then(response => response.json())
        .then(async json => {
          const options = await json.hits.hits.map(product => {
            return {
              id: product._source.productfields.unique_id,
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
          // eslint-disable-next-line no-console
          console.log(options);
          this.options = options;
          this.isLoading = false;
          this.updateSize();
        });
    },
    onSelect: function() {
      this.save(this.selectedTypes);
      this.updateSize();
    },
    save: function(value) {
      this.$emit("update:value", value);
      this.updateSize();
    }
  }
};
</script>

<style src="vue-multiselect/dist/vue-multiselect.min.css"></style>
<style scoped></style>
