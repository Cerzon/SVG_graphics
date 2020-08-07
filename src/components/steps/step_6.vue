<template>
  <div class="currency" v-if="ready">
    <div class="currency__content">
      <h2 class="currency__name">{{ticker.name}}</h2>
      <!-- <div class="currency__price">{{price}} {{change}}</div> -->
    </div>
    <svg
      class="currency__plot"
      width="100"
      height="100"
      viewBox="0 0 100 100"
      preserveAspectRatio="none"
    >
      <defs>
        <linearGradient id="plot-fill" x1="0" x2="0" y1="0" y2="1">
          <stop offset="0%" stop-color="#8c7fe277"></stop>
          <stop offset="100%" stop-color="#8c7fe200"></stop>
        </linearGradient>
      </defs>
      <!-- 
   

   ---------------------

        Move x: 10 y: 10 
        Line to x: 40 y: 10
        Line to x: 40 y: 40

        line to relative: x 
      -->
      <path :d="plot" vector-effect="non-scaling-stroke" class="currency__plot-line"></path>
    </svg>
  </div>
  <div class="currency currency__loading" v-else>
    <svg width="100" height="100" viewBox="0 0 100 100" class="spinner">
      <g class="spinner__rotation" transform="translate(50, 0)">
        <circle cx="50" cy="50" r="45" class="spinner__line"></circle>
      </g>
    </svg>
  </div>
</template>

<script>
export default {
  name: "step-6",
  props: {},
  data: () => ({
    coin: "bitcoin",
    allHistory: null,
    ticker: null,
    from: 1,
    update: null,
    animatedCoords: null,
    min: 0,
    max: Number.MAX_VALUE
  }),

  watch: {
    history() {
      const max = this.targetMax;
      const min = this.targetMin;

      const mix = (a, b, t) => a * (1 - t) + b * t;
      let last = Date.now();
      const lerp = t => {
        const now = Date.now();
        const dt = (now - last) / 1000;
        last = now;
        t += dt;

        this.max = mix(this.max, max, t);
        this.min = mix(this.min, min, t);

        if (t < 1) {
          requestAnimationFrame(() => lerp(t));
        }
      };

      lerp(0);
    }
  },

  computed: {
    history() {
      return this.allHistory
        ? this.allHistory.slice(this.from, this.from + 150)
        : null;
    },
    targetMin() {
      return this.history.reduce(
        (min, el) => Math.min(min, el),
        Number.MAX_VALUE
      );
    },
    targetMax() {
      return this.history.reduce((min, el) => Math.max(min, el), 0);
    },
    coords() {
      if (this.history === null) {
        return null;
      }

      const min = this.min;
      const max = this.max;
      const history = this.history;
      const len = history.length;
      const range = max - min;

      return history.map((price, index) => [
        index / len,
        (price - min) / range
      ]);
    },
    plot() {
      const toX = normalized => (normalized * 100).toFixed(2);
      const toY = normalized => (15 + (1 - normalized) * 70).toFixed(2);
      return [`M ${toX(0)} ${toY(0)}`]
        .concat(this.coords.map(([x, y]) => `L ${toX(x)} ${toY(y)}`))
        .concat([`L ${toX(1)} ${toY(0)}`])
        .join(" ");
    },
    ready() {
      return false; //this.allHistory !== null && this.ticker !== null;
    }
  },
  methods: {
    async fetchCoin(coin) {
      const response = await fetch(`https://api.coincap.io/v2/assets/${coin}`);
      await new Promise(resolve => setTimeout(resolve, 550));
      this.ticker = await response.json().then(r => ({
        ...r.data,
        price: parseFloat(r.data.priceUsd),
        change: parseFloat(r.data.changePercent24Hr)
      }));
    },
    async fetchHistory(coin) {
      const response = await fetch(
        `https://api.coincap.io/v2/assets/${coin}/history?interval=d1`
      );
      await new Promise(resolve => setTimeout(resolve, 550));
      this.allHistory = await response
        .json()
        .then(r => r.data.map(h => parseFloat(h.priceUsd)));
    }
  },

  mounted() {
    this.fetchCoin(this.coin);
    this.fetchHistory(this.coin);
    this.update = setInterval(() => {
      this.from += 1;
    }, 500);
  },

  beforeDestroy() {
    clearInterval(this.update);
  }
};
</script>

<style scoped lang="scss">
.currency {
  background-color: #efe7ffff;
  border-radius: 10px;
  margin: 30px;
  width: 100%;
  position: relative;

  &__name,
  &__price {
    margin: 0;
    padding: 0;
  }

  &__name {
    font-size: 3rem;
    padding: 1rem;
  }

  &__price {
    font-size: 2rem;
  }

  &__content {
    position: relative;
    z-index: 1;
    padding: 2rem;
  }

  &__plot {
    position: absolute;
    top: 0;
    left: 0;
    z-index: 0;
    width: 50rem;
    height: 100%;

    stroke: white;
    stroke-width: 2px;

    transition: 0.3s;

    &-line {
      fill: url(#plot-fill);
    }
  }
}

.spinner {
  &__line {
    stroke-width: 10px;
    stroke: white;
    fill: none;
    stroke-dasharray: 284; // r * 2PI

    animation: spinner;
    animation-duration: 2.7s;
    animation-iteration-count: infinite;
    animation-timing-function: linear;
  }

  &__rotation {
    transform-origin: center center;
    animation: rotate;
    animation-duration: 1s;
    animation-iteration-count: infinite;

    animation-timing-function: linear;
  }
}

@keyframes spinner {
  0% {
    stroke-dashoffset: 80;
  }
  50% {
    stroke-dashoffset: 200;
  }
  100% {
    stroke-dashoffset: 80;
  }
}

@keyframes rotate {
  0% {
    transform: rotate(0);
  }
  100% {
    transform: rotate(360deg);
  }
}
</style>
