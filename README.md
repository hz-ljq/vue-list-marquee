# vue-list-marquee

> 基于vue2的列表滚动插件
```
特点1：滚动方向，自下而上（目前，滚动方向不可自定义）；
特点2：当列表内容超过列表高度，自动滚动，否则，不滚动；
特点3：如果滚动，则首尾无缝拼接；
```

### NPM
```
npm install vue-list-marquee --save
```

### Mount
- mount with global
```js
import Vue from 'vue'
// require styles
import 'vue-list-marquee/dist/vue-list-marquee.css'
import VueListMarquee from 'vue-list-marquee'
Vue.use(VueListMarquee)
```

- mount with component
```js
// require styles
import 'vue-list-marquee/dist/vue-list-marquee.css'
import { VueListMarquee } from 'vue-list-marquee'
export default {
  components: {
    VueListMarquee
  }
}
```

### Usage
```js
<vue-list-marquee class='my-marquee' :listData='myList' :option='myOption'>
  <template slot-scope="{ item, index }">

    <!-- 每一条内容的结构 -->
    <div class="list-item">
      <div class='col1'>-{{index}}-</div>
      <div class='col2' :title="item.content">{{item.content}}</div>
    </div>

  </template>
</vue-list-marquee>
```

### Props
| Name | Type | Default | Description |
| ------ | ------ | ------ | ------ |
| :listData | Array | [] | <font size=2>列表内容数组 |
| :option | Object | moveTime: 1000,<br/>needRestTime: false,<br/>restTime: 2000,<br/>needHover: false | <font size=2>配置项 |

### :option（Detail explanation）
| Name | Type | Default | Description |
| ------ | ------ | ------ | ------ |
| moveTime | Number | <font size=1>1000（单位：ms） | <font size=1>滚动一个条目高度的过渡时间； |
| needRestTime | Boolean | false| <font size=1>每滚动一个条目，是否需要停顿；如果为false，restTime属性将无效； |
| restTime | Number | <font size=1>2000（单位：ms） | <font size=1>每滚动一个条目后的停顿时间(尽量大于100，否则效果不好)，needRestTime为true时，才有效； |
| needHover | Boolean | false | <font size=1>当鼠标移入和移出时，是否需要暂停和继续滚动；|

- - -

1
