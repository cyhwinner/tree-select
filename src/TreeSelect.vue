<style lang="scss">
  .popover-custom{
    display: inline-block;
    width: 100%;
    .el-autocomplete{
      width: 100%;
      .el-input{
        &:hover{
          .el-icon-arrow-down{
            display: none;
          }
          .el-input__suffix-inner{
            .el-icon-circle-close{
              display: block;
            }
          }
        }
        .el-icon-circle-close{
          display: none;
          cursor: pointer;
        }
      }
        
    }
      
    .icon-arrow-transition{
      transform: rotate(180deg);
      transition: all 0.3s;
    }
    .el-input__inner{
      height: 36px;
      color: #1f2d3d;
      padding-left: 10px;
    }
  }
   
</style>
<template>
<div>
  <el-popover
    trigger="manual"
    placement="bottom-start"
    v-model="isShowTree"
    ref="popover"
    class="popover-custom"
    :width="popoverWidth"
    >
    <el-tree
      ref="tree"
      :props="props"
      :data="treeData"
      style="height:300px;overflow-y: scroll;"
      v-if="isShowTree"
      @node-click="handleNodeClick"
      :node-key="uniqueKey"
      :default-expanded-keys="defaultExpandKeys"
      :show-checkbox="showCheckbox"
      @check="handleChecked"
      v-bind="$attrs">
    </el-tree>
    <el-autocomplete
      ref="input"
      slot="reference"
      :fetchSuggestions="fetchSuggestion"
      @click.native="handleShowTree"
      :readonly="true"
      :value="selectedName"
      :placeholder="placeholder"
      >
      <i 
        :class="{'icon-arrow-transition': isShowTree}"
        class="el-input__icon el-icon-arrow-down"
        slot="suffix"></i>
      <i 
        class="el-icon-circle-close el-input__icon "
        slot="suffix"
        @click.stop="clearValue"></i>
    </el-autocomplete>
  </el-popover>
</div>
</template>

<script>
import { flatTreeToMap } from './eval'; 
export default {
  props: {
    value: [String, Number, Array],
    treeData: Array,
    showCheckbox: {
      type: Boolean,
      default: false
    },
    uniqueKey: {
      type: String,
      default: 'id'
    },
    props: {
      type: Object,
      default() {
        return {
          label: 'name',
          children: 'children'
        }
      }
    },
    placeholder: String
  },
  data () {
    return {
      selectedData: {
        name: ''
      },
      isShowTree: false,
      defaultExpandKeys: [],
      popoverWidth: '',
      dataList: []
    };
  },
  computed: {
    model: {
      get() {
        return this.value
      },
      set(val) {
        this.$emit('input', val);
      }
    },
    selectedName() {
      const selectedName = this.selectedData.name;
      return this.showCheckbox
        ? selectedName
          ? selectedName.join(',')
          : ''
        : selectedName;
    },
    nodeMap() {
      return flatTreeToMap(this.treeData, {}, this.uniqueKey, this.props.children)
    }
  },
  watch: {
    value(cur, pre) {
      if (cur === '') {
        this.selectedData.name = this.showCheckbox ? [] : '';
        return;
      }
      this.selectedData.name = this.showCheckbox
        ? cur.map((id) => this.nodeMap[id].name)
        : this.nodeMap[cur]
          ? this.nodeMap[cur].name
          : ''
    },
    isShowTree(cur) {
      if (cur) {
        this.init();
      }
    }
  },
  created () {
    this.init();
  },
  mounted() {
    document.addEventListener('click', this.$refs.popover.handleDocumentClick);
  },
  beforeDestory() {
    document.removeEventListener('click', this.$refs.popover.handleDocumentClick)
  },

  methods: {
    init() {
      if (this.showCheckbox) {
        this.model = Array.isArray(this.value)
          ? this.value.slice()
          : this.value !== ''
            ? [this.value]
            : []
        this.selectedData.name = this.selectedData.name ? this.selectedData.name : [];
      }
    },
    fetchSuggestion(queryString, cb) {
      return cb([]);
    },
    handleShowTree() {
      this.isShowTree = !this.isShowTree;
      !this.popoverWidth && (this.popoverWidth = this.$refs.input.$children[0].$el.getBoundingClientRect().width);
      this.$nextTick(() => {
        if (this.showCheckbox) {
          this.$refs.tree.setCheckedKeys(this.model);
        } else {
          this.defaultExpandKeys = this.showCheckbox
            ? this.model.slice()
            : this.model
              ? [this.model.toString()]
              : []
        }
      })
    },
    handleNodeClick(data, node) {
      const { selectedData, uniqueKey, props } = this;
      if (!this.showCheckbox) {
        this.model = data[uniqueKey];
        this.isShowTree = false;
        selectedData.name = data[props.label];
      } else {
        if (!this.showCheckbox) {
          selectedData.name.push(data[props.label]);
          this.model.push(data[uniqueKey]);
        }
      }
    },
    getNode(id) {
      return this.nodeMap[id];
    },
    clearValue() {
      this.selectedData.name = '';
      this.model = '';
      this.showCheckbox && this.isShowTree && this.$refs.tree.setCheckedKeys([]);
    },
    handleChecked(checkedNodes, checkedObject) {
      this.model = checkedObject.checkedKeys;
      this.$emit('checkedChange', checkedObject)
    }
  }
}
</script>