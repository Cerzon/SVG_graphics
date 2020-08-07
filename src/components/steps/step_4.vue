<template>
  <svg
    version="1.1"
    baseProfile="full"
    width="100"
    height="100"
    viewBox="0 0 100 100"
    preserveAspectRatio="none"
    xmlns="http://www.w3.org/2000/svg"
  >
    <rect x="5" y="5" rx="15" ry="15" class="rect"></rect>
    <circle cx="50" cy="50" r="35" class="circle"></circle>
    <line x1="20" y1="20" x2="80" y2="80" class="line"></line>
    <line x1="20" y1="80" x2="80" y2="20" class="line"></line>
  </svg>
</template>

<script>
export default {
  name: "step-2",
  props: {}
};
</script>

<style scoped>
svg {
  width: 100%;
}

.rect {
  fill: #efe7ffff;
  width: 90px;
  height: 90px;

  animation: rect;
  animation-iteration-count: infinite;
  animation-duration: 1s;
}

@keyframes rect {
  0% {
    rx: 45;
    ry: 45;
  }
  50% {
    rx: 15;
    ry: 15;
  }
  100% {
    rx: 45;
    ry: 45;
  }
}

.circle {
  fill: #e1fff6ff;
}

.line {
  stroke: #8c7fe2ff;
  stroke-width: 5;
}
</style>
