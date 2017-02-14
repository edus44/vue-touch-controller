<template>
    <svg
        class="touch-control-shot"
        v-tapstart="setCenter"
        v-tapmove="moveStick"
        v-tapend="resetCenter"
    >
        <clipPath id="frontier">
            <circle :r="size*2" />
        </clipPath>

        <g 
            :transform="translateCenter" 
            clip-path="url(#frontier)"
        >
            <circle 
                class="touch-control-shot-stick-limits"
                v-for="n in steps"
                :r="size * n" 
            />
            <line 
                class="touch-control-shot-stick-line"
                v-for="line in lines"
                :x1="line.x1" 
                :y1="line.y1" 
                :x2="line.x2" 
                :y2="line.y2"
            />
            <line 
                class="touch-control-shot-stick-fire"
                v-show="holding"
                :x2="fire.x" 
                :y2="fire.y"
            />

            <circle 
                class="touch-control-shot-stick-handler"
                :r="size*.3" 
                :cx="stick.x" 
                :cy="stick.y" 
            />
        </svg>
    </svg>
</template>

<script>

import * as tapDirectives from 'tapDirectives'

export default {
    directives:tapDirectives,
    data(){
        return {
            steps: [.08,.22,.42,.66,1,2],
            size: 80,
            center: {x: 0,y: 0},
            stick: {x: 0,y: 0},
            fire: {x: 0, y: 0},
            bounds: null,
            holding: false,
        }
    },
    computed:{
        lines(){
            let d = this.size*2
            return [
                {x1:0, y1:-d, x2:0, y2:d},
                {x1:-d, y1:0, x2:d, y2:0},
            ]
        },
        translateCenter(){
            return `translate(${this.center.x},${this.center.y})`
        }
    },
    mounted(){
        this.onResize()
        window.addEventListener('resize',this.onResize)
    },
    destroyed(){
        window.removeEventListener('resize',this.onResize)
    },
    methods:{
        onResize(){
            this.bounds = this.$el.getBoundingClientRect()
            this.resetCenter()
        },
        resetCenter(){
            this.center = {
                x: this.bounds.width/2,
                y: this.bounds.height/2
            }
            this.stick = {x:0,y:0}
            this.holding = false
            this.emitPos()
        },
        setCenter(e){
            this.center = this.getXY(e)
            this.holding = true
            this.emitPos()
        },
        moveStick(e){
            if (!this.holding)
                return
            let coords = this.getXY(e)

            //Stick
            let pos = calcStickPos(coords,this.center,this.size)

            this.emitPos(pos.x,pos.y)

            this.stick = {
                x: pos.x * this.size,
                y: pos.y * this.size
            }

            //Fireline
            this.fire = {
                x: pos.x * 10000,
                y: pos.y * 10000
            }
        },
        getXY(e){
            return {
                x:e.pageX - this.bounds.left,
                y:e.pageY - this.bounds.top
            }
        },
        emitPos(x,y){
            this.$emit('update',{
                x:x || 0,
                y:y || 0
            })
        }
    }
}

function calcStickPos(coords,center,size){
    let dX = (coords.x - center.x) / size
    let dY = (coords.y - center.y) / size

    //Distance to center
    let module = Math.sqrt( (dX*dX) + (dY*dY) )


    //Off limits
    if (module>1){

        //Angle
        let angle = Math.atan(dY/dX)

        //Intersect position
        let limX = Math.cos(angle)
        let limY = Math.sin(angle)

        if (dX<0){
            dX = -limX
            dY = -limY
        }else{
            dX = limX
            dY = limY
        }
    }

    return {x:dX,y:dY} 
}

</script>

<style>
.touch-control-shot{
    background-color: rgba(207, 33, 33,.1);
    width: 100%;
    height: 100%;
    display: block;
}

.touch-control-shot-stick-center{
    fill:rgb(207, 33, 33);
}
.touch-control-shot-stick-limits{
    fill:none;
    stroke:rgb(207, 33, 33);
    stroke-width:2;
}
.touch-control-shot-stick-line{
    stroke:rgb(207, 33, 33);
    stroke-width:2;
}
.touch-control-shot-stick-fire{
    stroke:rgb(255, 166, 8);
    stroke-width:3;
    stroke-dasharray: 10, 5;
    animation: dash .2s linear infinite;
}
.touch-control-shot-stick-handler{
    fill:rgba(0,0,0,.6);
}
@keyframes dash {
  to {
    stroke-dashoffset: -15;
  }
}
</style>
