<style lang="scss">
  .icon-arrow-transition{
    transform: rotate(180deg);
    transition: all 0.3s;
  }
</style>
<template>
<div>
  <el-popover
    trigger="manual"
    placement="bottom-start"
    v-model="isShowTree"
    ref="popover"
    width="168"
    >
    <el-tree
      style="height:300px;overflow-y: scroll;"
      :data="data"
      :props="defaultProp"
      v-show="isShowTree"
      @node-click="handleNodeClick"
      node-key="id"
      :default-expanded-keys="defaultExpandKeys">
    </el-tree>
    <el-autocomplete
      slot="reference"
      class=""
      :fetchSuggestions="fetchSuggestion"
      @click.native="handleShowTree"
      :readonly="true"
      :value="selectedName">
      <i 
        :class="{'icon-arrow-transition': isShowTree}"
        class="el-input__icon el-icon-arrow-down"
        slot="suffix"></i>
    </el-autocomplete>
  </el-popover>
</div>
</template>

<script>
export default {
  props: {
    value: [ String, Number, Array ],
    multiple: {
      type: Boolean,
      defalut: false
    },
    data: Array,
    props: {
      type: Object,
      default: {
        label: 'label',
        children: 'children'
      }
    }
  },
  data () {
    return {
      defaultProp: {
        ...this.props
      },
      selectedData: {
        name: ''
      },
      isShowTree: false,
      defaultExpandKeys: []
    };
  },
  computed: {
    model: {
      get(){
        return this.value
      },
      set(val) {
        this.$emit('input', val);
      }
    },
    selectedName() {
      let name = '';
      if (this.multiple) {
        name = this.selectedData.name.join(',')
      } else {
        name = this.selectedData.name
      }
      return name;
    }
  },

  created () {
    if (this.multiple) {
      this.selectedData.name = []
      if (Array.isArray(this.value)) {
        this.model = this.value.slice();
      } else {
        if (this.value !== '') {
          this.model = [ this.value ];
        } else {
          this.model = [];
        }
      }
    };
  },
  mounted() {
    document.addEventListener('click', this.$refs.popover.handleDocumentClick)
  },
  beforeDestory() {
    document.removeEventListener('click', this.$refs.popover.handleDocumentClick)
  },

  methods: {
    fetchSuggestion(queryString, cb) {
      return cb([]);
    },
    handleShowTree() {
      if (!this.multiple) {
        const selectedId = this.model.toString();
        this.defaultExpandKeys =  [selectedId]
      } else {
        this.defaultExpandKeys = this.model.slice();
      }
      this.isShowTree = !this.isShowTree;
    },
    handleNodeClick(data, node) {
      if (node.isLeaf) {
        const { selectedData } = this;
        if (!this.multiple) {
          this.model = data.id;
          this.isShowTree = false;
          selectedData.name = data.name;
        } else {
          selectedData.name.push(data.name);
          this.model.push(data.id);
        }
      }
    }
  }
}
</script>