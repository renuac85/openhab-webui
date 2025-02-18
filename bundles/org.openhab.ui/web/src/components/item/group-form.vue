<template>
  <div class="group-form no-padding">
    <!-- Type -->
    <f7-list-item v-if="item.type === 'Group'" :disabled="!editable" title="Members Base Type" class="align-popup-list-item" smart-select :smart-select-params="{searchbar: true, openIn: 'popup', closeOnSelect: true}">
      <select name="select-basetype" @change="groupType = $event.target.value">
        <option v-for="type in types.GroupTypes" :key="type" :value="type" :selected="item.groupType ? type === item.groupType.split(':')[0] : false">
          {{ type }}
        </option>
      </select>
    </f7-list-item>
    <!-- Dimension -->
    <f7-list-item v-if="dimensions.length && item.groupType && item.groupType.startsWith('Number')" :disabled="!editable" title="Dimension" class="align-popup-list-item" smart-select :smart-select-params="{searchbar: true, openIn: 'popup', closeOnSelect: true}">
      <select name="select-dimension" @change="groupDimension = $event.target.value">
        <option key="" value="Number" :selected="item.type === 'Number'" />
        <option v-for="d in dimensions" :key="d.name" :value="d.name" :selected="'Number:' + d.name === item.groupType">
          {{ d.label }}
        </option>
      </select>
    </f7-list-item>
    <!-- (Internal) Unit & State Description -->
    <template v-if="createMode && groupType && groupDimension">
      <f7-list-input label="Unit"
                     type="text"
                     info="Used internally, for persistence and external systems. It is independent from the state visualization in the UI, which is defined through the state description."
                     :value="item.unit"
                     @input="item.unit = $event.target.value" clear-button />
      <f7-list-input label="State Description Pattern"
                     type="text"
                     info="Pattern or transformation applied to the state for display purposes. Only saved if you change the pre-filled default value."
                     :value="item.stateDescriptionPattern"
                     @input="item.stateDescriptionPattern = $event.target.value" clear-button />
    </template>
    <!-- Aggregation Functions -->
    <f7-list-item v-if="aggregationFunctions" :disabled="!editable" title="Aggregation Function" class="align-popup-list-item" smart-select :smart-select-params="{openIn: 'popup', closeOnSelect: true}">
      <select name="select-function" @change="groupFunctionKey = $event.target.value">
        <option v-for="type in aggregationFunctions" :key="type.name" :value="type.name" :selected="type.name === groupFunctionKey">
          {{ type.value }}
        </option>
      </select>
    </f7-list-item>
  </div>
</template>

<style lang="stylus">
.group-form
  .item-inner
    padding-left 1em
  .list
    margin-top 0
    margin-bottom 0
</style>

<script>
import * as types from '@/assets/item-types.js'

import uomMixin from '@/components/item/uom-mixin'

export default {
  mixins: [uomMixin],
  props: ['item', 'createMode'],
  data () {
    return {
      types
    }
  },
  computed: {
    editable () {
      return this.createMode || (this.item && this.item.editable)
    },
    groupType: {
      get () {
        return this.item.groupType?.split(':')[0]
      },
      set (newType) {
        const previousAggregationFunctions = this.aggregationFunctions
        this.$set(this.item, 'groupType', '')
        this.$nextTick(() => {
          if (newType !== 'None') {
            this.$set(this.item, 'groupType', newType)
            if (previousAggregationFunctions !== this.aggregationFunctions) {
              this.$set(this.item, 'functionKey', 'None')
            }
          }
        })
      }
    },
    groupDimension: {
      get () {
        const parts = this.item.groupType.split(':')
        return parts.length > 1 ? parts[1] : ''
      },
      set (newDimension) {
        if (!newDimension) {
          this.groupType = 'Number'
          return
        }
        const dimension = this.dimensions.find((d) => d.name === newDimension)
        this.$set(this.item, 'groupType', 'Number:' + dimension.name)
        this.$set(this.item, 'unit', dimension.systemUnit)
        this.$set(this.item, 'stateDescriptionPattern', `%.0f ${dimension.systemUnit}`)
      }
    },
    groupFunctionKey: {
      get () {
        return this.item.functionKey
      },
      set (newFunctionKey) {
        if (!newFunctionKey) {
          delete this.item.function
          this.$set(this.item, 'functionKey', '')
          return
        }
        this.$set(this.item, 'functionKey', newFunctionKey)
        const parts = newFunctionKey.split('_')
        let func = {
          name: parts[0]
        }
        if (parts.length > 1) {
          func.params = [parts[1], parts[2]]
        }
        this.$set(this.item, 'function', func)
      }
    },
    aggregationFunctions () {
      switch (this.groupType) {
        case 'Dimmer':
        case 'Rollershutter':
        case 'Number':
          return types.ArithmeticFunctions
        case 'Contact':
          return types.LogicalOpenClosedFunctions
        case 'Player':
          return types.LogicalPlayPauseFunctions
        case 'DateTime':
          return types.DateTimeFunctions
        case 'Switch':
          return types.LogicalOnOffFunctions
      }
      return null
    }
  },
  beforeMount () {
    if (this.item.function) {
      this.$set(this.item, 'functionKey', this.item.function.name)
      if (this.item.function.params) {
        this.item.functionKey += '_' + this.item.function.params.join('_')
      }
    } else {
      this.$set(this.item, 'functionKey', '')
    }
  }
}
</script>
