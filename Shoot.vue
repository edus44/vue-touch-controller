<template>
    <svg
        id="touch-controller"
        v-tapstart="setCenter"
        v-tapmove="moveStick"
        v-tapend="resetCenter"
    >
    <defs>
        <clipPath id="limits">
            <circle 
                :r="size*2" 
                :cx="center.x" 
                :cy="center.y"
                fill="none"
                stroke-width="10"
            />
        </clipPath>
    </defs>
        <circle 
            class="touch-controller-stick-limits"
            v-for="n in steps"
            :r="size * n" 
            :cx="center.x" 
            :cy="center.y" 
        />
        <line 
            class="touch-controller-stick-line"
            v-for="line in lines"
            :x1="center.x" 
            :y1="center.y" 
            :x2="line.x" 
            :y2="line.y"
        />
        <line 
            class="touch-controller-stick-fire"
            v-show="holding"
            :x1="center.x" 
            :y1="center.y" 
            :x2="fire.x" 
            :y2="fire.y"
            clip-path="url(#limits)"
        />

        <circle 
            class="touch-controller-stick-handler"
            :r="size*.3" 
            :cx="stick.x" 
            :cy="stick.y" 
        />
    </svg>
</template>

<script>

import * as tapDirectives from 'tapDirectives'

export default {
    directives:tapDirectives,
    data(){
        return {
            lineMarks:[0,90,180,270],
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
            return this.lineMarks.map(angle=>{
                let rads = angle/180*Math.PI
                return {
                    x: this.center.x + this.size*2  * Math.cos(rads),
                    y: this.center.y + this.size*2  * Math.sin(rads),
                }
            })
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
            this.stick = {x,y}
            this.holding = false
            this.emitPos()
        },
        setCenter(e){
            this.center = this.stick = this.getXY(e)
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
                x: this.center.x + pos.x * this.size,
                y: this.center.y + pos.y * this.size
            }

            //Fireline
            this.fire = {
                x: this.center.x + pos.x * 10000,
                y: this.center.y + pos.y * 10000
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
    stroke-width:3;
}
.touch-controller-stick-line{
    stroke:rgba(207, 33, 33, 1);
    stroke-width:3;
}
.touch-controller-stick-fire{
    stroke:rgb(255, 166, 8);
    stroke-width:4;
}
.touch-controller-stick-handler{
    fill:rgba(0,0,0,.6);
}
</style>
