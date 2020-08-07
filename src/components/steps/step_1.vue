<template>
  <div>
    <svg width="100" height="100" preserveAspectRatio="none">
      <rect
        v-for="(pillar, id) in pillars"
        :key="id"
        :x="pillar.x"
        :y="pillar.y"
        :width="pillar.width"
        :height="pillar.height"
        fill="#efe7ffff"
      ></rect>
      <!-- <circle cx="50" cy="50" r="35" fill="#e1fff6ff"></circle>
      <line x1="40" y1="40" x2="60" y2="60" stroke="#8c7fe2ff" stroke-width="5"></line>
      <line x1="40" y1="60" x2="60" y2="40" stroke="#8c7fe2ff" stroke-width="5"></line>-->
    </svg>
  </div>
</template>

<script>
export default {
  name: "step-1",
  props: {},
  data: () => ({
    values: Array(10)
      .fill(0)
      .map(Math.random)
  }),

  computed: {
    pillars() {
      const width = 100 / this.values.length;

      return this.values.map((v, index) => {
        const height = v * 100;
        return {
          x: index * width,
          y: 100 - height,
          height,
          width
        };
      });
    }
  }
};
</script>

<style scoped>
svg {
  width: 100%;
}
</style>
