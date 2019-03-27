<template>
<div class="vue-list-marquee-list">

  <!-- 列表dom -->
  <div class="list-items">
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
  <div class="list-items-copy" v-show="listCopyExistFlag">
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
// import './VueListMarquee.scss'

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

    // 来自用户的选项配置
    'option': {
      type: Object,
      default: () => {
        return {}
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

      listCopyExistFlag: false, // 列表副本dom的存在标志位；

      startTime: 0,
      currentItemIndex: -1,

      // marquee内部使用的选项配置
      innerOption: {

      }
    }
  },
  watch: {
    // 生命周期顺序为：beforeCreate -> props -> watch -> computed -> created，
    // 因此，对 option 属性的验证和赋初值放在watch里进行
    'option': {
      handler(newVal, oldVal) {
        this.optionValidateAndSetDefaultValue(); // 对没有定义的option属性进行默认值赋值；
      },
      deep: true,
      immediate: true // 该配置项必须为true，不然进入computed之前，option属性的值还没来得及验证和赋初值；
    },

    // list数据更新
    listData(newVal, oldVal) {
      this.listCopyExistFlag = false;
      this.$nextTick(() => {
        clearInterval(this.loopTimer);
        // let listDom = document.getElementById('list-items');
        // let listCopyDom = document.getElementById('list-items-copy');
        let listDom = this.$el.getElementsByClassName('list-items')[0];
        let listCopyDom = this.$el.getElementsByClassName('list-items-copy')[0];

        // 每次刷新数据时，数据运动到顶部的时间设置到0ms(使其瞬间归位)，不然会以每条item本身的运动速度置顶，太慢了；
        listDom.style.transition = `transform 0ms`;
        listCopyDom.style.transition = `transform 0ms`;

        // 这个$nextTick()是为了保证在transition成功设置成0ms之后，再对数据条目做归位处理；
        this.$nextTick(() => {
          this.currentItemIndex = -1;
          this.gap1 = 0;
          this.gap2 = 0;

          listDom.style.transform = `translateY(${this.gap1}px)`;
          listCopyDom.style.transform = `translateY(${this.gap2}px)`;

          // 滚动前，先根据delayTime做等待处理；
          setTimeout(() => {
            this.switchLoop('start');
          }, this.innerOption.delayTime);
        });
      })
    }
  },
  methods: {
    // 对$nextTick()进行封装，使其可以通过 async-await 的方式，在真实dom变化完之前，保持js阻塞状态；
    myNextTick() {
      return new Promise((resolve) => {
        this.$nextTick(() => {
          resolve();
        });
      })
    },

    // 定时器配合css3的transition过渡，实现每滚动一个条目就停顿一段时间的效果；
    // 优点：性能相对好一些，因为定时器时间间隔较长，transition性能较高；
    // 缺点：以单个条目的高度为滚动粒度，当鼠标移入暂时滚动时，能观察到这个问题；
    hasRestMode(indi) {
      if (!this.listData.length) return;

      let listDom = this.$el.getElementsByClassName('list-items')[0];
      let listCopyDom = this.$el.getElementsByClassName('list-items-copy')[0];
      let listBoxDom = this.$el;

      let boxHeight = listBoxDom.offsetHeight; // list容器的高度；
      let offsetHeightTotal = listDom.offsetHeight; // 获取所有条目的总高度；

      if (offsetHeightTotal >= boxHeight) { // 所有条目的总高度 大于或等于 list容器的高度，才轮播；
        this.listCopyExistFlag = true;
        listDom.style.transition = `transform ${this.innerOption.moveTime}ms ${this.innerOption.timingFunc}`;
        listCopyDom.style.transition = `transform ${this.innerOption.moveTime}ms ${this.innerOption.timingFunc}`; // TODO：运动函数做出可配置

        if (indi === 'start') {
          clearInterval(this.loopTimer);
          this.loopTimer = setInterval(async() => {
            this.currentItemIndex = (this.currentItemIndex + 1) % this.listData.length;
            let offsetHeightTotal = listDom.offsetHeight; // 重新获取所有条目的总高度，以保证数据的即使有效性；
            let offsetHeight = listDom.children[this.currentItemIndex].offsetHeight; // 重新获取单个条目的高度，以保证数据的即使有效性；

            listDom.style.opacity = 1;
            listCopyDom.style.opacity = 1;
            await this.myNextTick(); // 在真实dom变化完之前，先阻塞js；

            if (this.gap1 <= -offsetHeightTotal) {
              this.gap1 = offsetHeightTotal;
              this.gap2 = -offsetHeightTotal; // 避免js计算精度损失而导致的gap1和gap2变化率不一致的问题；
              listDom.style.opacity = 0; // 位置重置时先透明掉，否则，重置过程会在过渡效果中被观察到；

              await this.myNextTick(); // 在真实dom的opacity被改变为0之前，先阻塞js；
              listDom.style.transform = `translateY(${this.gap1}px)`;
            }

            if (this.gap2 <= -offsetHeightTotal * 2) {
              this.gap2 = 0;
              this.gap1 = 0; // 避免js计算精度损失而导致的gap1和gap2变化率不一致的问题；
              listCopyDom.style.opacity = 0; // 位置重置时先透明掉，否则，重置过程会在过渡效果中被观察到；

              await this.myNextTick(); // 在真实dom的opacity被改变为0之前，先阻塞js；
              listCopyDom.style.transform = `translateY(${this.gap2}px)`;
            }

            this.gap1 -= offsetHeight; // 每次滑动一个条目的高度；
            this.gap2 -= offsetHeight;

            listDom.style.transform = `translateY(${this.gap1}px)`;
            listCopyDom.style.transform = `translateY(${this.gap2}px)`;
          }, this.innerOption.moveTime + this.innerOption.restTime);
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

      let listDom = this.$el.getElementsByClassName('list-items')[0];
      let listCopyDom = this.$el.getElementsByClassName('list-items-copy')[0];
      let listBoxDom = this.$el;

      let boxHeight = listBoxDom.offsetHeight; // list容器的高度；
      let offsetHeightTotal = listDom.offsetHeight; // 获取所有条目的总高度；
      let offsetHeight = listDom.children[0].offsetHeight; // 获取第一个条目的高度；

      if (offsetHeightTotal >= boxHeight) { // 所有条目的总高度 大于或等于 list容器的高度，才轮播；
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

            this.gap1 -= offsetHeight / (this.innerOption.moveTime / timeGap); // 计算每次滚动的高度（像素）；
            this.gap2 -= offsetHeight / (this.innerOption.moveTime / timeGap);

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
      if (this.innerOption.needRestTime) {
        this.hasRestMode(indi);
      } else {
        this.hasNoRestMode(indi);
      }
    },

    // 对 innerOption 进行赋值（通过默认配置与 option 配置的合并）；
    optionValidateAndSetDefaultValue() {
      let defaultOption = {
        moveTime: 1000, // 滚动一个条目高度的过渡时间；
        needRestTime: true, // 每滚动一个条目，是否需要停顿；如果为false，restTime和timingFunc属性将无效；
        restTime: 2000, // 每滚动一个条目后的停顿时间(尽量大于100，否则效果不好)，当needRestTime为true时，才有效；
        needHover: true, // 当鼠标移入和移出时，是否需要暂停和继续滚动；
        delayTime: 3000, // 滚动前的延迟时间；
        timingFunc: 'linear' // 速度曲线，当needRestTime为true时，才有效；
      }
      // this.innerOption = Object.assign(defaultOption, this.option);
      this.innerOption = { ...defaultOption, ...this.option };

      if (this.innerOption.moveTime < 0) this.innerOption.moveTime = 1000;
      if (this.innerOption.restTime < 0) this.innerOption.restTime = 2000;
      if (this.innerOption.delayTime < 0) this.innerOption.delayTime = 3000;

      // if (!this.innerOption.hasOwnProperty('moveTime') || this.innerOption.moveTime < 0) {
      //   this.innerOption.moveTime = 1000;
      // }
      //
      // if (!this.innerOption.hasOwnProperty('needRestTime')) this.innerOption.needRestTime = false;
      //
      // if (this.innerOption.needRestTime) {
      //   if (!this.innerOption.hasOwnProperty('restTime') || this.innerOption.restTime < 0) {
      //     this.innerOption.restTime = 2000;
      //   }
      // }
      //
      // if (!this.innerOption.hasOwnProperty('needHover')) this.innerOption.needHover = true;
    }
  },
  created() {
    // // 对没有定义的属性进行默认值赋值；
    // if (!this.innerOption.hasOwnProperty('moveTime') || this.innerOption.moveTime < 0) {
    //   this.innerOption.moveTime = 1000;
    // }
    //
    // if (!this.innerOption.hasOwnProperty('needRestTime')) this.innerOption.needRestTime = false;
    //
    // if (this.innerOption.needRestTime) {
    //   if (!this.innerOption.hasOwnProperty('restTime') || this.innerOption.restTime < 0) {
    //     this.innerOption.restTime = 2000;
    //   }
    // }
    //
    // if (!this.innerOption.hasOwnProperty('needHover')) this.innerOption.needHover = true;
  },
  mounted() {
    this.$nextTick(() => {
      // console.log(this.innerOption);
      this.switchLoop('start');
    });
  },
  components: {},
  beforeDestroy() {
    clearInterval(this.loopTimer);
  }
}
</script>

<style lang="scss" src="./VueListMarquee.scss" scoped></style>
