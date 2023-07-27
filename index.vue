<template>
  <div class="puzzle-container" v-show="verificationShow">
    <div class="puzzle-header">
      <span class="puzzle-header-left">拖动下方滑块完成拼图</span>
      <div>
        <span class="refrech" @click="refreshImg">
          <svg t="1690277209042" class="icon" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg"
            p-id="1462" xmlns:xlink="http://www.w3.org/1999/xlink" width="200" height="200">
            <path
              d="M929.536 208c-17.664 0-32 14.336-32 32l0 45.024C819.712 152.896 676.448 64 512 64 264.576 64 64 264.576 64 512s200.576 448 448 448c149.728 0 281.6-73.952 362.88-186.816 0.256-0.256 0.416-0.544 0.64-0.8 0.992-1.408 2.208-2.656 3.2-4.064-0.128-0.064-0.192-0.16-0.32-0.224 3.52-5.184 6.08-11.072 6.08-17.792 0.032-17.856-14.432-32.288-32.224-32.288-12.608 0-23.264 7.424-28.576 17.984l-0.224 0c-69.728 96.768-183.072 160-311.456 160C299.936 896 128 724.064 128 512S299.936 128 512 128c148.544 0 276.832 84.608 340.672 208l-51.136 0c-17.664 0-32 14.336-32 32s14.336 32 32 32l128 0c17.664 0 32-14.336 32-32l0-128C961.536 222.336 947.2 208 929.536 208z"
              fill="#272636" p-id="1463"></path>
          </svg>
        </span>
      </div>
    </div>
    <div :style="'position:relative;overflow:hidden;width:' + dataWidth + 'px;'">
      <div :style="'position:relative;width:' + dataWidth + 'px;height:' + dataHeight + 'px;'">
        <img id="scream" ref="scream" :src="imgRandom"
          :style="'width:' + dataWidth + 'px;height:' + dataHeight + 'px;'" />
        <canvas id="puzzle-box" ref="puzzleBox" :width="dataWidth" :height="dataHeight"></canvas>
      </div>
      <div class="puzzle-lost-box"
        :style="'left:' + left_Num + 'px;width:' + dataWidth + 'px;height:' + dataHeight + 'px;'">
        <canvas id="puzzle-shadow" ref="puzzleShadow" :width="dataWidth" :height="dataHeight"></canvas>
        <canvas id="puzzle-lost" ref="puzzleLost" :width="dataWidth" :height="dataHeight"></canvas>
      </div>
      <p :class="'ver-tips' + (displayTips ? ' slider-tips' : '')" ref="verTips">
        <template v-if="verification">
          <i style="background-position:-4px -1207px;"></i>
          <span style="color:#42ca6b;">验证通过</span>
          <span></span>
        </template>
        <template v-if="!verification">
          <i style="background-position:-4px -1229px;"></i>
          <span style="color:red;">验证失败:</span>
          <span style="margin-left:4px;">拖动滑块将悬浮图像正确拼合</span>
        </template>
      </p>
    </div>

    <div class="slider-container" :style="'width:' + dataWidth + 'px;'">
      <div class="slider-bar"></div>
      <div class="slider-btn" ref="sliderBtn" @mousedown="startMove" @touchstart="startMove"></div>
    </div>
  </div>
</template>

