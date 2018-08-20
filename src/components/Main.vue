<template>
<div id="main">

  <div class='yclb'>
    <div class='box-title'>实时异常列表</div>

    <vue-list-marquee class='my-marquee' :listData='yichangList' :option='marqueeOption'>
      <template slot-scope="{ item, index }">
        <div class="item">
          <div class='col1'>-{{index+1}}-</div>
          <div class='col2' :title="item.content">{{item.content}}</div>
        </div>
      </template>
    </vue-list-marquee>

  </div>

</div>
</template>

<script>
import './Main.scss'

export default {
  name: 'Main',
  data() {
    return {
      yichangList: [],
      listData: [{
        content: 'todo1todo1todo1todo1todo1todo1todo1todo1todo1todo1todo1todo1todo1todo1todo1todo1'
      }, {
        content: 'todo2'
      }, {
        content: 'todo3'
      }, {
        content: 'todo4'
      }, {
        content: 'todo5'
      }, {
        content: 'todo6'
      }, {
        content: 'todo7'
      }, {
        content: 'todo8'
      }, {
        content: 'todo9'
      }, {
        content: 'todo10'
      }, {
        content: 'todo11'
      }, {
        content: 'todo12'
      }, {
        content: 'todo13'
      }, {
        content: 'todo14'
      }, {
        content: 'todo15'
      }, {
        content: 'todo16'
      }, {
        content: 'todo17'
      }, {
        content: 'todo18'
      }, {
        content: 'todo19'
      }, {
        content: 'todo20'
      }, {
        content: 'todo21'
      }, {
        content: 'todo22'
      }, {
        content: 'todo23'
      }, {
        content: 'todo24'
      }],
      timer: -1,

      marqueeOption: {
        moveTime: 500,
        needRestTime: true,
        restTime: 500,
        needHover: true
      }
    }
  },
  methods: {
    getYichangListAndRender() {
      // this.yichangList = this.listData.slice(0, +(Math.random() * 26).toFixed(0));
      this.yichangList = this.listData.slice(0, 13);
      console.log(this.yichangList.length);
    }
  },
  mounted() {
    this.$nextTick(() => {
      this.getYichangListAndRender();

      this.timer = setInterval(() => {
        this.getYichangListAndRender();
      }, 8000);
    });
  },
  components: {

  },
  beforeDestroy() {
    clearInterval(this.timer);
  }
}
</script>
