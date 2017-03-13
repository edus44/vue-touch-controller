<template>
    <div id="box">
        <Fullscreen/>
        <!-- <Move @update="updatePos"/> -->
        <Simple @update-simple="updatePos"/>
        <!-- <Simple @update-simple="updatePos2"/> -->
        <!-- <Shoot @update="updatePos"/> -->
    </div>
</template>

<script>
import Shoot from 'Shoot'
import Simple from 'Simple'
import Move from 'Move'
import Fullscreen from 'Fullscreen'
import deepstream from 'deepstream.io-client-js'

let params = new URLSearchParams(window.location.search)
let username = params.get('name')
let session = 1

let client = deepstream('192.168.1.6:6020').login({username:username})

client.on('error',err=>{
    console.log('error',err)
})


export default {
    data(){
        return{
            pos:{x:0,y:0}
        }
    },
    methods:{
        updatePos(pos){
            client.event.emit(`session:${session}:player-dir`,{
                dir:getDir(pos),
                id:username
            })
        }
    },
    components: {Shoot,Move,Fullscreen,Simple}
}

function getDir(pos){

    let dir = 0
    if (pos.y==-1)      dir = 'UP' 
    else if (pos.x==1)  dir = 'RIGHT'
    else if (pos.y==1)  dir = 'DOWN' 
    else if (pos.x==-1) dir = 'LEFT'

    return dir
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
    /*width: 50%;*/
    /*height: 50%;*/
}
</style>
