<template>
    <svg
        class="touch-control-simple"
        v-tapstart="setCenter"
        v-tapmove="moveStick"
        v-tapend="resetCenter"
    >
        
        <g :transform="translateCenter">
            <circle 
                class="touch-control-simple-stick-center"
                :r="size*0.17" 
            />
            <circle 
                class="touch-control-simple-stick-limits"
                :r="size" 
            />
            <circle 
                class="touch-control-simple-stick-handler"
                :r="size*.3" 
                :cx="pos.x * size" 
                :cy="pos.y * size" 
            />

            <polygon 
                v-if="simplePos.x || simplePos.y"
                class="touch-control-simple-direction"
                points="-40,-10 0,-70 40,-10"
                :transform="transformDir"
            />

        </g>
    </svg>
</template>

<script>

import circleMixin from './circleMixin'

export default {
    mixins: [circleMixin],
    computed:{
        transformDir(){
            let {x,y} = this.simplePos
            let deg = x ? (x * 90) : ( y > 0 ? 180 : 0)
            return `translate(${x*this.size},${y*this.size}) rotate(${deg})`
        },
        simplePos(){
            let simplePos = {x:0,y:0}
            let {x,y} = this.pos
            let deadZone = 0.2
            if (Math.abs(x) > Math.abs(y)){
                if (x > deadZone)
                    simplePos.x = 1
                else if (x < -deadZone)
                    simplePos.x = -1
            }else{
                if (y > deadZone)
                    simplePos.y = 1
                else if (y < -deadZone)
                    simplePos.y = -1
            }
            return simplePos
        }
    },
    created(){
        this.lastEmit = {}
    },
    watch:{
        simplePos(){
            if (this.lastEmit.x != this.simplePos.x || this.lastEmit.y != this.simplePos.y){
                this.$emit('update-simple',this.simplePos)
                this.lastEmit = {x:this.simplePos.x,y:this.simplePos.y}
            }
        }
    },
    methods:{
        setCenter(e){
            this.holding = true
            this.moveStick(e)
        },
    }
}


</script>

<style>
.touch-control-simple{
    background-color:rgba(50, 220, 147, 0.1);
    width: 100%;
    height: 100%;
    display: block;
}

.touch-control-simple-stick-center{
    fill:rgb(50, 220, 147); 
}
.touch-control-simple-stick-limits{
    fill:rgba(50, 220, 147, 0.3);
    stroke:rgb(50, 220, 147);
    stroke-width:3;
}
.touch-control-simple-stick-handler{
    fill:rgba(255,255,255,.6);
}
.touch-control-simple-direction{
    fill:none;
    stroke-width:1;
    stroke:rgba(255,255,255,.6);
}
</style>
