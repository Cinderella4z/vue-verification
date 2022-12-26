<template>
  <div class="verification-box" :style="verificationBoxStyle" v-if="iSverificationShow" ref="verificationBox">
    <div class="box" :style="{ backgroundImage: `url('/src/components/verificationBox/assets/${backgroundImg}.jpg')` }">
      <div class="box-child" ref="checkBox" :style="boxChildStyle"></div>
      <div class="success-box" v-show="status === 'success'"></div>
      <div class="fail-box" v-show="status === 'fail'"></div>
      <div class="cover-box" v-if="status"></div>
    </div>

    <div class="drag">
      <span>滑动以进行验证</span>
      <div class="drag-box" @mousedown="handleDrag" ref="dragBox">>></div>
    </div>
  </div>

</template>
<script  setup>
import { ref, reactive, toRefs, computed, onMounted } from 'vue';
const props = defineProps({
  width: {
    type: Number,
    default: 400
  },
  success: {
    type: Function,
    default: () => { }
  },
  fail: {
    type: Function,
    default: () => { }
  }
})
const { width, success, fail } = toRefs(props)
const height = ref((width.value / 4) * 3)
const dragBox = ref(null)
const checkBox = ref(null)
const verificationBox = ref(null)
const dragBoxPosition = reactive({
  dragX: renderNum((width.value / 2), (width.value * 0.8)),
  dragY: Math.random() * (height.value * 0.6) + 'px',
  _dragX: renderNum(0, (width.value * 0.3)),
})
const status = ref('')
const slide = ref('')
const iSverificationShow = ref(true)
const imgName = ['a0', 'a1', 'a2']

const verificationBoxStyle = computed(() => {
  return { 'width': width.value + 'px', 'height': height.value + 'px' }
})
const boxChildStyle = computed(() => {
  return { 'backgroundSize': `${width.value}px ${height.value * 0.8}px`, 'backgroundPosition': `-${dragBoxPosition.dragX} -${dragBoxPosition.dragY}` }
})
const backgroundImg = computed(() => {
  const random = Math.floor(Math.random() * 10)
  return random >= imgName.length ? `a${imgName.length - 1}` : `a${random}`
})
const verificationBoxLeft = computed(() => verificationBox.value.getBoundingClientRect().left)
/**
 * @description 生成所需要的随机数区间
 * @param {最小值} min 
 * @param {最大值} number 
 */
function renderNum (min, number) {
  const num = Math.floor(Math.random() * number)
  if (num < min) return min + 'px'
  else return num + 'px'
}
/**
 * @description 主要功能函数，拖动滑块
 * @param {拖动的距离} event.pageX 
 */
const drag = ({ pageX, }) => {
  const num = dragBoxPosition._dragX.slice(0, dragBoxPosition._dragX.length - 2)
  const slideWidth = pageX - verificationBoxLeft.value
  if (slideWidth < 0 || slideWidth > width.value * 0.8 - Number(num)) {
    return
  }
  slide.value = slideWidth + 'px'
  dragBox.value.style.transform = `translateX(${slideWidth}px)`
  checkBox.value.style.transform = `translateX(${slideWidth}px)`
}
const handleDrag = (e) => {
  document.addEventListener('mousemove', drag)
  document.addEventListener('mouseup', handleCheck)
  return a
}
const handleCheck = (e) => {
  function check (pageX, range) {
    const num = dragBoxPosition.dragX.slice(0, dragBoxPosition.dragX.length - 2)
    const initial = dragBoxPosition._dragX.slice(0, dragBoxPosition._dragX.length - 2)
    const subtraction = Math.abs(pageX + Number(initial) - Number((num)))
    if (subtraction < range) {
      success.value()
      handleSuccess()
    } else {
      fail.value()
      handleFail()
    }
  }
  document.removeEventListener('mousemove', drag)
  check(e.pageX - verificationBoxLeft.value, 10)
}
const handleFail = () => {
  new Promise((res) => {
    status.value = 'fail'
    document.removeEventListener('mouseup', handleCheck)
    res()
  }).then(res => {
    setTimeout(() => {
      status.value = ''
      drag({ pageX: verificationBoxLeft.value })
      dragBoxPosition.dragX = renderNum((width.value / 2), (width.value * 0.8))
      dragBoxPosition.dragY = Math.random() * (height.value * 0.6) + 'px'
    }, 2000);
  })
}
const handleSuccess = () => {
  status.value = 'success'
  document.removeEventListener('mouseup', handleCheck)
  setTimeout(() => {
    iSverificationShow.value = false
  }, 2000);
}

</script>
<style  scoped>
.verification-box {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

.box {
  width: 100%;
  height: 80%;
  background-size: 100% 100%;
  background-image: url('./assets/a2.jpg');
  position: relative;
}

.box::before {
  content: '';
  width: 20%;
  height: 20%;
  background: rgba(0, 0, 0, 0.5);
  border: 1px solid #fff;
  position: absolute;
  top: v-bind('dragBoxPosition.dragY');
  left: v-bind('dragBoxPosition.dragX');
}

.box-child {
  width: 20%;
  height: 20%;
  border: 1px solid #fff;
  background-image: inherit;
  background-repeat: inherit;
  position: absolute;
  top: v-bind('dragBoxPosition.dragY');
  left: v-bind('dragBoxPosition._dragX');
}

.drag {
  width: 100%;
  height: 18%;
  min-height: 25px;
  background-color: #dad6d6;
  text-align: center;
  position: relative;
  user-select: none;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
}

.drag::after {
  content: '';
  position: absolute;
  background-color: #52c41a;
  left: 0px;
  width: v-bind(slide);
  height: 100%;
}

.drag span {
  position: absolute;
  display: inline-block;
  width: 50%;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  text-overflow: ellipsis;
  overflow: hidden;
  white-space: nowrap
}

.drag-box {
  width: 20%;
  height: 100%;
  background-color: #fff;
  position: absolute;
  top: 0;
  font-weight: bolder;
  color: #c0c4cc;
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 9;
}

.success-box {
  width: 50px;
  height: 50px;
  background-color: #52c41a;
  border-radius: 30px;
  position: absolute;
  transform: translate(-50%, -50%);
  left: 50%;
  top: 50%;
  color: white;
  z-index: 9;
}

.success-box::before {
  content: '✔';
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-size: 30px;
  color: white;
}

.success-box::after {
  content: '成功';
  position: absolute;
  bottom: 0;
  transform: translate(-50%, 100%);
  font-size: 15px;
  text-align: center;
  left: 50%;
  width: 100%;
}

.fail-box {
  width: 50px;
  height: 50px;
  background-color: #e84335;
  border-radius: 30px;
  position: absolute;
  transform: translate(-50%, -50%);
  left: 50%;
  top: 50%;
  color: white;
  z-index: 9;
}

.fail-box::before {
  content: '✖';
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-size: 30px;
  color: white;
}

.fail-box::after {
  content: '失败';
  position: absolute;
  bottom: 0;
  transform: translate(-50%, 100%);
  font-size: 15px;
  text-align: center;
  left: 50%;
  width: 100%;
}

.cover-box {
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  position: absolute;
}
</style>