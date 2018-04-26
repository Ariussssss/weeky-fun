<template>
  <div id="app" v-on="{ mousedown: select,mousemove: selectmove,mouseup: selectover,mouseleave: selectover }">
    <router-view />
    <div class="select" v-bind:style="selectstyle()">
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      startX: 0,
      startY: 0,
      newT: 0,
      newL: 0,
      newR: 0,
      newB: 0,
      action: 0,
      loc:'',
      moveChoose: 0
    }
  },
  methods: {
    selectstyle() {
      return {
        display: this.action !== 1 ? 'none':'block',
        top: this.newT + 'px',
        left: this.newL + 'px',
        height: Math.abs(this.newB - this.startY) + 'px',
        width: Math.abs(this.newR - this.startX) + 'px',
      }
    },
    select() {
      if (event.path[0].className == 'board') {
        this.startX = event.clientX
        this.startY = event.clientY
        this.newT = 200
        this.newL = event.clientX
        this.newB = event.clientY
        this.newR = event.clientX
        this.action = 1
        this.$root.Bus.$emit('reset')
      }
      else if (event.path[0].className == 'block') {
        this.$root.Bus.$emit('reset')
        this.action = 3
        let choose = [].indexOf.call(event.target.parentNode.children,event.target)
        this.moveChoose = choose
        this.$root.Bus.$emit('onechange', { 'index': choose, 'state': true})
      }
      else if (event.path[0].className == 'block is-active is-run') {
        this.moveChoose = [].indexOf.call(event.target.parentNode.children,event.target)
        this.action = 2
      }
      else {
      }
    },
    selectmove() {
      if (this.action === 1) {
        // console.log(event)
        this.newT = Math.min(this.startY, event.clientY)
        this.newL = Math.min(this.startX, event.clientX)
        this.newB = event.clientY
        this.newR = event.clientX
        this._choose()
      }
      else if (this.action === 2) {
        // console.log(event)
        this.$root.Bus.$emit('onemove', 
          { 
            'x': event.movementX, 
            'y': event.movementY,
            'choose':this.moveChoose
          })
        if (this._delete(event)) {
          this.$root.Bus.$emit('trashState', { 'state': 1 })
        }
        else {
          this.$root.Bus.$emit('trashState', { 'state': 0 })
        }
      }
      else if (this.action === 3) {
       this.$root.Bus.$emit('onewidth', 
          { 
            'x': event.clientX , 
          })
      }
    },
    selectover() {
      if (this.action === 2) {
        this.action = 0
        if (this._delete(event)) {
          this.$root.Bus.$emit('trashState', { 'state': 2 })
        }
        else {
          this.$root.Bus.$emit('movereset')
        }
      }
      else {
        if (this.action === 3) {
          this.$root.Bus.$emit('reset')
        }
        this.$root.Bus.$emit('movereset')
      }
      this.action = 0
    },
    getPosition(obj)  {  
       var left = 0;  
       var top   = 0;  
       while(obj != document.body)
       {  
              left = obj.offsetLeft;  
              top   = obj.offsetTop;  
              obj = obj.offsetParent;  
       }  
       return {'left':left,'top':top}  
    },  
    _choose() {
      let seldiv = document.documentElement.getElementsByClassName('select')[0]
      let seldivL = seldiv.offsetLeft - 40 
      let seldivT = seldiv.offsetTop - 175
      let seldivW = seldiv.offsetWidth
      let seldivH = seldiv.offsetHeight 
      let selList = document.documentElement.getElementsByClassName('weeky')[0].children
      for (var i = selList.length - 1; i >= 0; i--) {
        let res = this.getPosition(selList[i])
        let sl = selList[i].offsetWidth + selList[i].offsetLeft + res.left 
        let st = selList[i].offsetHeight + selList[i].offsetTop + res.top 
        if (sl > seldivL && 
            st > seldivT && 
            selList[i].offsetLeft + res.left < seldivL + seldivW && 
            selList[i].offsetTop  + res.top < seldivT + seldivH) { 
            this.$root.Bus.$emit('onechange', { 'index': i, 'state': true})
        }
        else {
            this.$root.Bus.$emit('onechange', { 'index': i, 'state': false})
        }
      }
    },
    _delete(mous) {
      let mousL = mous.clientX
      let mousT = mous.clientY
      let trash = document.documentElement.getElementsByClassName('trash')[0]
      let res = this.getPosition(trash)
      let sl = trash.offsetWidth + trash.offsetLeft + res.left 
      let st = trash.offsetHeight + trash.offsetTop + res.top 
      if (sl > mousL - 50 && 
          st > mousT - 50 && 
          trash.offsetLeft + res.left < mousL && 
          trash.offsetTop  + res.top < mousT) { 
          return true
      }
      else {
         return false
      }
    },
  }
}
</script>

<style lang="stylus" rel="stylesheet/stylus">
#app 
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  -moz-user-select: none;  
  -webkit-user-select: none;  
  user-select: none;  
  text-align: center;
  width: 100%
  height: 100%
  overflow: hidden
  .select
    position: absolute
    background-color:#C3D5ED
    color: black
    z-index:1000
    border-radius: 10px
    border: 2px solid #00e2ff
    opacity:0.6
</style>
