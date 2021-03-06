<template>
  <div class="sidebar">
    <div class="filters">
      <h4>
        {{ $t('Filter') }}
      </h4>
      <div
        v-for="(filter, filterIndex) in availableFilters"
        :key="filterIndex"
      >
        <h5>
          {{ $t(filterIndex + '_filter') }}
        </h5>
        <div v-if="filterIndex === 'color'">
          <color-selector
            context="category"
            :attribute_code="filter.attribute_code"
            code="color"
            v-for="(color, index) in filter.options"
            :key="index"
            :id="color.id"
            :label="color.label"
          />
        </div>
        <div v-else-if="filter.frontend_input==='price'">
          <price-slider
            context="category"
            code="price"
            :id="getMaxPrice"
            :from="getMinPrice"
            :to="getMaxPrice"
            content="Price "
            label="Price Label"
          />
        </div>
        <div v-else-if="isSelector(filter.frontend_input)">
          <selector
            context="category"
            :filter-type="filter.frontend_input"
            :attribute_code="filter.attribute_code"
            :code="filter.attribute_code"
            v-for="(filterOption, index) in filter.options"
            :key="index"
            :id="filterOption.id"
            :label="filterOption.label"
          />
        </div>
        <div v-else>
          <generic-selector
            context="category"
            :attribute_code="filter.attribute_code"
            class="generic-select mb10 block"
            :code="filterIndex"
            v-for="(filterOption, index) in filter.options"
            :key="index"
            :id="filterOption.id"
            :label="filterOption.label"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { buildFilterProductsQuery } from '@vue-storefront/core/helpers'
import { mapGetters } from 'vuex'
import GenericSelector from './FilterTypes/GenericSelector'
import ColorSelector from './FilterTypes/ColorSelector'
import Selector from './FilterTypes/Selector'
import PriceSlider from './FilterTypes/PriceSlider'
import pickBy from 'lodash-es/pickBy'

export default {
  name: 'CategorySidebar',
  components: {
    GenericSelector,
    ColorSelector,
    Selector,
    PriceSlider
  },
  props: {
    filters: {
      type: Object,
      required: true
    }
  },
  data () {
    return {
      selectorFilterTypes: ['select', 'multiselect']
    }
  },
  computed: {
    ...mapGetters('category', ['getCurrentCategory', 'getActiveCategoryFilters', 'getCurrentCategoryProductQuery']),
    category () {
      return this.getCurrentCategory
    },
    availableFilters () {
      return pickBy(this.filters, (filter) => { return (filter.options.length) })
    },
    currentProductList () {
      return this.$store.state.product.list.items
    },
    getMaxPrice () {
      return Math.max.apply(Math, this.currentProductList.map((attribute) => { return attribute.priceInclTax }))
    },
    getMinPrice () {
      return Math.min.apply(Math, this.currentProductList.map((attribute) => { return attribute.priceInclTax }))
    },
    activeFilters () {
      return this.getActiveCategoryFilters
    }
  },
  methods: {
    sortById (filters) {
      return [...filters].sort((a, b) => { return a.id - b.id })
    },
    isSelector (filterType) {
      return this.selectorFilterTypes.includes(filterType)
    },
    resetAllFilters () {
      this.$bus.$emit('filter-reset')
      this.$store.dispatch('category/resetFilters')
      this.$store.dispatch('category/searchProductQuery', buildFilterProductsQuery(this.category, this.activeFilters))
      this.$store.dispatch('category/products', {searchProductQuery: this.getCurrentCategoryProductQuery})
    }
  }
}
</script>
