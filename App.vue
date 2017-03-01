<template>
    <div id="box">
        <Fullscreen/>
        <!-- <Move @update="updatePos"/> -->
        <Simple @update-simple="updatePos1"/>
        <Simple @update-simple="updatePos2"/>
        <!-- <Shoot @update="updatePos"/> -->
    </div>
</template>

<script>
import Shoot from 'Shoot'
import Simple from 'Simple'
import Move from 'Move'
import Fullscreen from 'Fullscreen'


import deepstream from 'deepstream.io-client-js'
let client = deepstream('192.168.1.8:6020').login()
let record = client.record.getRecord('positions')

export default {
    data(){
        return{
            pos:{x:0,y:0}
        }
    },
    methods:{
        updatePos1(pos){
            record.set('player1',{x:pos.x,y:pos.y})
        },
        updatePos2(pos){
            record.set('player2',{x:pos.x,y:pos.y})
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
