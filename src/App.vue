<template>
  <span v-if="isLoading">Loading...</span>
  <div v-else>
    <form @reset="resetProductFilters">
      <label for="type">Type:</label>
      <select v-model="typeModel" id="type">
        <option
          v-for="item in filterProductTypes"
          :key="item"
        >
          {{ item }}
        </option>
      </select>
      
      <label for="min">Min price:</label>
      <input
        v-model="minPriceModel"
        :max="maxProductPrice"
        id="min"
        type="number"
        min="0"
      />

      <label for="max">Max price:</label>
      <input
        v-model="maxPriceModel"
        :max="maxProductPrice"
        id="max"
        type="number"
        min="0"
      />

      <button type="reset">Reset</button>
    </form>

    <ul v-if="filteredProducts.length">
      <li
        v-for="{ label, cost, type } in filteredProducts"
        :key="label"
      >
        {{ label }} {{ type }} {{ cost }} y.e.
      </li>
    </ul>
    <span v-else>
      Not found
    </span>
  </div>
</template>

<script>
import { items } from '../mock/data';

const DEFAULT_TYPE_OPTION = 'all';
const DEFAULT_MIN_PRICE_OPTION = 0;
const DEFAULT_MAX_PRICE_OPTION = 0;

export default {
  name: 'App',
  data() {
    return {
      products: [],
      isLoading: false,
      typeModel: DEFAULT_TYPE_OPTION,
      minPriceModel: DEFAULT_MIN_PRICE_OPTION,
      maxPriceModel: DEFAULT_MAX_PRICE_OPTION,
      validators: [
        (item) => item.cost >= this.minPriceModel,
        (item) =>  {
          if (Number(this.maxPriceModel) === DEFAULT_MAX_PRICE_OPTION) return true;

          return item.cost <= this.maxPriceModel
        },
        (item) => item.type === this.typeModel || this.typeModel === DEFAULT_TYPE_OPTION,
      ],
    };
  },
  computed: {
    filterProductTypes() {
      return [DEFAULT_TYPE_OPTION, ...new Set(this.products.map((item) => item.type))];
    },

    maxProductPrice() {
      return Math.max(...this.products.map(({ cost }) => cost))
    },

    filteredProducts() {
      return this.products.filter(this.isProductValid);
    },
  },
  created() {
    this.getProductData();
  },
  destroyed() {
    clearTimeout(this.productsTimeoutId);
  },
  methods: {
    getProductData() {
      const result = new Promise((resolve) => {
        this.setIsLoading(true);

        this.productsTimeoutId = setTimeout(() => resolve(items), 2000);
      });

      result
        .then((data) => this.products = data)
        .finally(() => this.setIsLoading(false));
    },

    isProductValid(item) {
      return this.validators.every((cb) => cb(item));
    },

    setIsLoading(flag) {
      this.isLoading = flag;
    },

    resetProductFilters() {
      this.typeModel = DEFAULT_TYPE_OPTION;
      this.minPriceModel = DEFAULT_MIN_PRICE_OPTION;
      this.maxPriceModel = DEFAULT_MAX_PRICE_OPTION;
    },
  },
};
</script>
