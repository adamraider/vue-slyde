<template lang="html">
<div class="ui-slider" tabindex="0" ref="slider"
     :class="sliderClassList"
     :style="sliderStyle"
     @mousedown="sliderClick" 
     @keydown.left.prevent="decrement" 
     @keydown.right.prevent="increment">

  <div class="ui-slider-containment" ref="container"></div>

  <div class="ui-slider-wrapper">
    <div class="ui-slider-track-container">
      <div class="ui-slider-track"></div>
      <div class="ui-slider-track-fill" :style="fillStyle"></div>
    </div>

    <div class="ui-slider-thumb-container" ref="thumb">
      <div class="ui-slider-focus-ring"></div>
      <div class="ui-slider-thumb"></div>
    </div>
  </div>
</div>
</template>

<script>
import Draggabilly from 'draggabilly'

export default {
  props: {
    valueToSync: {
      default: 1
    },
    keyboardStep: {
      default: 0.1
    },
    orientation: {
      default: 'x'
    },
    height: {
      default: '200px'
    }
  },

  watch: {
    valueToSync (value) {
      this.setValue(value)
    }
  },

  data () {
    return {
      value: 1,
      dragging: false,
      draggable: null
    }
  },

  mounted () {
    // Set initial value
    if (this.orientation === 'y') {
      this.$refs.thumb.style.bottom = (this.value * 100) + '%'  
    } else {
      this.$refs.thumb.style.left = (this.value * 100) + '%'  
    }

    // Initialize Draggabilly
    this.draggable = new Draggabilly(this.$refs.thumb, {
      containment: this.$refs.container,
      axis: this.orientation
    })

    // Setup drag events
    this.draggable.on('dragStart', this.dragStart)
    this.draggable.on('dragMove', this.dragMove)
    this.draggable.on('dragEnd', this.dragEnd)
  },

  methods: {
    sliderClick (e) {
      let sliderPosition = this.$refs.slider.getBoundingClientRect()

      let newValue
      if (this.orientation === 'y') {
        newValue = -((e.clientY - sliderPosition.bottom) / sliderPosition.height)
      } else {
        newValue = ((e.clientX - sliderPosition.left) / sliderPosition.width)
      }

      newValue = Math.round(newValue * 100) / 100
      this.setValue(newValue)

      if (e.target !== this.$refs.thumb) {
        this.draggable._pointerDown(e, e)
      }

      this.$refs.slider.focus()
    },

    dragStart () {
      this.dragging = true
      this.$refs.slider.focus()
    },

    dragMove () {
      let newValue
      if (this.orientation === 'y') {
        let y = this.draggable.position.y
        let height = this.$refs.slider.getBoundingClientRect().height
        let yFromBottom = height - y
        newValue = (yFromBottom / height)
      } else {
        let x = this.draggable.position.x
        newValue = (x / this.$refs.slider.getBoundingClientRect().width)
      }
      this.setValue(newValue)
    },

    dragEnd () {
      this.dragging = false
    },

    increment () {
      if (this.value === 1) {
        return
      }

      this.setValue(this.value + this.keyboardStep)
    },

    decrement () {
      if (this.value === 0) {
        return
      }

      this.setValue(this.value - this.keyboardStep)
    },

    setValue (newValue) {
      newValue = Math.round(newValue * 1000) / 1000
      if (newValue === this.value) {
        return
      }

      // Round the value
      let moderatedValue = newValue

      // Check for max
      if (moderatedValue >= 1) {
        moderatedValue = 1
      }

      // Check for min
      if (moderatedValue <= 0) {
        moderatedValue = 0
      }

      if (!this.dragging) {
        if (this.orientation === 'y') {
          this.$refs.thumb.style.bottom = `${moderatedValue * 100}%`
        } else {
          this.$refs.thumb.style.left = `${moderatedValue * 100}%`
        }
      }

      this.value = moderatedValue
      this.$emit('update', moderatedValue)
    }
  },

  computed: {
    sliderClassList () {
      return { 
        min: this.value === 0, 
        max: this.value === 1, 
        dragging: this.dragging, 
        'orientation-x': this.orientation === 'x', 
        'orientation-y': this.orientation === 'y'
      }
    },

    sliderStyle () {
      if (this.orientation === 'y') {
        return { height: this.height }
      }
    },

    fillStyle () {
      if (this.orientation === 'y') {
        return { height: `${this.value * 100}%` }
      } else {
        return { width: `${this.value * 100}%` }
      }
    }
  }
}
</script>

