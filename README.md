## 基于element-ui下拉树组件
|接收参数|类型|说明|默认值|
|:----:|:----|----:|----:|
|v-model|String|绑定的数据|
|treeData|Array|树结构数组|
|props|Object|同element-ui的props|
|multiple|Boolean|是否多选| false
|uniqueKey|String|树节点的唯一值|
|showCheckbox|Boolean|是否显示checkbox|
|placeholder|String|占位符|

## 抛出的事件
|事件名|接收参数|说明|默认值|
|:----:|:----|----:|----:|
|getNode|id(需要指明uniqueKey)|返回对应data|
|checkedChange|无|返回同选中节点的四个对象同el-tree中的check事件中的第二个参数|

