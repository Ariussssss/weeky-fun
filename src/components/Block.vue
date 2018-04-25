<template>
  <div class="block" :class = "{'is-active': data.active ,'is-run': data.run ,'is-ban': data.ban}" :style='moves()'>
    {{ data.name }}
  </div>
</template>

<script>
export default {
  name: 'Block',
  props: {
    data: {}
  },
  data () {
    return {
      movemomentX:0,
      movemomentY:0,
      movementDelay:0,
      deleteMove:0,
      form: 0,
      to : 0,
      left: 0 ,
      width: 80,
    }
  },
  methods: {
    moves() {
      if (this.data.ban) {
        setTimeout(()=>{
          this.die = true
        },200)
        return {
          left: this.left + 'px',
          width: this.width + 'px',
          transition: 'all .2s linear, opacity .2s linear .1s',
          transform: 'translateX(' + (this.movemomentX + this.deleteMove) + 'px) translateY(' + this.movemomentY +'px) scale(0.1)',
          opacity: 0
          }
      }
      return {
        width: this.width + 'px',
        left: this.left + 'px',
        transition: 'all .2s linear',
        transform: 'translateX(' + this.movemomentX + 'px) translateY(' + this.movemomentY +'px) '
      }
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
    _checkBan(i) {
      return 
    }
  },
  mounted() {
    this.left = (this.data.id * 100) 
    this.width = 80
    this.from = this.data.id
    this.to = this.data.id
    this.$root.Bus.$on('onemove', val => {
      if (this.data.active) {
        this.movementDelay = Math.abs(val.choose - this.data.id)
        this.deleteMove = (val.choose - this.data.id) * 130
        this.left = (this.data.id * 100)
        this.width = 80
        setTimeout(()=>{
          this.movemomentX += val.x
          this.movemomentY += val.y
        },this.movementDelay * 70)
      }
    })
    this.$root.Bus.$on('movereset', () => {
      setTimeout(()=>{
        if (!this.data.ban) {
          this.movemomentX = 0
          this.movemomentY = 0
        }
      },this.movementDelay * 70)
    })
    this.$root.Bus.$on('widthreset', () => {
      this.from = this.data.id
      this.to = this.data.id
      this.left = (this.data.id * 100)
      this.width = 80
    })
    this.$root.Bus.$on('letOther', val => {
      if ((this.data.id != val.puller) && (this.to !== this.from)) {
        console.log(this.data.id)
        if (val.isfrom) {
          if (val.count <= this.to && val.count >= this.from) {
            this.to = val.count - 1
            this.width = ((this.to - this.from) * 100 + 80)
          }
        }
        else {
          if (val.count >= this.from && val.count <= this.to) {
            this.from = val.count + 1
            this.left = this.from * 100
            this.width = ((this.to - this.from) * 100 + 80)
          }
        }
      }
    })
    this.$root.Bus.$on('onewidth', val => {
      if (this.data.active) {
        let selList = document.documentElement.getElementsByClassName('weeky')[0].children
        let res = this.getPosition(selList[this.data.id])
        let newx =(val.x - 90 - (this.data.id * 100) - res.left)
        if (newx > 40 ) {
          let num = parseInt(newx / 80)
          this.left = (this.data.id * 100)
          for (var i = 1; i <= num; i++) {
            if ((i <= (6 - this.data.id)) && (selList[(this.data.id + i)].className == 'block is-ban')) {
                this.width = (80 + i * 100)
                this.from = this.data.id 
                this.to = this.data.id + i
                this.$root.Bus.$emit('letOther',{
                  'puller':this.data.id,
                  'count':this.to,
                  'isfrom':false
                })
            }
            else{
              break
            }
          }
        }
        else if (newx < -40 ) {
          let num = -parseInt(newx / 80)
          for (var i = 1; i <= num; i++) {
            if ((i <= (this.data.id)) && (selList[(this.data.id - i)].className == 'block is-ban')) {
                this.from = this.data.id - i
                this.to = this.data.id
                this.left = ((this.data.id - i) * 100)
                this.width = (80 + i * 100)
                this.$root.Bus.$emit('letOther',{
                  'puller':this.data.id,
                  'count':this.from,
                  'isfrom':true
                })
            }
            else{
              break
            }
          }
        }
        else {
          this.from = this.data.id
          this.to = this.data.id
          this.left = (this.data.id * 100)
          this.width = 80
        }
      }
    })
  },
  beforeDestroy() {
    this.$root.Bus.$off(['onemove','movereset','onewidth','widthreset','letOther'])
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="stylus" rel="stylesheet/stylus" scoped>
.block
  position: absolute
  min-width: calc(100% / 7 - 20px)
  height: calc(100% )
  margin: 0 10px
  background-color: #ffffff
  border-radius: 10px
  box-shadow: 1px 22px 40px -20px #6e78e2bf, 
        10px 10px 40px -10px #6e78e24f;
  display: flex
  box-sizing:border-box
  justify-content: center
  align-items: center
  transition: all .2s
  color: #3e197e
  font-size: 18px
  font-weight: bold
  transition: all .2s linear
  opacity: 1
  &.is-active
    background-color: #f7e3ff
    border: 5px solid #6555ff
    color: #6555ff
    animation:shake .2s 
    @keyframes shake
      0%
        transform: skew(0deg, 0deg)
      60%
        transform: skew(-2deg, -2deg) scale(1.1)
      100%
        transform: skew(0deg, 0deg)
  &.is-run
    z-index: 500
</style>
