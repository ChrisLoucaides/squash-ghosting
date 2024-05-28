<template>
  <div id="line-drawing">
    <svg width="480" height="669" viewBox="0 0 160 223" xmlns="http://www.w3.org/2000/svg">
      <rect x="1" y="1" width="158" height="221" fill="none" stroke="red"/>
      <line x1="1" y1="125" x2="159" y2="125" stroke="red"/>
      <line x1="80" y1="125" x2="80" y2="222" stroke="red" />
      <line x1="120" y1="125" x2="120" y2="160" stroke="red" />
      <line x1="40" y1="125" x2="40" y2="160" stroke="red" />
      <line x1="1" y1="160" x2="40.5" y2="160" stroke="red"/>
      <line x1="119.5" y1="160" x2="159" y2="160" stroke="red"/>
    </svg>
  </div>
</template>


<script>
export default {
  name: "SquashCourt"
}
</script>

<style scoped>
#line-drawing {
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>