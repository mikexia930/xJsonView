# x-json-view

### 版本
***
* v1.0.0

### 基于
***
* vue2
* recursion

### 安装
***
````
npm install xjsonview
````

### 使用
***
**在main.js中引入**
````
import xJsonView from '../x-json-view';
````

**组件中使用**
````
 <x-json-view
  :show-depth="1"
  :json-data="jsonData"
  :is-show-err-notice="false"
  @handleJsonView="handleJsonView"
>
  <template slot="open">+</template>
  <template slot="close">-</template>
</x-json-view>
````

### 说明
***
| 参数 | 类型 | 说明 |
| ------ | ------ | ------ |
| showDepth | number | 初始展开json的深度 -1:全部关闭，0：打开第一层，1：打开第二层 ... |
| jsonData | string | json格式的字符串
| isShowErrNotice | boolean | 是否显示默认报错信息，错误也会从子组件emit出来，type=parseError |
| handleJsonView | function | emit的接收方法 |
| open | slot | 打开的按钮slot |
| close | slot | 关闭的按钮slot |

