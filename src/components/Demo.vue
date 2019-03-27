<template>
<div class="wrapper">

  <div class="demo">
    <div class='my-list-1'>
      <div class='my-list-title'>My List 1</div>
      <div class="process-bar" :style="{width: clockPercent + '%'}" />

      <vue-list-marquee class='my-marquee' :listData='myListData1' :option='marqueeOption1'>
        <template slot-scope="{ item, index }">
          <div class="item">
            <div class='col1' :class="{'first': index === 0}">-{{index+1}}-</div>
            <div class='col2' :title="item.content">{{item.content}}</div>
          </div>
        </template>
      </vue-list-marquee>
    </div>

    <div class='my-list-2'>
      <div class='my-list-title'>My List 2</div>
      <div class="process-bar" :style="{width: clockPercent + '%'}" />

      <vue-list-marquee class='my-marquee' :listData='myListData2' :option='marqueeOption2'>
        <template slot-scope="{ item, index }">
          <div class="item">
            <div class='col1' :class="{'first': index === 0}">-{{index+1}}-</div>
            <div class='col2' :title="item.content">{{item.content}}</div>
          </div>
        </template>
      </vue-list-marquee>
    </div>

    <div class='my-list-3'>
      <div class='my-list-title'>My List 3</div>
      <div class="process-bar" :style="{width: clockPercent + '%'}" />

      <vue-list-marquee class='my-marquee' :listData='myListData3' :option='marqueeOption3'>
        <template slot-scope="{ item, index }">
          <div class="item">
            <div class='col1' :class="{'first': index === 0}">-{{index+1}}-</div>
            <div class='col2' :title="item.content">{{item.content}}</div>
          </div>
        </template>
      </vue-list-marquee>
    </div>
  </div>

  <div class="options">
    <pre>{{marqueeOption1}}</pre>
    <pre>{{marqueeOption2}}</pre>
    <pre>{{marqueeOption3}}</pre>
  </div>

</div>
</template>

<script>
import './Demo.scss'

export default {
  name: 'Demo',
  data() {
    return {
      myListData1: [],
      myListData2: [],
      myListData3: [],

      listData1: [{
        content: 'todo1todo1todo1todo1todo1todo1todo1todo1'
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
      }],

      listData3: [{
        content: 'todo1todo1todo1todo1todo1todo1todo1todo1todo1todo1'
      }, {
        content: 'todo2todo2todo2todo2todo2'
      }, {
        content: 'todo3todo3todo3todo3todo3todo3todo3todo3'
      }, {
        content: 'todo4todo4todo4todo4todo4todo4todo4todo4todo4todo4todo4todo4todo4todo4'
      }, {
        content: 'todo5todo5'
      }, {
        content: 'todo6todo6todo6todo6todo6todo6todo6todo6todo6todo6'
      }, {
        content: 'todo7todo7todo7todo7todo7todo7todo7todo7todo7todo7todo7todo7todo7todo7todo7'
      }, {
        content: 'todo8todo8todo8todo8todo8todo8'
      }, {
        content: 'todo9todo9todo9'
      }, {
        content: 'todo10todo10todo10todo10todo10todo10todo10'
      }, {
        content: 'todo11todo11todo11todo11'
      }, {
        content: 'todo12todo12todo12todo12todo12todo12'
      }, {
        content: 'todo13todo13todo13todo13todo13todo13todo13'
      }, {
        content: 'todo14todo14todo14todo14todo14todo14todo14todo14todo14'
      }, {
        content: 'todo15todo15todo15todo15'
      }],

      marqueeOption1: {
        moveTime: 500,
        needRestTime: true,
        restTime: 500,
        needHover: true,
        delayTime: 1000
      },

      marqueeOption2: {
        moveTime: 1000,
        needRestTime: false,
        restTime: 1000,
        needHover: true,
        delayTime: 1000
      },

      marqueeOption3: {
        moveTime: 1200,
        needRestTime: true,
        restTime: 600,
        needHover: true,
        delayTime: 1000,
        timingFunc: 'ease-in-out'
      },

      dataRefreshTimer: null,
      dataRefreshFreq: 24000, // 数据刷新频率，每24000ms；

      clockTimer: null,
      clockPercent: 0,
      clockFreq: 500 // 数据刷新冷冻时间指示器，更新频率，每1000ms；
    }
  },
  computed: {
    option1() {
      return JSON.stringify(this.marqueeOption1, null, 4);
    }
  },
  methods: {
    getListData() {
      this.clockPercent = 0;
      clearInterval(this.clockTimer);
      this.clockTimer = setInterval(() => {
        this.clockPercent += (100 - this.clockPercent) * 0.12; // 数据刷新CD时间进度，每次累计剩余进度的12%；
      }, this.clockFreq);

      this.myListData1 = this.listData1.slice(0, +(9 + Math.random() * 6).toFixed(0));
      this.myListData2 = this.listData1.slice(0, +(9 + Math.random() * 6).toFixed(0));
      this.myListData3 = this.listData3.slice(0, +(9 + Math.random() * 6).toFixed(0));
      // this.myListData = this.listData.slice(0, 13);
      console.log(`-------------------split line-------------------`);
      console.log(`list data updated,【My List 1】 list number: ${this.myListData1.length}`);
      console.log(`list data updated,【My List 2】 list number: ${this.myListData2.length}`);
      console.log(`list data updated,【My List 3】 list number: ${this.myListData3.length}`);
    }
  },
  mounted() {
    this.$nextTick(() => {
      this.getListData();

      // 刷新数据；
      this.dataRefreshTimer = setInterval(() => {
        this.getListData();
      }, this.dataRefreshFreq);
    });
  },
  components: {

  },
  beforeDestroy() {
    clearInterval(this.dataRefreshTimer);
    clearInterval(this.clockTimer);
  }
}
</script>