<script>
import { defineComponent, onMounted, toRefs, ref, reactive, unref, nextTick, watch } from 'vue'
export default defineComponent({
  props: {
    // 画布图片的尺寸
    width: {
      type: [String, Number],
      default: 260
    },
    height: {
      type: [String, Number],
      default: 120
    },
    // 图集
    puzzleImgList: {
      type: Array,
      default: () => [
        require('@/assets/puzzleImgList/thumbnail-img02.jpg')
      ]
    },
    // 滑块的大小
    blockSize: {
      type: [String, Number],
      default: 40
    },
    // 误差
    deviation: {
      type: [String, Number],
      default: 4
    },
    // 滑块的圆角大小
    blockRadius: {
      type: [String, Number],
      default: 4
    },
    // 滑块随机出现的范围
    wraperPadding: {
      type: [String, Number],
      default: 20
    },
    // 滑块形状 square  puzzle
    blockType: {
      type: String,
      default: 'square'
    },
    // 成功的回调
    onSuccess: {
      type: Function,
      default: () => {}
    },
    // 失败的回调
    onError: {
      type: Function,
      default: () => {}
    },
    verificationShow: {
      type: Boolean,
      default: false
    }
  },
  setup(props, ctx) {
    const puzzleBox = ref()
    const puzzleLost = ref()
    const puzzleShadow = ref()
    const sliderBtn = ref()
    const state = reactive({
      moveStart: "",
      displayTips: false,
      verification: false,
      randomX: null,
      randomY: null,
      imgRandom: "",
      left_Num: 0,
      dataWidth: null,
      dataHeight: null,
      puzzleSize: null, // 滑块的大小
      deviationValue: null,
      radius: null,
      padding: null
    })

    onMounted(() => {
      // 随机显示一张图片
      let imgRandomIndex = Math.round(
        Math.random() * (props.puzzleImgList.length - 1)
      );
      state.imgRandom = props.puzzleImgList[imgRandomIndex];

      state.puzzleSize = Number(props.blockSize);
      state.deviationValue = Number(props.deviation);
      state.radius = Number(props.blockRadius);
      state.dataWidth = Number(props.width);
      state.dataHeight = Number(props.height);
      state.padding = Number(props.wraperPadding);
      nextTick(() => {
        initCanvas();
      });
    })

    /* 刷新 */
    const refreshImg = () => {
      let imgRandomIndex = Math.round(
        Math.random() * (props.puzzleImgList.length - 1)
      );
      state.imgRandom = props.puzzleImgList[imgRandomIndex];
      initCanvas();
    }
    /* 画布初始化 */
    const initCanvas = () => {
      clearCanvas();
      let w = state.dataWidth;
      let h = state.dataHeight;
      let PL_Size = state.puzzleSize;
      let padding = state.padding;
      let MinN_X = padding + PL_Size;
      let MaxN_X = w - padding - PL_Size - PL_Size / 6;
      let MaxN_Y = padding;
      let MinN_Y = h - padding - PL_Size - PL_Size / 6;
      state.randomX = Math.round(Math.random() * (MaxN_X - PL_Size) + MinN_X);
      state.randomY = Math.round(Math.random() * MaxN_Y + MinN_Y);
      let X = state.randomX;
      let Y = state.randomY;
      state.left_Num = -X + 10;
      let d = PL_Size / 3;
      let radius = Number(state.radius);

      let c = puzzleBox;
      let c_l = puzzleLost;
      let c_s = puzzleShadow;
      let ctx = c.value.getContext("2d");
      let ctx_l = c_l.value.getContext("2d");
      let ctx_s = c_s.value.getContext("2d");
      ctx.globalCompositeOperation = "xor";
      ctx.shadowBlur = 10;
      ctx.shadowColor = "#fff";
      ctx.shadowOffsetX = 3;
      ctx.shadowOffsetY = 3;
      ctx.fillStyle = "rgba(0,0,0,0.7)";
      ctx.beginPath();
      ctx.lineWidth = "1";
      ctx.strokeStyle = "rgba(0,0,0,0)";
      if (state.blockType === 'square') {
        ctx.arc(X + radius, Y + radius, radius, Math.PI, (Math.PI * 3) / 2);
        ctx.lineTo(PL_Size - radius + X, Y);
        ctx.arc(
          PL_Size - radius + X,
          radius + Y,
          radius,
          (Math.PI * 3) / 2,
          Math.PI * 2
        );
        ctx.lineTo(PL_Size + X, PL_Size + Y - radius);
        ctx.arc(
          PL_Size - radius + X,
          PL_Size - radius + Y,
          radius,
          0,
          (Math.PI * 1) / 2
        );
        ctx.lineTo(radius + X, PL_Size + Y);
        ctx.arc(
          radius + X,
          PL_Size - radius + Y,
          radius,
          (Math.PI * 1) / 2,
          Math.PI
        );
      } else {
        ctx.moveTo(X, Y)
        ctx.lineTo(X + d, Y)
        ctx.bezierCurveTo(X + d, Y - d, X + 2 * d, Y - d, X + 2 * d, Y)
        ctx.lineTo(X + 3 * d, Y)
        ctx.lineTo(X + 3 * d, Y + d)
        ctx.bezierCurveTo(X + 2 * d, Y + d, X + 2 * d, Y + 2 * d, X + 3 * d, Y + 2 * d)
        ctx.lineTo(X + 3 * d, Y + 3 * d)
        ctx.lineTo(X, Y + 3 * d)
      }
      ctx.closePath();
      ctx.stroke();
      ctx.fill();

      let img = new Image();
      img.src = state.imgRandom;

      img.onload = function () {
        ctx_l.drawImage(img, 0, 0, w, h);
      };
      ctx_l.beginPath();
      ctx_l.strokeStyle = "rgba(0,0,0,0)";
      if (state.blockType === 'square') {
        ctx_l.arc(X + radius, Y + radius, radius, Math.PI, (Math.PI * 3) / 2);
        ctx_l.lineTo(PL_Size - radius + X, Y);
        ctx_l.arc(
          PL_Size - radius + X,
          radius + Y,
          radius,
          (Math.PI * 3) / 2,
          Math.PI * 2
        );
        ctx_l.lineTo(PL_Size + X, PL_Size + Y - radius);
        ctx_l.arc(
          PL_Size - radius + X,
          PL_Size - radius + Y,
          radius,
          0,
          (Math.PI * 1) / 2
        );
        ctx_l.lineTo(radius + X, PL_Size + Y);
        ctx_l.arc(
          radius + X,
          PL_Size - radius + Y,
          radius,
          (Math.PI * 1) / 2,
          Math.PI
        );
      } else {
        ctx_l.moveTo(X, Y)
        ctx_l.lineTo(X + d, Y)
        ctx_l.bezierCurveTo(X + d, Y - d, X + 2 * d, Y - d, X + 2 * d, Y)
        ctx_l.lineTo(X + 3 * d, Y)
        ctx_l.lineTo(X + 3 * d, Y + d)
        ctx_l.bezierCurveTo(X + 2 * d, Y + d, X + 2 * d, Y + 2 * d, X + 3 * d, Y + 2 * d)
        ctx_l.lineTo(X + 3 * d, Y + 3 * d)
        ctx_l.lineTo(X, Y + 3 * d)
      }
      ctx_l.closePath();
      ctx_l.stroke();
      ctx_l.shadowBlur = 10;
      ctx_l.shadowColor = "black";
      ctx_l.clip();
      ctx_s.beginPath();
      ctx_s.lineWidth = "1";
      ctx_s.strokeStyle = "rgba(0,0,0,0)";
      if (state.blockType === 'square') {
        ctx_s.arc(X + radius, Y + radius, radius, Math.PI, (Math.PI * 3) / 2);
        ctx_s.lineTo(PL_Size - radius + X, Y);
        ctx_s.arc(
          PL_Size - radius + X,
          radius + Y,
          radius,
          (Math.PI * 3) / 2,
          Math.PI * 2
        );
        ctx_s.lineTo(PL_Size + X, PL_Size + Y - radius);
        ctx_s.arc(
          PL_Size - radius + X,
          PL_Size - radius + Y,
          radius,
          0,
          (Math.PI * 1) / 2
        );
        ctx_s.lineTo(radius + X, PL_Size + Y);
        ctx_s.arc(
          radius + X,
          PL_Size - radius + Y,
          radius,
          (Math.PI * 1) / 2,
          Math.PI
        );
      } else {
        ctx_s.moveTo(X, Y)
        ctx_s.lineTo(X + d, Y)
        ctx_s.bezierCurveTo(X + d, Y - d, X + 2 * d, Y - d, X + 2 * d, Y)
        ctx_s.lineTo(X + 3 * d, Y)
        ctx_s.lineTo(X + 3 * d, Y + d)
        ctx_s.bezierCurveTo(X + 2 * d, Y + d, X + 2 * d, Y + 2 * d, X + 3 * d, Y + 2 * d)
        ctx_s.lineTo(X + 3 * d, Y + 3 * d)
        ctx_s.lineTo(X, Y + 3 * d)
      }
      ctx_s.closePath();
      ctx_s.stroke();
      ctx_s.shadowBlur = 20;
      ctx_s.shadowColor = "black";
      ctx_s.fill();
    }
    /* 通过重置画布尺寸清空画布，这种方式更彻底 */
    const clearCanvas = () => {
      let c = puzzleBox;
      let c_l = puzzleLost;
      let c_s = puzzleShadow;
      c.value.setAttribute("height", c.value.getAttribute("height"));
      c_l.value.setAttribute("height", c.value.getAttribute("height"));
      c_s.value.setAttribute("height", c.value.getAttribute("height"));
    }
    /* 按住滑块后初始化移动监听，记录初始位置 */
    const startMove = (e) => {
      // console.log(e);
      e = e || window.event;
      sliderBtn.value.style.backgroundPosition = "0 -216px";
      state.moveStart = e.pageX || e.targetTouches[0].pageX;
      addMouseMoveListener();
    }
    /* 滑块移动 */
    const moving = (e) => {
      e = e || window.event;
      let moveX = e.pageX || e.targetTouches[0].pageX;
      let d = moveX - state.moveStart;
      let w = state.dataWidth;
      let PL_Size = state.puzzleSize;
      let padding = state.padding;
      if (state.moveStart === "") {
        return "";
      }
      if (d < 0 || d > w - padding - PL_Size) {
        return "";
      }
      sliderBtn.value.style.left = d + "px";
      sliderBtn.value.style.transition = "inherit";
      puzzleLost.value.style.left = d + "px";
      puzzleLost.value.style.transition = "inherit";
      puzzleShadow.value.style.left = d + "px";
      puzzleShadow.value.style.transition = "inherit";
    }
    /* 移动结束，验证并回调 */
    const moveEnd = (e) => {
      e = e || window.event;
      let moveEnd_X = (e.pageX || e.changedTouches[0].pageX) - state.moveStart;
      let ver_Num = state.randomX - 10;
      let deviationValue = state.deviationValue;
      let Min_left = ver_Num - deviationValue;
      let Max_left = ver_Num + deviationValue;
      if (state.moveStart !== "") {
        if (Max_left > moveEnd_X && moveEnd_X > Min_left) {
          state.displayTips = true;
          state.verification = true;
          setTimeout(function () {
            state.displayTips = false;
            initCanvas();
            /* 成功的回调函数 */
            props.onSuccess();
          }, 500);
        } else {
          state.displayTips = true;
          state.verification = false;
          setTimeout(function () {
            state.displayTips = false;
            initCanvas();
            /* 失败的回调函数 */
            props.onError();
          }, 800);
        }
      }
      if (
        typeof sliderBtn.value !== "undefined" &&
        typeof puzzleLost.value !== "undefined" &&
        typeof puzzleShadow.value !== "undefined"
      ) {
        setTimeout(function () {
          sliderBtn.value.style.left = 0;
          sliderBtn.value.style.transition = "left 0.5s";
          puzzleLost.value.style.left = 0;
          puzzleLost.value.style.transition = "left 0.5s";
          puzzleShadow.value.style.left = 0;
          puzzleShadow.value.style.transition = "left 0.5s";
        }, 400);
        sliderBtn.value.style.backgroundPosition = "0 -84px";
      }
      state.moveStart = "";
    }
    /* 全局绑定滑块移动与滑动结束，移动过程中鼠标可在页面任何位置 */
    const addMouseMoveListener = () => {
      document.addEventListener("mousemove", moving);
      document.addEventListener("touchmove", moving);
      document.addEventListener("mouseup", moveEnd);
      document.addEventListener("touchend", moveEnd);
    }
    return {
      ...toRefs(state), addMouseMoveListener, moveEnd, moving, startMove, clearCanvas, initCanvas, refreshImg,
      puzzleBox, puzzleLost, puzzleShadow, sliderBtn
    }
  }
})
</script>

