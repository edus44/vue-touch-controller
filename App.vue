<template>
    <div id="box">
        <Fullscreen/>
        <!-- <Move @update="updatePos"/> -->
        <Simple @update="updatePos"/>
        <!-- <Shoot @update="updatePos"/> -->
    </div>
</template>

<script>
import Shoot from 'Shoot'
import Simple from 'Simple'
import Move from 'Move'
import Fullscreen from 'Fullscreen'

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
    components: {Shoot,Move,Fullscreen,Simple}
}
</script>

<style>
html,body{
    height: 100%;
    margin: 0;
    padding: 0;
    min-height: 100%;
    overflow: hidden;
    background-color: #111;
}
body{
    cursor: default;
    display: flex;
}
#box{
    flex:1;
    display: flex;
    flex-wrap:wrap;

}
#box>svg{
    width: 50%;
    /*height: 50%;*/
}
</style>
