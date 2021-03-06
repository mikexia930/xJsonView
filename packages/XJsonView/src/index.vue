<template>
  <div :class="[wrapClass]">
    <template v-if="isValidData">
      <node-html
        :node-data="formalData"
        :is-last-one="true"
        @emit="handleNodeSwitch"
      >
        <template slot='open'>
          <slot name="open">[+]</slot>
        </template>
        <template slot='close'>
          <slot name="close">[-]</slot>
        </template>
      </node-html>
    </template>
    <template v-else-if="isShowErrNotice">
      {{ errNotice }}
    </template>
  </div>
</template>

<script>
/**
 * json 具体条目组件
 */
const NodeItem = {
  name: 'node-item',
  props: {
    itemData: {
      type: Object,
    },
    isLastOne: {
      type: Boolean,
      default: false,
    },
  },
  computed: {
    valueTypeClass() {
      let colorClass = '';
      switch (typeof this.itemData.value) {
        case 'string':
          colorClass = 'stringColor';
          break;
        case 'number':
          colorClass = 'numberColor';
          break;
        case 'boolean':
          colorClass = 'booleanColor';
          break;
        default:
          colorClass = 'nullColor';
          break;
      }
      return colorClass;
    }
  },
  render() {
    return <p>
      <span>"{ this.itemData.name }"<span>:</span></span>
      <span class={ this.valueTypeClass }>
        { this.valueTypeClass === 'stringColor' ? `"${ String(this.itemData.value) }"` : String(this.itemData.value) }
        { this.isLastOne ? '' : <span>,</span> }
      </span>
    </p>
  },
}
/**
 * json 显示组件
 */
const NodeHtml = {
  name: 'node-html',
  components: {
    NodeItem,
  },
  props: {
    nodeData: {
      type: Object,
      default: () => {},
    },
    nodeType: {
      type: String,
      default: () => 'object',
    },
    isLastOne: {
      type: Boolean,
      default: false,
    }
  },
  computed: {
    nodeDataKeys() {
      return Object.keys(this.nodeData.child);
    }
  },
  methods: {
    handleSwitch(path) {
      this.$emit('emit', path);
    },
  },
  render() {
    return (
      <div key={this.nodeData.path}>
        <p>
          { this.nodeData.name && this.nodeType === 'object' ? <span>"{ this.nodeData.name }"<span>:</span></span> : ''}
          <span>
            <span class="node-switch" onClick={ () => this.handleSwitch(this.nodeData.path) }>{ this.nodeData.show ? this.$slots.close : this.$slots.open }</span>
            { this.nodeData.type === 'object' ? '{' : '['}
            { this.nodeData.show ? '' : `...${ this.nodeData.type === 'object' ? '}' : ']' }` }
          </span>
        </p>
        { !this.nodeData.show ? '' : <div class="node-list">
          {
            this.nodeDataKeys.map((itemKey, index) => {
              if (this.nodeData.child[itemKey].child) {
                return <node-html
                  nodeData={ this.nodeData.child[itemKey] }
                  nodeType={ this.nodeData.type }
                  isLastOne={ index + 1 === this.nodeDataKeys.length }
                  onEmit={ this.handleSwitch }
                >
                  <template slot='open'>
                    { this.$slots.open }
                  </template>
                  <template slot='close'>
                    { this.$slots.close }
                  </template>
                </node-html>
              } else {
                return <node-item
                  key={ this.nodeData.child[itemKey].path }
                  itemData={ this.nodeData.child[itemKey] }
                  isLastOne={ index + 1 === this.nodeDataKeys.length }
                />
              }
            })
          }
          </div>
        }
        { !this.nodeData.show ? '' : <p>
            <span>
              { this.nodeData.type === 'object' ? '}' : ']'}
              { this.isLastOne ? '' : <span>,</span> }
            </span>
          </p>
        }
      </div>
    );
  },
}

