<template>
  <div class="app">

    <!-- Background glow orbs -->
    <div class="orb orb-1"></div>
    <div class="orb orb-2"></div>

    <header class="header">
      <div class="logo">
        <div class="logo-mark">◈</div>
        <div>
          <div class="logo-text">VAULTEX</div>
          <div class="logo-sub">Crypto Dashboard</div>
        </div>
      </div>

      <div class="portfolio">
        <div class="portfolio-label">PORTAFOLIO TOTAL</div>
        <div class="portfolio-value">${{ fmt(totalValue) }}</div>
        <div class="portfolio-change" :class="avgChange >= 0 ? 'up' : 'down'">
          {{ avgChange >= 0 ? '▲' : '▼' }} {{ Math.abs(avgChange).toFixed(2) }}% hoy
        </div>
      </div>

      <div class="header-right">
        <div class="live-pill">
          <span class="pulse-dot"></span>
          EN VIVO
        </div>
        <div class="clock">{{ clock }}</div>
      </div>
    </header>

    <section class="coins-grid">
      <CoinCard
        v-for="(coin, i) in coins"
        :key="coin.symbol"
        :coin="coin"
        :active="selected === i"
        @click="selected = i"
      />
    </section>

    <section class="bottom-grid">
      <PriceChart :coin="coins[selected]" />
      <StatsPanel :coin="coins[selected]" :coins="coins" />
    </section>

  </div>
</template>

<script>
import CoinCard from './components/CoinCard.vue'
import PriceChart from './components/PriceChart.vue'
import StatsPanel from './components/StatsPanel.vue'

//const USE_LIVE_DATA = true

function genHistory(base) {
  let v = base, arr = []
  for (let i = 0; i < 30; i++) {
    v += (Math.random() - 0.49) * base * 0.011
    arr.push(parseFloat(v.toFixed(2)))
  }
  return arr
}

export default {
  name: 'App',
  components: { CoinCard, PriceChart, StatsPanel },
  data() {
    return {
      selected: 0,
      clock: '',
      coins: [
        { name: 'Bitcoin',   symbol: 'BTC', icon: '₿', color: '#f7931a', price: 67432, change:  2.14, amount: 0.52, history: genHistory(67432) },
        { name: 'Ethereum',  symbol: 'ETH', icon: 'Ξ', color: '#627eea', price:  3521, change: -1.08, amount: 3.20, history: genHistory(3521)  },
        { name: 'Solana',    symbol: 'SOL', icon: '◎', color: '#9945ff', price:   178, change:  5.33, amount: 42,   history: genHistory(178)   },
        { name: 'Avalanche', symbol: 'AVAX',icon: '▲', color: '#e84142', price:  38.4, change: -0.76, amount: 120,  history: genHistory(38.4)  },
      ]
    }
  },
  computed: {
    totalValue() { return this.coins.reduce((s, c) => s + c.price * c.amount, 0) },
    avgChange()  { return this.coins.reduce((s, c) => s + c.change, 0) / this.coins.length },
  },
  methods: {
    fmt(n) {
      if (n >= 1e6) return (n / 1e6).toFixed(2) + 'M'
      if (n >= 1e3) return (n / 1e3).toFixed(2) + 'K'
      return n.toFixed(2)
    },
    tick() {
      this.coins.forEach(c => {
        c.price  = parseFloat((c.price  + (Math.random() - 0.495) * c.price * 0.008).toFixed(2))
        c.change = parseFloat((c.change + (Math.random() - 0.50)  * 0.12).toFixed(2))
        c.history = [...c.history.slice(-29), c.price]
      })
    }
  },
  mounted() {
    const tick = () => this.clock = new Date().toLocaleTimeString('es-CO', { hour12: false })
    tick(); setInterval(tick, 1000)
    setInterval(this.tick, 1800)
  }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Space+Mono:wght@400;700&family=Syne:wght@500;700;800&display=swap');

*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

:root {
  --bg: #07070f;
  --text: #e2e2f0;
}

body {
  background: var(--bg);
  color: var(--text);
  font-family: 'Syne', sans-serif;
  min-height: 100vh;
  overflow-x: hidden;
}

.app {
  max-width: 1280px;
  margin: 0 auto;
  padding: 0 28px 60px;
  position: relative;
}

/* Orbs */
.orb {
  position: fixed;
  border-radius: 50%;
  filter: blur(100px);
  pointer-events: none;
  z-index: 0;
}
.orb-1 { width: 500px; height: 500px; background: #3b2fff18; top: -100px; left: -100px; }
.orb-2 { width: 400px; height: 400px; background: #ff6b2b12; bottom: 0; right: -80px; }

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 32px 0 36px;
  border-bottom: 1px solid #ffffff09;
  position: relative; z-index: 1;
}

.logo { display: flex; align-items: center; gap: 14px; }
.logo-mark { font-size: 28px; color: #7c6fff; line-height: 1; }
.logo-text { font-size: 21px; font-weight: 800; letter-spacing: 5px; line-height: 1; }
.logo-sub { font-family: 'Space Mono', monospace; font-size: 10px; color: #33334a; letter-spacing: 2px; margin-top: 3px; }

.portfolio { text-align: center; }
.portfolio-label { font-family: 'Space Mono', monospace; font-size: 10px; letter-spacing: 3px; color: #33334a; margin-bottom: 6px; }
.portfolio-value { font-size: 36px; font-weight: 800; letter-spacing: -1px; }
.portfolio-change { font-family: 'Space Mono', monospace; font-size: 13px; margin-top: 5px; }

.header-right { display: flex; flex-direction: column; align-items: flex-end; gap: 8px; }
.live-pill {
  display: flex; align-items: center; gap: 7px;
  font-family: 'Space Mono', monospace; font-size: 10px; letter-spacing: 2px;
  color: #00e676; border: 1px solid #00e67622; padding: 5px 14px; border-radius: 20px;
  background: #00e67608;
}
.pulse-dot {
  width: 6px; height: 6px; border-radius: 50%; background: #00e676;
  animation: pulse 1.4s ease-in-out infinite;
}
@keyframes pulse { 0%,100%{opacity:1;transform:scale(1)} 50%{opacity:.3;transform:scale(.6)} }
.clock { font-family: 'Space Mono', monospace; font-size: 13px; color: #33334a; }

/* Grids */
.coins-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 16px;
  margin: 28px 0 20px;
  position: relative; z-index: 1;
}

.bottom-grid {
  display: grid;
  grid-template-columns: 1fr 320px;
  gap: 18px;
  position: relative; z-index: 1;
}

.up { color: #00e676; }
.down { color: #ff4560; }

@media (max-width: 960px) {
  .coins-grid { grid-template-columns: repeat(2, 1fr); }
  .bottom-grid { grid-template-columns: 1fr; }
  .header { flex-direction: column; gap: 20px; text-align: center; }
  .header-right { align-items: center; }
}
</style>