<style lang="sass?indentedSyntax" scoped>
// Overall slider width
$slider-width: 100%

// Track line
$track-height: 3px
$track-background-color: rgba(0, 0, 0, 0.28)
$track-fill-color: #F44336

// Thumb
$thumb-size: 16px
$thumb-scale-active: 1.1
$thumb-container-size: 38px

// Transition durations
$thumb-transition-duration: 0.2s
$track-transition-duration: 0.1s

// Colors
$thumb-fill-color: $track-fill-color
$thumb-background-color: $track-fill-color

// Slider container
.ui-slider
  position: relative
  display: flex
  outline: none
  &.orientation-x
    width: $slider-width
    height: $thumb-container-size + 4px
    align-items: center
  &.orientation-y
    width: $thumb-container-size + 4px
    height: $slider-width
    justify-content: center
  &:hover
    .ui-slider-thumb-container
      display: flex

  // Increase size of thumb on slider hover
  &:hover
    .ui-slider-thumb
      transform: scale($thumb-scale-active)

  // Increase size of thumb and focus dot on focus
  &:focus,
  &.dragging
    .ui-slider-thumb
      transform: scale($thumb-scale-active)
    
    .ui-slider-focus-ring
      transform: scale(1)

.ui-slider-wrapper
  position: relative
.orientation-x
  .ui-slider-wrapper
    width: 100%
.orientation-y
  .ui-slider-wrapper
    height: 100%

.ui-slider-track-container
  position: relative
.orientation-y
  .ui-slider-track-container
    height: 100%
    display: flex
    justify-content: center

.ui-slider-track
  background-color: $track-background-color
.orientation-x
  .ui-slider-track
    width: 100%
    height: $track-height
.orientation-y
  .ui-slider-track
    width: $track-height
    height: 100%

.ui-slider-track-fill
  position: absolute
  width: $track-height
  background-color: $track-fill-color
.orientation-x
  .ui-slider-track-fill
    top: 0
    height: 100%
.orientation-y
  .ui-slider-track-fill
    bottom: 0

.ui-slider-thumb-container
  position: absolute
  display: none // updated on hover
  align-items: center
  justify-content: center
  width: $thumb-container-size
  height: $thumb-container-size
  top: 0
  left: 0
  bottom: 0
  right: 0
  margin-top: -($thumb-container-size / 2) - ($track-height / 2)
  margin-left: -($thumb-container-size / 2) - ($track-height / 2)
.orientation-y
  .ui-slider-thumb-container
    margin-left: -17.5px

.ui-slider-thumb
  display: flex
  align-items: center
  justify-content: center
  width: $thumb-size
  height: $thumb-size
  background-color: $thumb-fill-color
  border-radius: 50%
  transition-property: transform
  transition-duration: $thumb-transition-duration
  transition-timing-function: linear
  transform: scale(1)

.ui-slider-focus-ring
  position: absolute
  top: 0
  left: 0
  
  width: $thumb-container-size
  height: $thumb-container-size
  
  border-radius: 50%
  background-color: rgba($thumb-fill-color, 0.38)
  
  transition: transform 0.2s ease
  transform: scale(0)

.ui-slider-containment
  left: 0
  right: 0
  position: absolute
.orientation-x
  .ui-slider-containment
    margin-left: -($thumb-container-size / 2) - ($track-height / 2)
    margin-right: -($thumb-container-size / 2) - ($track-height / 2)
.orientation-y
  .ui-slider-containment
    top: 0
    bottom: 0
    margin-top: -($thumb-container-size / 2) - ($track-height / 2)
    margin-bottom: -($thumb-container-size / 2) - ($track-height / 2)
</style>