export default {
  name: 'x-json-view',
  components: {
    NodeHtml
  },
  props: {
    jsonData: String,
    isShowErrNotice: {
      type: Boolean,
      default: true,
    },
    errNotice: {
      type: String,
      default: 'json parse failure',
    },
    showDepth: {
      type: Number,
      default: 1,
    },
    wrapClass: {
      type: String,
      default: '',
    },
  },
  data() {
    return {
      isValidData: true,
      parseData: undefined,
      formalData: {
        show: this.showDepth < 0 ? false : true,
        name: '',
        path: '-1',
        depth: 0,
        type: '',
        child: {},
      },
      pointData: {
        key: '',
        name: '',
      },
      activeNodeData: [],
    };
  },
  watch: {
    jsonData(newValue) {
      console.log(newValue);
      this.formatJson();
    }
  },
  created() {
   this.formatJson();
  },
  methods: {
    formatJson() {
      this.parseJsonData();
      if (this.isValidData) {
        this.makeHtml();
      }
    },
    parseJsonData() {
      try {
        this.isValidData = true;
        this.parseData = JSON.parse(this.jsonData);
        this.formalData.type = Array.isArray(this.parseData) ? 'array' : 'object';
      } catch (err) {
        this.isValidData = false;
        this.emitTo('parseError', {
          jsonData: this.jsonData,
        });
      }
    },
    formatData() {
      const bubbleData = {};
      this.dataRecursion(this.parseData, '', 0, 0, bubbleData);
      if (this.$set) {
        this.$set(this.formalData, 'child', bubbleData);
      } else {
        this.formalData.child = bubbleData;
      }
      this.parseData = null;
    },
    dataRecursion(data, path, key, depth, lastData) {
      let keyCopy = key;
      const lastDataCopy = lastData;
      Object.keys(data).forEach((jsonKey) => {
        const curPath = path.toString() ? `${path}-${keyCopy}` : keyCopy;
        if (data[jsonKey] !== null && typeof data[jsonKey] === 'object') {
          lastDataCopy[keyCopy] = {
            show: depth < this.showDepth ? true : false,
            name: jsonKey,
            path: curPath,
            type: Array.isArray(data[jsonKey]) ? 'array' : 'object',
            depth,
            child: {},
          };
          this.dataRecursion(data[jsonKey], curPath, 0, depth + 1, lastDataCopy[keyCopy].child);
        } else {
          lastDataCopy[keyCopy] = {
            name: jsonKey,
            path: curPath,
            depth,
            value: data[jsonKey],
          };
        }
        keyCopy += 1;
      });
    },
    makeHtml() {
      this.formatData();
    },
    handleNodeSwitch(nodePath) {
      const path = nodePath.toString();
      if (path) {
        if (path === '-1') {
          this.formalData.show = !this.formalData.show;
        } else {
          const pathArr = path.split('-');
          let activeNodeData = this.formalData;
          pathArr.forEach((itemKey) => {
            activeNodeData = activeNodeData.child[itemKey];
          })
          if (activeNodeData) {
            activeNodeData.show = !activeNodeData.show;
          }
        }
      }
    },
    emitTo(type, data = '') {
      this.$emit('handleJsonView', {
        type,
        data,
      });
    },
  },
};
</script>
<style scoped>
  /deep/ .node-switch{
    cursor: pointer;
    font-size: 12px;
  }
  /deep/ .node-list{
    padding-left: 25px;
  }
  /deep/ div > p {
    padding: 0;
    margin: 0;
    height: 30px;
    line-height: 30px;
  }
  /deep/ div > p > span > span {
    display: inline-block;
    padding: 0 8px;
  }
  /deep/ .stringColor{
    color: #3ab54a;
  }
  /deep/ .numberColor{
    color: #25aae2;
  }
  /deep/ .booleanColor{
    color: #f98280;
  }
  /deep/ .nullColor{
    color: #f1592a;
  }
</style>
