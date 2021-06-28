
## nigo-vue-drag v1.1.0
`    npm install nigo-vue-drag
`

`    yarn install nigo-vue-drag
`
## 使用示例
`    import Tab from 'nigo-vue-drag'
`
```
  <drag
        mode="list"
        :drag-ary="[{color: '',html:''}]"
        drag-height="100"
        box-width="500"
        @dragMouseup="dragMouseup"
         >
 
         </drag>

```
## props
| 参数 | 说明 | 类型 | 默认值 |
| :---| :--- | :--- | :--- |
| box-width | 盒子宽度 | String | auto |
| drag-height | 拖拽元素高度 | String | 100 |
| drag-mode | 是插入模式还是交换位置 | Number | 0 |
| drag-ary | 传入拖拽的数组,object字段可以添加color 和html | Array | 单元格 |
| dragMouseup | 返回拖拽的当前和目标索引值 | Events | 单元格 |


## 更新
1.增加mode  flex| list

2.增加返回拖拽的当前和目标索引值的事件dragMouseup	

  
 ### Keywords
vue 、JavaScript 、drag
