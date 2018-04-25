<template>
<div class="board"  >
    <div class="weeky-background" >
      <div class="block-background" v-for='one in 7'>
      </div>
    </div>
    <div class="weeky" >
      <Block v-for = '(one,index) in week' :data='one' :key='index'></Block>
    </div>
    <div class="trash" :class="{ 'before-active' : trashState === 1 ,'active' : trashState === 2 }" v-on:click='resetBan'>
      <img src="static/image/cc-trash-hat.png" alt="trash-hat" class="trash-hat">
      <img src="static/image/cc-trash-body.png" alt="trash-body" class="trash-body">
    </div>
  <div class="speak"  :class="{ 'active' : showSpeak }">{{ words }}</div>
  <div class="speak"  :class="{ 'active' : showBusy }">{{ words }}</div>
</div>
</template>

<script>
import Block from './Block'

export default {
  name: 'Board',
  data () {
    return {
      week:[
        {
          'id': 0,
          'name':'SUN',
          'active':false,
          'run': false,
          'ban': false
        },{
          'id': 1,
          'name':'MON',
          'active':false,
          'run': false,
          'ban': false
        },{
          'id': 2,
          'name':'TUE',
          'active':false,
          'run': false,
          'ban': false
        },{
          'id': 3,
          'name':'WED',
          'active':false,
          'run': false,
          'ban': false
        },{
          'id': 4,
          'name':'THU',
          'active':false,
          'run': false,
          'ban': false
        },{
          'id': 5,
          'name':'FRI',
          'active':false,
          'run': false,
          'ban': false
        },{
          'id': 6,
          'name':'SAT',
          'active':false,
          'run': false,
          'ban': false
        },],
      trashState: 0,
      words: 0,
      showSpeak: false,
      showBusy: false
    }
  },
  components: {
    Block
  },
  methods: {
    onechange(data) {
      this.week[data.index].active = data.state
      this.week[data.index].run = data.state
    },
    reset() {
      this.week.map((e)=>{
        e.active = false
        e.run = false
      })
    },
    resetBan() {
      let num = 0
      this.$root.Bus.$emit('widthreset')
      this.week.map((e)=>{
        if (e.ban) {
          num +=1
        }
        e.active = false
        e.run = false
        e.ban = false
      })
      if (num === 0) {
        this.words = 'EMPTY'
      }
      else if (num === 1){
        this.words = num + ' DAY BACK'
        this.trashState = 2
      }
      else {
        this.words = num + ' DAYS BACK'
        this.trashState = 2
      }
      if (this.showSpeak) {
          this.showBusy = true
           setTimeout(()=>{
              this.showBusy = false
            },1200)
        }
        else {
          this.showSpeak = true
            setTimeout(()=>{
              this.trashState = 0
              this.showSpeak = false
            },1200)
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
    delete(data) {
      if (data.state === 0) {
        this.trashState = 0
      }
      else  if (data.state === 1 && this.trashState === 0) {
        this.trashState = 1
      }
      else if (data.state === 2) {
        this.trashState = 2
        let num = 0
        this.week.map((e)=>{
          if (e.active && !e.ban) {
            e.ban = true
            num +=1
          }
        })
        if (num === 0) {
          this.words = 'EMPTY'
        }
        else if (num === 1){
          this.words = num + ' DAY REMOVED'
        }
        else {
          this.words = num + ' DAYS REMOVED'
        }
        if (this.showSpeak) {
          this.showBusy = true
           setTimeout(()=>{
              this.showBusy = false
            },1200)
        }
        else {
          this.showSpeak = true
            setTimeout(()=>{
              this.trashState = 0
              this.showSpeak = false
            },1200)
        }
      console.log(this.trashState)
      }
    }
  },
  mounted() {
    this.$root.Bus.$on('reset', this.reset)
    this.$root.Bus.$on('trashState', this.delete)
    this.$root.Bus.$on('onechange', this.onechange)
    // let selList = document.documentElement.getElementsByClassName('weeky')[0].children
    //   for (var i = selList.length - 1; i >= 0; i--) {
    //     let sl = selList[i].offsetWidth + selList[i].offsetLeft; 
    //     let st = selList[i].offsetHeight + selList[i].offsetTop ; 
    //     let res = this.getPosition(selList[i])
    //     this.week[i].name = res.top+ '\n' + res.left + '\n' + 
    //       selList[i].offsetWidth + '\n' + selList[i].offsetHeight + '\n'
    //   }
  },
  beforeDestroy() {
    this.$root.Bus.$off(['reset','onechange','trashState'])
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="stylus" rel="stylesheet/stylus" scoped>
.board
  position: absolute
  width: 800px
  height: 500px
  top: 0
  left: 0
  right: 0
  bottom: 0
  margin: auto
  .weeky-background,.weeky
    position: absolute
    width: calc(100% - 100px)
    height: 130px
    margin: 175px 50px
    // display: flex
    .block-background
      float: left
      width: calc(100% / 7 - 20px)
      height: calc(100% )
      margin: 0 10px
      background-color: #cfd1d8
      border-radius: 10px
    // background-color: #3e197e
  .trash
    position: absolute
    width: 128px   
    height: 128px 
    left: 0 
    right: 0
    bottom: 20px
    margin: 0 auto
    transition: all .5s
    opacity: 0
    img
      position: absolute
      top: 0
      left: 0
      transition: all .5s
      transform: scale(0.6)
    &.before-active
      opacity: 1
      img
        transition: all .5s 
        transform: scale(0.6) 
    &:hover
      opacity: 1
      img
        transition: all .5s 
        transform: scale(0.6) 
    &.active
      opacity: 1
      img
        transition: all .5s
        &.trash-body
          rot = -20deg
          tray = 0px
          transform-origin: 65px 70px 
          animation:tra1 1s .2s forwards
          @keyframes tra1
            0%
              transform: scale(0.6)
            30%
              transform: scale(0.6) rotate(rot) translateY(tray) 
            50%
              transform: scale(0.6) rotate(0deg) translateY(0px)
              opacity: 1
            100%
              transform: scale(0.6)
        &.trash-hat
          rot = 20deg
          tray = -10px
          trax = -15px
          opacity: 1
          transform-origin: 65px 60px 
          animation:tra2 1s .2s forwards
          @keyframes tra2
            0%
              transform: scale(0.6)
            30%
              transform: scale(0.6) rotate(rot) translateX(trax) translateY(tray)
            50%
              transform: scale(0.6) rotate(0deg) translateY(0px)
            100%
              transform: scale(0.6)
  .select-base
    position: absolute
    width: 100%
    height: 100%
    z-index: 500
  .speak 
    position: absolute
    left: 0 
    right: 0
    color: #fc0372
    bottom: 150px
    margin: 0 auto
    font-size: 15px 
    background-color: #f2f5f7
    opacity: 0
    font-weight: bold
    &.active
      animation:words 1s .1s forwards
      @keyframes words
        0%
          opacity: 0
          transform: translateY(30px)
        40%
          transform: translateY(5px)
          opacity: 1
        60%
          transform: translateY(0px)
          opacity: 1
        100%
          transform: translateY(-30px)
          opacity: 0
</style>
