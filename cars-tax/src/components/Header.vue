<template>
  <header>
    <img src="../assets/jeep.gif" alt="Wheel Icon" class="no-print" />
  </header>
</template>

<script>
export default {
  name: "Header",
};
</script>

<style scoped>
header {
  width: 50vw;
  max-width: 350px;
  margin: 5px auto;
}
img {
  width: 100%;
}

@keyframes wheelMove {
  from {
    transform: rotateZ(0deg);
  }
  to {
    transform: rotateZ(360deg);
  }
}
</style>
