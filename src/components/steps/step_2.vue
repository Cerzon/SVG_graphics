<template>
  <svg version="1.1" baseProfile="full" width="100" height="100" xmlns="http://www.w3.org/2000/svg">
    <circle cx="50" cy="50" r="35" class="rect"></circle>
    <line x1="40" y1="40" x2="60" y2="60" stroke="#8c7fe2ff" stroke-width="5"></line>
    <line x1="40" y1="60" x2="60" y2="40" stroke="#8c7fe2ff" stroke-width="5"></line>
  </svg>
</template>

<script>
export default {
  name: "step-2",
  props: {}
};
</script>

<style scoped lang="scss"> 
svg {
  width: 70%;
}

.rect {
  fill: #efe7ffff;
  width: 90px;
  height: 90px;

  &:hover {
    fill: saturate(darken(#efe7ffff, 5%), 5%);
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
