<template>
    <div id="box">
        {{ pos.x }}<br>
        {{ pos.y }}<br>
        <Shoot @update="updatePos"/>
        <Circle2 @update="updatePos"/>
    </div>
</template>

<script>
import Shoot from 'Shoot'
import Circle from 'Circle'

export default {
    data(){
        return{
            pos:{x:0,y:0}
        }
    },
    methods:{
        updatePos(pos){
            this.pos = pos
        }
    },
    components: {Shoot,Circle2:Circle}
}
</script>

<style>
html,body{
    height: 100%;
    margin: 0;
    padding: 0;
    min-height: 100%;
}
body{
    display: flex;
    /*align-items:center;*/
    /*justify-content: center;*/
}
#box{
    box-shadow:0 0 10px rgba(0,0,0,.1);
    /*flex:1;*/
    width: 400px;
    height: 400px;
}
</style>
