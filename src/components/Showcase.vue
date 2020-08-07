<template>
  <div class="showcase">
    <h2>Пример {{this.step}}</h2>
    <component :is="`step-${this.step}`" :class="`showcase__step showcase__step--${this.step}`"/>
  </div>
</template>

<script>
export default {
  name: "showcase",
  props: {
    step: Number
  }
};
</script>

<style>
</style>