<style scoped>
.slider-btn {
  position: absolute;
  width: 44px;
  height: 44px;
  left: 0;
  top: -7px;
  z-index: 12;
  cursor: pointer;
  background-image: url(@/assets/sprite.3.2.0.png);
  background-position: 0 -84px;
  transition: inherit;
}

.ver-tips {
  position: absolute;
  left: 0;
  bottom: -22px;
  background: rgba(255, 255, 255, 0.9);
  height: 22px;
  line-height: 22px;
  font-size: 12px;
  width: 100%;
  margin: 0;
  text-align: left;
  padding: 0 8px;
  transition: all 0.4s;
}

.slider-tips {
  bottom: 0;
}

.ver-tips i {
  display: inline-block;
  width: 22px;
  height: 22px;
  vertical-align: top;
  background-image: url(@/assets/sprite.3.2.0.png);
  background-position: -4px -1229px;
}

.ver-tips span {
  display: inline-block;
  vertical-align: top;
  line-height: 22px;
  color: #455;
}

.active-tips {
  display: block;
}

.hidden {
  display: none;
}

.puzzle-container {
  position: relative;
  display: inline-block;
  padding: 15px 15px 28px;
  border: 1px solid #ddd;
  background: #ffffff;
  border-radius: 16px;
}

.puzzle-header {
  display: flex;
  justify-content: space-between;
  margin: 5px 0;
}

