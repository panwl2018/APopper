<template>
  <div class="select__container" :style="{ width: `${width}px` }">
    <input v-model="text" placeholder="--请选择--" :id="`input__${refsId}`" />
    <div
      class="dropdown__wrap"
      :id="`list__${refsId}`"
      :style="{ minWidth: `${width}px` }"
    >
      <div class="arrow arrow__top" data-popper-arrow></div>
      <ul>
        <li>list1</li>
        <li>list2</li>
        <li>list3</li>
        <li>list4</li>
        <li>list5</li>
      </ul>
    </div>
  </div>
</template>

<script>
import { createPopper } from "@popperjs/core";
import {createUuid} from "@/utils/index";
export default {
  props: {
    width: {
      type: Number,
      default: 240,
    },
    size: {
      // 控件大小，medium/small/mini
      type: String,
      default: "medium",
    },
  },
  name: "ASelect",
  data() {
    return {
      text: "",
      refsId: createUuid(),
    };
  },
  mounted() {
    this.init();
  },
  methods: {
    init() {
      const selectInput = document.querySelector(`#input__${this.refsId}`);
      const selectList = document.querySelector(`.dropdown__wrap`);
      const opts = {
        placement: "bottom",
        modifiers: [
          {
            name: "offset",
            options: {
              offset: [0, 2],
            },
          },
          {
            name: "preventOverflow",
            options: {
              padding: 8,
            },
          },
          {
            name: "flip",
            options: {
              fallbackPlacements: ["top"],
            },
          },
        ],
      };
      const instance = createPopper(selectInput, selectList, opts);

      const hiderHandler = () => {
        selectList.style.display = "none";
      };

      const close = () => {
        selectList.className = "select__list_hidden";
        selectList.addEventListener("webkitAnimationEnd", hiderHandler);
        window.removeEventListener("click", close);
      };

      const open = (event) => {
        selectList.style.display = "block";
        selectList.className = "select__list_show";
        event.preventDefault();
        event.stopPropagation();
        window.addEventListener("click", close);
      };

      selectInput.addEventListener("click", (e) => {
        selectList.removeEventListener("webkitAnimationEnd", hiderHandler);
        if (selectList.style.display === "block") {
          close();
        } else {
          open(e);
          instance.update();
        }
      });

      selectList.addEventListener("click", (e) => {
        this.selectInput = e.target.textContent;
        close();
        instance.update();
      });
    },
  },
};
</script>


<style lang="scss" scoped>
.select__container {
  input {
    width: 100%;
  }
  .dropdown__wrap {
    display: none;
  }
  .select__list_hidden {
    animation: s2 0.25s;
  }
  .select__list_show {
    opacity: 1;
    animation: s1 0.45s;
  }
  ul {
    padding: 0;
    list-style: none;
    border: #ccc solid thin;
    text-align: left;
  }

  @keyframes s1 {
    from {
      opacity: 0;
    }
    to {
      opacity: 1;
    }
  }

  @keyframes s2 {
    from {
      opacity: 1;
    }
    to {
      opacity: 0;
    }
  }

  /**添加剪头样式*/
  .arrow {
    border-width: 6px;
    border-bottom-color: #ebeef5;
    border-top-width: 0px;

    &::before {
      // position: absolute;
      // width: 8px;
      // height: 8px;
      // // background: #333;
      // border: 1px solid #ebeef5;
      // border-bottom: none;
      // border-right: none;
    }
  }

  .arrow__top::before {
    visibility: visible;
    content: "";
    transform: rotate(45deg);
    top: 12px;
    left: -80px;
  }
}
</style>
