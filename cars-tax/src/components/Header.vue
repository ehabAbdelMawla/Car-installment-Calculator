<template>
 <header>
      <img src="../assets/wheel.png" alt="Wheel Icon"/>
 </header>
</template>

<script>
export default {
    name:"Header"
}
</script>

<style scoped>
header{
 width: 50vw;
 max-width: 350px;
 margin: 5px auto;
}
img{
    width:100%;
    animation-name: wheelMove;
    animation-duration: 9s;
    animation-iteration-count: infinite;
    animation-fill-mode: both ;
}

@keyframes wheelMove {
    from{
        transform: rotateZ(0deg);
    }
    to{
       transform: rotateZ(360deg);
    }
    
}

</style>