.puzzle-header-left {
  color: #333;
}

.re-btn,
.close-btn {
  font-size: 16px;
  cursor: pointer;
  color: #666;
}

.re-btn:hover {
  color: #67c23a;
}

.close-btn:hover {
  color: #f56c6c;
}

.close-btn {
  margin-left: 5px;
}

.slider-container {
  position: relative;
  margin: 10px auto 0;
  min-height: 15px;
}

.slider-bar {
  height: 10px;
  border: 1px solid #c3c3c3;
  border-radius: 5px;
  background: #e4e4e4;
  box-shadow: 0 1px 1px rgba(12, 10, 10, 0.2) inset;
  position: absolute;
  width: 100%;
  top: 7px;
}

#puzzle-box {
  position: absolute;
  left: 0;
  top: 0;
  z-index: 22;
}

#puzzle-shadow {
  position: absolute;
  left: 0;
  top: 0;
  z-index: 22;
}

#puzzle-lost {
  position: absolute;
  left: 0;
  top: 0;
  z-index: 33;
}

.puzzle-lost-box {
  position: absolute;
  width: 260px;
  height: 116px;
  left: 0;
  top: 0;
  z-index: 111;
}

.refrech svg{
  width: 20px;
  height: 20px;
  cursor: pointer;
}
</style>