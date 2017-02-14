<template>
    <svg
        id="touch-controller"
        v-tapstart="setCenter"
        v-tapmove="moveStick"
        v-tapend="resetCenter"
    >
        
        <g :transform="translateCenter">
            <circle 
                class="touch-controller-stick-center"
                :r="this.size*0.17" 
            />
            <circle 
                class="touch-controller-stick-limits"
                :r="this.size" 
            />
            <circle 
                class="touch-controller-stick-handler"
                :r="this.size*.3" 
                :cx="stick.x" 
                :cy="stick.y" 
            />
        </g>
    </svg>
</template>

<script>

import * as tapDirectives from 'tapDirectives'

export default {
    directives:tapDirectives,
    data(){
        return {
            size:100,
            center:{x:0,y:0},
            stick:{x:0,y:0},
            bounds:null,
            holding: false,
        }
    },
    computed:{
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
            let {dX,dY} = calcStickPos(coords,this.center,this.size)
            this.emitPos(dX,dY)
            this.stick = {
                x: dX * this.size,
                y: dY * this.size
            }
        },
        getXY(e){
            return {
                x:e.pageX - this.bounds.left,
                y:e.pageY - this.bounds.top
            }
        },
        emitPos(dX,dY){
            this.$emit('update',{
                x:dX || 0,
                y:dY || 0
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

    return {dX,dY} 
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
    fill:#3273dc; 
}
.touch-controller-stick-limits{
    fill:rgba(50, 115, 220, 0.3);
    stroke:#3273dc;
    stroke-width:3;
}
.touch-controller-stick-handler{
    fill:rgba(0,0,0,.6);
}
</style>
