<template>
  <div class="a__popper_container">
    <div class="tips__target">
      <span :ref="`target_${refsId}`" :style="{ width: `${activeLength}px` }">
        <slot v-if="$slots.tipsTarget" name="tipsTarget"></slot>
        <span v-else>{{ tipsTargetText }}</span>
      </span>
    </div>
    <div
      class="tips__content"
      v-if="renderTipsContent"
      role="tooltip"
      :ref="`content_${refsId}`"
    >
      <template v-if="refsId === activeTipsId">
        <slot v-if="$slots.tipsContent" name="tipsContent"></slot>
        <div v-else>{{ tipsContent }}</div>
        <div
          class="arrow"
          :class="`arrow__${placement}`"
          data-popper-arrow
        ></div>
      </template>
    </div>
  </div>
</template>

<script>
import { createPopper } from "@popperjs/core";
import {createUuid} from "@/utils/index";
export default {
  name: "APopper",
  props: {
    tipsTargetText: {
      // 触发tips的目标文本
      type: String,
      default: "",
    },
    activeLength: {
      // 触发气泡时文本长度
      type: Number,
      default: 300,
    },
    autoCalcShow: {
      // 是否根据传入的activeLength计算长度触发气泡显示
      type: Boolean,
      default: true,
    },
    tipsContent: {
      // 气泡内显示的内容
      type: String,
      default: "",
    },
    placement: {
      // 气泡弹出的位置
      type: String,
      default: "right",
    },
    offset: {
      // 气泡弹出偏移量
      type: Array,
      default: () => [0, 10],
    },
    openDelay: {
      // 气泡延迟弹出的时间
      type: Number,
      default: 300,
    },
    fontStyle: {
      type: String,
      default: "14px Avenir, Helvetica, Arial, sans-serif",
    },
  },
  data() {
    return {
      refsId: createUuid(), // 创建tips实例的唯一标识
      activeTipsId: "", // 当前鼠标需要显示的气泡实例Id
      showTimmer: null, // 防抖计时器
      renderTipsContent: true, // 性能优化参数，对不满足触发tips的情况，不渲染空的tipsContent承载容器，减少初始化的dom渲染
    };
  },
  mounted() {
    this.calcShowTipsByLength();
    // 性能优化
    if (this.autoCalcShow && this.renderTipsContent) {
      this.init();
    }
  },
  methods: {
    init() {
      const targetRefs = this.$refs[`target_${this.refsId}`];
      const contentRefs = this.$refs[`content_${this.refsId}`];
      // tips初始化参数配置
      const tipsOpt = {
        placement: this.placement,
        modifiers: [
          {
            name: "offset", // 偏移量修饰符配置
            options: {
              offset: this.offset,
            },
          },
        ],
      };

      // 创建tips实例
      const instance = createPopper(targetRefs, contentRefs, tipsOpt);

      const eventListener = (isShow) => {
        // 清空计时器
        const clearTimmer = () => {
          clearTimeout(this.showTimmer);
          this.showTimmer = null;
        };
        clearTimmer();
        // 当鼠标移入的时候显示设置显示tips样式
        if (isShow) {
          // 根据pops中的openDelay延迟显示tips
          this.showTimmer = setTimeout(() => {
            contentRefs.setAttribute("data-show", "");
            this.activeTipsId = this.refsId; // 只显示鼠标移入的节点tips，减少初始时候的dom渲染
            isShow && instance.update(); // 更新tips显示时候的位置
          }, this.openDelay);
        } else {
          // hide tips气泡
          this.activeTipsId = "";
          contentRefs.removeAttribute("data-show", "");
        }
        instance.setOptions((opt) => ({
          ...opt,
          modifiers: [
            ...opt.modifiers,
            { name: "eventListeners", enabled: isShow },
          ],
        }));
      };

      // 手动监听tips鼠标事件
      const showEvents = ["mouseenter", "focus"];
      const hideEvents = ["mouseleave", "blur"];

      showEvents.concat(hideEvents).forEach((fn) => {
        targetRefs.addEventListener(fn, () =>
          eventListener(showEvents.includes(fn))
        );
      });
    },
    calcShowTipsByLength() {
      // 判断是否需要通过autoCalcShow长度计算出现气泡
      const targetRefs = this.$refs[`target_${this.refsId}`];
      const canvas = document.createElement("canvas");
      const context = canvas.getContext("2d");
      // 这里需要设置成和最终页面真实渲染的字体和字体大小
      context.font = this.fontStyle;
      const metrics = context.measureText(targetRefs.innerText);
      // 通过canvas计算文本渲染后真实宽度
      this.renderTipsContent = metrics.width > this.activeLength;
    },
  },
};
</script>

<style lang="scss" scoped>
.a__popper_container {
  .tips__content {
    display: none;
    background-color: #333;
    color: #fff;
    padding: 6px 12px;
    border-radius: 4px;
    z-index: 100;
    font-size: 12px;

    &[data-show] {
      display: block;
    }

    /**当出现滚动条时，隐藏tips */
    &[data-popper-reference-hidden] {
      visibility: hidden;
      pointer-events: none;
      .arrow__right::before {
        visibility: hidden;
      }
    }
  }

  /**添加箭头样式*/
  .arrow {
    visibility: hidden;
    &::before {
      position: absolute;
      width: 8px;
      height: 8px;
      background: #333;
    }
  }

  .arrow__right::before {
    visibility: visible;
    content: "";
    transform: rotate(45deg);
    top: 10px;
    left: -4px;
  }
  .tips__target {
    span {
      font-size: 14px;
      font-family: "Avenir, Helvetica, Arial, sans-serif";
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: nowrap;
      display: block;
      text-align: left;
    }
  }

  // .tips__content[data-popper-placement^="top"] > .arrow {
  //   bottom: -4px;
  // }
  // .tips__content[data-popper-placement^="bottom"] > .arrow {
  //   top: -4px;
  // }
  // .tips__content[data-popper-placement^="left"] > .arrow {
  //   right: -4px;
  // }
  // .tips__content[data-popper-placement^="right"] > .arrow {
  //   left: -4px;
  // }
}
</style>
