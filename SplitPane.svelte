<script>
// Based on https://github.com/Readiz/svelte-split-pane
import { onMount, onDestroy } from 'svelte'

export let split// = ''
export let height = {
  top: '50%',
  btm: '50%',
  rest: 0,
}
// export let width = {
//   left: '50%',
//   right: '50%',
//   rest: 0,
// }
// export let min = {
//   top: '0',
//   btm: '0',
//   left: '0',
//   right: '0',
// }
export let updateCallback = () => {}

let capture = {}
let top, btm, left, right
let rowSep, colSep

// FIXME: Need redo drag events
const onMouseDown = (e) => {
  e.preventDefault()
  if (e.button !== 0) 
    return
  
  capture = split==='-' ? { // VSplitPane
    e,
    offsetLeft: rowSep.offsetLeft,
    offsetRight: rowSep.offsetRight,
    topHt: top.offsetHeight,
    btmHt: btm.offsetHeight,
  } : { // HSplitPane
    e,
    offsetLeft: colSep.offsetLeft,
    offsetRight: colSep.offsetRight,
    leftWd: left.offsetWidth,
    rightWd: right.offsetWidth,
  }
  window.addEventListener('mousemove', onMouseMove)
  window.addEventListener('mouseup', onMouseUp)
  window.addEventListener('touchmove', onMouseMove)
  window.addEventListener('touchend', onMouseUp)
}

const onMouseMove = (e) => { 
  e.preventDefault()
  if (e.button !== 0)
    return
  let delta = {
    x: e.clientX - capture.e.clientX,
    y: e.clientY - capture.e.clientY,
  }
  if (split==='-') { // VSplitPane
    delta.y = Math.min(Math.max(delta.y, -capture.topHt), capture.btmHt)
    rowSep.style.top = capture.offsetTop + delta.y + 'px'
    top.style.height = (capture.topHt + delta.y) + 'px'
    btm.style.height = (capture.btmHt - delta.y) + 'px'
  } 
  else { // HSplitPane
    delta.x = Math.min(Math.max(delta.x, -capture.leftWd), capture.rightWd)
    colSep.style.left = capture.offsetLeft + delta.x + 'px'
    console.log('delta', delta.x)
    // const width = capture.leftWd + capture.rightWd
    // const rightWd = max.right ? Math.max(max.right, capture.rightWd) : capture.rightWd
    // const leftWd = max.right ? width - rightWd : capture.leftWd
    // left.style.width = (leftWd + delta.x) + 'px'
    // right.style.width = (rightWd - delta.x) + 'px'
    left.style.width = (capture.leftWd + delta.x) + 'px'
    right.style.width = (capture.rightWd - delta.x) + 'px'
  }
  updateCallback();
}
const onMouseUp = (e) => {
  if (e) {
    e.preventDefault();
    if (e.button !== 0) return
  }
  updateCallback()
  window.removeEventListener('mousemove', onMouseMove)
  window.removeEventListener('mouseup', onMouseUp)
  window.removeEventListener('touchmove', onMouseMove)
  window.removeEventListener('touchend', onMouseUp)
}

function resetSize() {
  if (top && btm) {
    if (!height.top && height.btm) {
      top.style.height = (window.innerHeight - height.btm - height.rest)+'px'
      btm.style.height = height.btm+'px'
// console.log('pane resize', top.style.height)
    }
  }
  //   if (right) {
  //     right.style.width = size.right+'px'
  //     left.style.width = `calc(100% - ${size.right}px)`
  // console.log('pane resize', left.style.width)
  //   }
}
function onResize() {
  onMouseUp()
  resetSize()
}

onMount(() => {
  window.addEventListener('resize', onResize)
  resetSize()
})
onDestroy(() => {
  window.removeEventListener('resize', onResize)
})
</script>

<style>
div.wrap {
  width: 100%;
  height: 100%;
  display: flex;
}
.row-wrap {
  display: flex;
  flex-direction: column;
}
.col-wrap {
  display: inline-flex;
}
div.separator {
  z-index: 1;
  background-color: #aaa;
}
.row-separator {
  cursor: row-resize;
  width: 100%;
  height: 4px;
  margin-top: -2px;
}
.col-separator {
  cursor: col-resize;
  height: 100%;
  width: 4px;
  margin-left: -2px;
  background-image: url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='10' height='30'><path d='M2 0 v30 M5 0 v30 M8 0 v30' fill='none' stroke='black'/></svg>");
  background-repeat: no-repeat;
  background-position: center;
}
div.top {
  width: 100%;
}
div.btm {
  width: 100%;
}
div.left {
  height: 100%;
}
div.right {
  height: 100%;
}
</style>

{#if split==='|'}
  <div class="col-wrap wrap">
    <div bind:this={left} class="left">
      <slot name="left"></slot>
    </div>
    <div bind:this={colSep} class="col-separator separator" on:mousedown={onMouseDown} on:touchstart={onMouseDown}>
    </div>
    <div bind:this={right} class="right">
      <slot name="right"></slot>
    </div>
  </div>
{:else if split==='-'}
  <div class="row-wrap wrap">
    <div bind:this={top} class="top">
      <slot name="top"></slot>
    </div>
    <div bind:this={rowSep} class="row-separator separator" on:mousedown={onMouseDown} on:touchstart={onMouseDown}>
    </div>
    <div bind:this={btm} class="btm">
      <slot name="btm"></slot>
    </div>
  </div>
{/if}
