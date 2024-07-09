<template>
  <v-select
    placeholder="PRODUTO"
    @search="debouncedSearch"
    :filter="fuseSearch"
    :options="products"
    :get-option-label="getOptionLabel"
    @option:selected="handleSelect"
    :clearable="false"
    :components="{ OpenIndicator, Deselect }
    "
  >
    <template #option="{ nome_produto, codigo_produto }">
      <div class="props">
        <p>{{ codigo_produto }}</p>
        <p> - </p>
        <p class="nome-produto">{{ nome_produto }}</p>
      </div>
    </template>
    <template #no-options="{ search, searching, loading }">
      <p class="no-options">Sem opção disponível</p>
    </template>
  </v-select>
</template>

<script>
import vSelect from 'vue-select'
import Fuse from 'fuse.js'
import 'vue-select/dist/vue-select.css';
import axios from 'axios';
import { debounce } from 'lodash';
import { PhCaretDown, PhX } from '@phosphor-icons/vue'; // Importe o ícone
import { h } from 'vue'

export default {
  components: {
    vSelect,
    PhCaretDown,
    PhX, // Adicione o ícone como um componente
  },
  props: {
    content: { type: Object, required: true },
    uid: { type: String, required: true },
  },
  data() {
    return {
      products: [],
      OpenIndicator: { 
        render() {
          return h(PhCaretDown, { size: 24, class: 'toggle' });
        }
      },
      Deselect: {
        render() {
          return h(PhX , { size: 24 });
        }
      }
    };
  },
  setup(props) {
    const { value: variableResult1, setValue: setValue1 } = wwLib.wwVariable.useComponentVariable({
      uid: props.uid,
      name: 'Input Value',
      type: 'string',
      defaultValue: ''
    });

    const { value: variableResult2, setValue: setValue2 } = wwLib.wwVariable.useComponentVariable({
      uid: props.uid,
      name: 'Product',
      type: 'object',
      defaultValue: ''
    });

    return {
      variableResult1,
      setValue1,
      variableResult2,
      setValue2,
    };
  },
  methods: {
    async handleSearch(search = '') {
      this.setValue1(search);
      try {
        const response = await axios.get(`https://eunaoestudo.com.br:3395/webhook/erp/v1/produtos`, {
          params: {
            filter: search
          }
        });
        this.products = response.data;
        this.$emit('trigger-event', { name: 'onSearch', event: { value: search } });
      } catch (error) {
        console.error('Error fetching products:', error);
      }
    },
    handleSelect(selectedOption) {
      this.setValue2(selectedOption);
    },
    fuseSearch(options, search) {
      const fuse = new Fuse(options, {
        keys: [
          'nome_produto',
          'codigo_produto',
          'marca_produto',
          'ean'
        ],
        shouldSort: true,
      });
      return search.length
        ? fuse.search(search).map(({ item }) => item)
        : options;
    },
    getOptionLabel(option) {
      return option.nome_produto || option.codigo_produto;
    },
  },
  created() {
    this.debouncedSearch = debounce(this.handleSearch, 300);
    this.debouncedSearch();
  },
  
};
</script>

<style>

@import url('https://fonts.googleapis.com/css2?family=Montserrat:ital,wght@0,100..900;1,100..900&display=swap');

@media (min-width: 1024px) {
  .vs__dropdown-menu {
    overflow-x: hidden !important;
    width: auto !important;
  }
}

@media (max-width: 767px) {
  .vs__dropdown-menu {
    max-height: 30vh !important;
  }
}

.props {
  display: flex !important;
  flex-direction: row !important;
  column-gap: 8px !important;
  align-items: center !important;
}

:root, :host {
  --vs-colors--lightest: rgba(60, 60, 60, 0.26);
  --vs-colors--light: rgba(60, 60, 60, 0.5);
  --vs-colors--dark: #333;
  --vs-colors--darkest: rgba(0, 0, 0, 0.15);

  /* Search Input */
  --vs-search-input-color: inherit;
  --vs-search-input-bg: rgb(255, 255, 255);
  --vs-search-input-placeholder-color: #b5b5b5;

  /* Font */
  --vs-font-size: 16px;
  --vs-line-height: auto;

  /* Disabled State */
  --vs-state-disabled-bg: rgb(248, 248, 248);
  --vs-state-disabled-color: var(--vs-colors--light);
  --vs-state-disabled-controls-color: var(--vs-colors--light);
  --vs-state-disabled-cursor: not-allowed;

  /* Borders */
  --vs-border-color: #a4a4a4 !important;
  --vs-border-width: 0px !important;
  --vs-border-style: solid !important;
  --vs-border-radius: 14px !important;

  /* Actions: house the component controls */
  --vs-actions-padding: 0px;

  /* Component Controls: Clear, Open Indicator */
  --vs-controls-color: #a4a4a4;
  --vs-controls-size: 1;
  --vs-controls--deselect-text-shadow: 0 1px 0 #fff;

  /* Selected */
  --vs-selected-bg: #f0f0f0;
  --vs-selected-color: var(--vs-colors--dark);
  --vs-selected-border-color: var(--vs-border-color);
  --vs-selected-border-style: var(--vs-border-style);
  --vs-selected-border-width: var(--vs-border-width);

  /* Dropdown */
  --vs-dropdown-bg: #fff;
  --vs-dropdown-color: #161616;
  --vs-dropdown-z-index: 1000;
  --vs-dropdown-min-width: 160px;
  --vs-dropdown-max-height: 350px;
  --vs-dropdown-box-shadow: 0px 1px 28px 0px rgba(0,0,0,0.12);

  /* Options */
  --vs-dropdown-option-bg: #000;
  --vs-dropdown-option-color: var(--vs-dropdown-color);
  --vs-dropdown-option-padding: 12px 12px;

  /* Active State */
  --vs-dropdown-option--active-bg: #f5eeff;
  --vs-dropdown-option--active-color: #6418c3;

  /* Deselect State */
  --vs-dropdown-option--deselect-bg: #fb5858;
  --vs-dropdown-option--deselect-color: #fff;

  /* Transitions */
  --vs-transition-timing-function: cubic-bezier(1, -0.115, 0.975, 0.855);
  --vs-transition-duration: 150ms;
}

.vs__dropdown-toggle {
  height: 50px !important;
  border-width: 1px !important;
  display: flex;
  align-items: center;
}

.vs__selected {
  margin: 4px 12px 0px 12px !important;
  display: flex;
  align-items: center;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  font-family: "Montserrat", sans-serif;
  font-weight: 500;
}

.vs__selected-options {
  flex-wrap: nowrap;
  max-width: calc(100% - 41px);
}

.vs__selected {
  display: block;
  white-space: nowrap;
  text-overflow: ellipsis;
  max-width: 100%;
  overflow: hidden;
}

.vs__search {
  position: absolute;
}

.vs--open .vs__search {
  position: static;
}

/* .vs__dropdown-option {
  white-space: normal;
} */

.vs__search, .vs__search:focus {
  padding: 0 12px !important;
  font-family: "Montserrat", sans-serif;
  font-weight: 500;
}

.vs__dropdown-option {
  font-family: "Montserrat", sans-serif;
  font-weight: 500;
}

.vs__actions {
  align-items: normal !important;
}

.no-options {
  padding: 12px !important;
  font-family: "Montserrat", sans-serif;
  font-weight: 500;
}


</style>
