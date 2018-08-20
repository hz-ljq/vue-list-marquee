<template>
<div id="vue-list-marquee-list">

  <!-- 列表dom -->
  <div class="list-items" id='list-items'>

    <template v-if="option.needHover">
      <div v-for='(item, index) in listData' :key='index' @mouseenter="switchLoop('stop')" @mouseleave="switchLoop('start')">
        <slot :item="item" :index="index"></slot>
      </div>
    </template>

    <template v-else>
      <div v-for='(item, index) in listData' :key='index'>
        <slot :item="item" :index="index"></slot>
      </div>
    </template>

  </div>

  <!-- 列表副本dom，用于首尾相连 -->
  <div class="list-items" id='list-items-copy' v-show="listCopyExistFlag">

    <template v-if="option.needHover">
      <div v-for='(item, index) in listData' :key='index' @mouseenter="switchLoop('stop')" @mouseleave="switchLoop('start')">
        <slot :item="item" :index="index"></slot>
      </div>
    </template>

    <template v-else>
      <div v-for='(item, index) in listData' :key='index'>
        <slot :item="item" :index="index"></slot>
      </div>
    </template>

  </div>

</div>
</template>

<script>
import './VueListMarquee.scss'

export default {
  name: 'VueListMarquee',
  // props: ['listData', 'option'],
  props: {
    'listData': { // 列表数据；
      type: Array,
      // required: true
      default: () => {
        return [];
      }
    },
    'option': {
      type: Object,
      default: () => {
        return {
          moveTime: 1000, // 滚动一个条目高度的过渡时间；
          needRestTime: false, // 每滚动一个条目，是否需要停顿；如果为false，restTime属性将无效；
          restTime: 2000, // 每滚动一个条目后的停顿时间(尽量大于100，否则效果不好)，needRestTime为true时，才有效；
          needHover: false // 当鼠标移入和移出时，是否需要暂停和继续滚动；
        }
      }
      // validator: (opt) => {
      //   return opt.moveTime >= 0 && opt.restTime >= 0;
      // }
    }
  },
  data() {
    return {
      loopTimer: -1, // 定时器id；
      gap1: 0, // 滑动距离（列表）；
      gap2: 0, // 滑动距离（列表副本）；

      listCopyExistFlag: true, // 列表副本dom的存在标志位；

      startTime: 0
    }
  },
  methods: {
    // 定时器配合css3的transition过渡，实现每滚动一个条目就停顿一段时间的效果；
    // 优点：性能相对好一些，因为定时器时间间隔较长，transition性能较高；
    // 缺点：以单个条目的高度为滚动粒度，当鼠标移入暂时滚动时，能观察到这个问题；
    hasRestMode(indi) {
      if (!this.listData.length) return;

      let listDom = document.getElementById('list-items');
      let listCopyDom = document.getElementById('list-items-copy');
      let listBoxDom = document.getElementById('vue-list-marquee-list');

      let boxHeight = listBoxDom.offsetHeight; // list容器的高度；
      let offsetHeightTotal = listDom.offsetHeight; // 获取所有条目的总高度；
      let offsetHeight = listDom.children[0].offsetHeight; // 获取单个条目的高度；

      if (offsetHeightTotal >= boxHeight) { // 所有条目的总高度 大于 list容器的高度，才轮播；
        this.listCopyExistFlag = true;
        listDom.style.transition = `transform ${this.option.moveTime}ms linear`;
        listCopyDom.style.transition = `transform ${this.option.moveTime}ms linear`; // TODO：运动函数做出可配置

        if (indi === 'start') {
          clearInterval(this.loopTimer);
          this.loopTimer = setInterval(() => {
            listDom.style.opacity = 1;
            listCopyDom.style.opacity = 1;

            if (this.gap1 <= -offsetHeightTotal) {
              this.gap1 = offsetHeightTotal;
              this.gap2 = -offsetHeightTotal; // 避免js计算精度损失而导致的gap1和gap2变化率不一致的问题；
              listDom.style.opacity = 0; // 位置重置时先透明掉，否则，重置过程会在过渡效果中被观察到；
              listDom.style.transform = `translateY(${this.gap1}px)`;
            }

            if (this.gap2 <= -offsetHeightTotal * 2) {
              this.gap2 = 0;
              this.gap1 = 0; // 避免js计算精度损失而导致的gap1和gap2变化率不一致的问题；
              listCopyDom.style.opacity = 0; // 位置重置时先透明掉，否则，重置过程会在过渡效果中被观察到；
              listCopyDom.style.transform = `translateY(${this.gap2}px)`;
            }

            this.gap1 -= offsetHeight; // 每次滑动一个条目的高度；
            this.gap2 -= offsetHeight;

            listDom.style.transform = `translateY(${this.gap1}px)`;
            listCopyDom.style.transform = `translateY(${this.gap2}px)`;
          }, this.option.moveTime + this.option.restTime);
        } else if (indi === 'stop') {
          clearInterval(this.loopTimer);
        }
      } else {
        this.listCopyExistFlag = false;
      }
    },

    // 通过定时器，实现滚动效果（不停顿）；
    // 优点：以像素作为滚动粒度，当鼠标移入暂时滚动时，能观察到这个优点；
    // 缺点：性能相对差一些，因为为了效果流畅，定时器时间间隔较短；
    hasNoRestMode(indi) {
      if (!this.listData.length) return;

      let listDom = document.getElementById('list-items');
      let listCopyDom = document.getElementById('list-items-copy');
      let listBoxDom = document.getElementById('vue-list-marquee-list');

      let boxHeight = listBoxDom.offsetHeight; // list容器的高度；
      let offsetHeightTotal = listDom.offsetHeight; // 获取所有条目的总高度；
      let offsetHeight = listDom.children[0].offsetHeight; // 获取单个条目的高度；

      if (offsetHeightTotal >= boxHeight) { // 所有条目的总高度 大于 list容器的高度，才轮播；
        this.listCopyExistFlag = true;

        if (indi === 'start') {
          this.startTime = +new Date();
          clearInterval(this.loopTimer);

          this.loopTimer = setInterval(() => {
            let endTime = +new Date();
            let timeGap = endTime - this.startTime;
            this.startTime = endTime;

            if (this.gap1 <= -offsetHeightTotal) {
              this.gap1 = offsetHeightTotal;
              this.gap2 = -offsetHeightTotal; // 避免js计算精度损失而导致的gap1和gap2变化率不一致的问题；
              listDom.style.transform = `translateY(${this.gap1}px)`;
            }

            if (this.gap2 <= -offsetHeightTotal * 2) {
              this.gap2 = 0;
              this.gap1 = 0; // 避免js计算精度损失而导致的gap1和gap2变化率不一致的问题；
              listCopyDom.style.transform = `translateY(${this.gap2}px)`;
            }

            this.gap1 -= offsetHeight / (this.option.moveTime / timeGap); // 计算每次滚动的高度（像素）；
            this.gap2 -= offsetHeight / (this.option.moveTime / timeGap);

            listDom.style.transform = `translateY(${this.gap1}px)`;
            listCopyDom.style.transform = `translateY(${this.gap2}px)`;
          }, 16); // 16ms很小，为了滚动效果的流畅，只能损失一些性能了；
        } else if (indi === 'stop') {
          clearInterval(this.loopTimer);
        }
      } else {
        this.listCopyExistFlag = false;
      }
    },

    switchLoop(indi) {
      if (this.option.needRestTime) {
        this.hasRestMode(indi);
      } else {
        this.hasNoRestMode(indi);
      }
    }
  },
  watch: {
    listData(newVal, oldVal) {
      this.$nextTick(() => {
        clearInterval(this.loopTimer);
        this.gap1 = 0;
        this.gap2 = 0;
        let listDom = document.getElementById('list-items');
        let listCopyDom = document.getElementById('list-items-copy');
        listDom.style.transform = `translateY(${this.gap1}px)`;
        listCopyDom.style.transform = `translateY(${this.gap2}px)`;
        this.switchLoop('start');
      })
    }
  },
  created() {
    // 对没有定义的属性进行默认值赋值；
    if (!this.option.hasOwnProperty('moveTime') || this.option.moveTime < 0) {
      this.option.moveTime = 1000;
    }

    if (!this.option.hasOwnProperty('needRestTime')) this.option.needRestTime = false;

    if (this.option.needRestTime) {
      if (!this.option.hasOwnProperty('restTime') || this.option.restTime < 0) {
        this.option.restTime = 2000;
      }
    }

    if (!this.option.hasOwnProperty('needHover')) this.option.needHover = true;
  },
  mounted() {
    this.$nextTick(() => {
      // console.log(this.option);
      this.switchLoop('start');
    });
  },
  components: {},
  beforeDestroy() {
    clearInterval(this.loopTimer);
  }
}
</script>
