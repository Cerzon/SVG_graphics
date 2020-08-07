<template>
  <div id="app" :class="`app ${this.debug && 'app--debug'}`">
    <button
      class="app__control app__control--debug control"
      :disabled="!this.allowNext"
      @click="() => this.debug = !this.debug"
    >Debug</button>
    <h1 class="app__title">Веб-графика с SVG</h1>
    <h2 class="app__subtitle">Пример {{this.step}}</h2>
    <component :is="`step-${this.step}`" :class="`app__step app__step--${this.step}`"/>
    <div class="app__controls">
      <button
        class="app__control app__control--previous control"
        :disabled="!this.allowPrev"
        @click="this.prev"
      >Предыдущий</button>
      <button
        class="app__control app__control--next control"
        :disabled="!this.allowNext"
        @click="this.next"
      >Следующий</button>
    </div>
  </div>
</template>

<script>
const STEPS = 10;
const STEP_CACHE = "geekbrain-showcase-step";

export default {
  name: "App",
  components: Array(STEPS)
    .fill(0)
    .reduce((dict, _, index) => {
      dict[`step-${index + 1}`] = () =>
        import(`./components/steps/step_${index + 1}`);
      return dict;
    }, {}),
  data: () => ({
    step: parseInt(localStorage.getItem(STEP_CACHE), 10) || 1,
    debug: false
  }),
  computed: {
    allowNext() {
      return this.step < STEPS;
    },
    allowPrev() {
      return this.step > 1;
    }
  },
  watch: {
    step() {
      localStorage.setItem(STEP_CACHE, this.step);
    }
  },
  methods: {
    next() {
      this.step++;
    },
    prev() {
      this.step--;
    }
  }
};
</script>

<style lang="scss">
html {
  font-size: 8px;
}

body {
  padding: 0;
  margin: 0;

  font-size: 2rem;
}

.app {
  font-family: "Montserrat", serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;

  padding: 4rem;

  position: fixed;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;

  display: flex;
  flex-direction: column;

  &--debug {
    svg {
      box-shadow: 0 0 5px 2px #8c7fe2ff;
    }
  }

  &__title {
    font-weight: 300;
    font-size: 4.5rem;
    background-color: #e1fff6ff;
    display: inline-block;
    align-self: center;

    margin: 0;
    margin-bottom: 0.5em;
  }

  &__subtitle {
    font-weight: 300;
    font-size: 2.5rem;

    margin: 0;
    margin-bottom: 3rem;
  }

  &__step {
    box-shadow: 
      3px 3px 80px 0 rgba(116, 106, 119, 0.14),
      0 5px 20px 0 rgba(221, 196, 234, 0.32);
    overflow: hidden;
    flex-grow: 1;
    display: block;

    margin-bottom: auto;
  }

  &__controls {
    padding-top: 3rem;
  }
  
  &__control {
    margin: 0 0.5rem;

    &--debug {
      position: absolute;
      top: 1rem; right: 1em;
      margin: 0;
    }
  }
}

.control {
  $fg: #e1fff6ff;
  $bg: #efe7ffff;
  background-color: $fg;
  box-shadow: 4px 4px 0 0 $bg;
  font-weight: 300;
  border: 2px solid $fg;
  white-space: nowrap;
  vertical-align: center;
  display: inline-block;
  padding: 15px 20px;
  user-select: none;
  text-align: center;

  text-transform: uppercase;
  letter-spacing: 0.05em;

  &:hover:not([disabled]) {
    $fg: saturate(lighten($fg, 5%), 5%);
    $bg: saturate(darken($bg, 5%), 5%);
    background-color: $fg;
    box-shadow: 2px 2px 0 0 $bg;
  }

  &:active:not([disabled]) {
    $bg: saturate(darken($bg, 5%), 5%);
    background-color: white;
    box-shadow: none;
    border-color: $bg

  }

  &:focus {
    outline: 0;
  }

  &[disabled] {
    opacity: 0.6;
    cursor: not-allowed;
  }
}
</style>
