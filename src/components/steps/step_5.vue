<template>
  <div class="currency" v-if="ready">
    <div class="currency__content">
      <h2 class="currency__name">{{ticker.name}}</h2>
      <div class="currency__price">{{price}} {{change}}</div>
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
      <path
        :d="plot"
        vector-effect="non-scaling-stroke"
        class="currency__plot-line"
        fill="url(#plot-fill)"
      ></path>
    </svg>
  </div>
  <div class="currency currency__loading" v-else>
    <svg width="100" height="100" viewBox="0 0 100 100" class="spinner">
      <g class="spinner__rotation">
        <circle cx="50" cy="50" r="45" class="spinner__line"></circle>
      </g>
    </svg>
  </div>
</template>

<script>
export default {
  name: "step-5",
  data: () => ({
    coin: "bitcoin",
    allHistory: null,
    ticker: null,
    from: 1,
    update: null,
    animatedCoords: null,
    min: 0,
    max: 0
  }),
  watch: {
    targetMin(min) {
      // this.min = min;
      // return;

      const mix = (a, b, t) => a * (1 - t) + b * t;
      let last = Date.now();
      const lerp = t => {
        const now = Date.now();
        const dt = (now - last) / 1000;
        last = now;
        t += dt;

        this.min = mix(this.min, min, t);

        if (t < 1) {
          requestAnimationFrame(() => lerp(t));
        }
      };

      lerp(0);
    },
    targetMax(max) {
      // this.max = max;
      // return;

      const mix = (a, b, t) => a * (1 - t) + b * t;
      let last = Date.now();
      const lerp = t => {
        const now = Date.now();
        const dt = (now - last) / 1000;
        last = now;
        t += dt;

        this.max = mix(this.max, max, t);

        if (t < 1) {
          requestAnimationFrame(() => lerp(t));
        }
      };

      lerp(0);
    },
    coords(coords) {
      this.animatedCoords = coords;
      return;
      // if (coords === null) {
      //   return;
      // }

      // if (this.animatedCoords === null) {
      //   this.animatedCoords = coords;
      //   return;
      // }

      // let last = Date.now();
      // const lerp = t => {
      //   const now = Date.now();
      //   const dt = (now - last) / 1000;
      //   last = now;
      //   t += dt;

      //   const mix = (a, b, t) => a * (1 - t) + b * t;

      //   const current = this.animatedCoords;
      //   const target = coords;

      //   this.animatedCoords = target.map(([tx, ty], i) => [
      //     mix(current[i][0], tx, t),
      //     mix(current[i][1], ty, t)
      //   ]);

      //   if (t < 1) {
      //     requestAnimationFrame(() => lerp(t));
      //   }
      // };

      // lerp(0);
    }
  },
  computed: {
    history() {
      return this.allHistory
        ? this.allHistory.slice(this.from, this.from + 60)
        : null;
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
      if (this.animatedCoords === null) {
        return "";
      }

      const a = this.animatedCoords;

      const x = rx => (rx * 100).toFixed(2);
      const y = ry => (5 + ry * 90).toFixed(2);

      const p = [
        ["M", -0.5, 1.5],
        ["L", -0.5, a[0][1]],
        ...a.map(([x, y]) => ["L", x, y]),
        ["L", 1.5, a[a.length - 1][1]],
        ["L", 1.5, 1.5]
      ];

      return p
        .map(([action, px, py]) => [action, x(px), y(py)].join(" "))
        .join(" ");

      // const min = this.min;
      // const max = this.max;
      // const history = this.history;
      // const len = history.length;
      // const step = (100 / len).toFixed(2);
      // const range = max - min;
      // const x = index => (step * index).toFixed(2);
      // const y = price => (5 + ((price - min) / range) * 90).toFixed(2);

      // return [
      //   `M  0, 100`,
      //   `L  ${x(-5)} ${y(history[0])}`,
      //   ...history.map((price, index) => `L ${x(index)} ${y(price)}`),
      //   `L ${x(len + 5)} ${y(history[len - 1])}`,
      //   `L ${x(len + 5)} 100`,
      //   "Z"
      // ].join(" ");
    },
    targetMin() {
      return this.history
        ? this.history.reduce((min, next) => (min < next ? min : next))
        : 0;
    },
    targetMax() {
      return this.history
        ? this.history.reduce((max, next) => (max > next ? max : next))
        : 0;
    },
    ready() {
      // return false;
      return !(this.ticker == null || this.history == null);
    },
    price() {
      return "$" + this.ticker.price.toLocaleString();
    },
    up() {
      return this.ticker.change > 0;
    },
    change() {
      return this.ticker.change.toFixed(2) + "%";
    }
  },
  props: {},
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
    width: 100%;
    height: 100%;

    stroke: white;
    stroke-width: 2px;

    transition: 0.3s;
  }
}

.spinner {
  &__line {
    stroke-width: 5px;
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
