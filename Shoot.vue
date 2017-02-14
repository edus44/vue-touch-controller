<template>
    <svg
        id="touch-controller"
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
                class="touch-controller-stick-limits"
                v-for="n in steps"
                :r="size * n" 
            />
            <line 
                class="touch-controller-stick-line"
                v-for="line in lines"
                :x1="line.x1" 
                :y1="line.y1" 
                :x2="line.x2" 
                :y2="line.y2"
            />
            <line 
                class="touch-controller-stick-fire"
                v-show="holding"
                :x2="fire.x" 
                :y2="fire.y"
            />

            <circle 
                class="touch-controller-stick-handler"
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
            steps:[.1,.25,.5,.75,1,2],
            size:80,
            center:{x:0,y:0},
            stick:{x:0,y:0},
            fire:{
                x:0,
                y:0,
            },
            bounds:null,
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
            let x = this.bounds.width/2
            let y = this.bounds.height/2
            this.center = {x,y}
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

<style scoped>
#touch-controller{
    background-color: #eee;
    width: 100%;
    height: 100%;
    display: block;
}

.touch-controller-stick-center{
    fill:rgba(207, 33, 33, 1);
}
.touch-controller-stick-limits{
    fill:none;
    stroke:rgba(207, 33, 33, 1);
    stroke-width:2;
}
.touch-controller-stick-line{
    stroke:rgba(207, 33, 33, 1);
    stroke-width:2;
}
.touch-controller-stick-fire{
    stroke:rgb(255, 166, 8);
    stroke-width:4;
}
.touch-controller-stick-handler{
    fill:rgba(0,0,0,.6);
}
</style>
