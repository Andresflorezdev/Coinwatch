<template>
  <div class="coin-card" :class="{ active }" @click="$emit('click')" :style="{ '--accent': coin.color }">
    <div class="card-top">
      <span class="coin-icon">{{ coin.icon }}</span>
      <span class="coin-symbol">{{ coin.symbol }}</span>
    </div>
    <div class="coin-name">{{ coin.name }}</div>
    <div class="coin-price">${{ fmt(coin.price) }}</div>
    <div class="coin-change" :class="coin.change >= 0 ? 'up' : 'down'">
      {{ coin.change >= 0 ? '▲' : '▼' }} {{ Math.abs(coin.change).toFixed(2) }}%
    </div>
    <div class="sparkline">
      <svg viewBox="0 0 100 32" preserveAspectRatio="none">
        <defs>
          <linearGradient :id="'g' + coin.symbol" x1="0" y1="0" x2="0" y2="1">
            <stop offset="0%" :stop-color="coin.color" stop-opacity="0.3"/>
            <stop offset="100%" :stop-color="coin.color" stop-opacity="0"/>
          </linearGradient>
        </defs>
        <polygon :points="areaPoints(coin.history)" :fill="'url(#g' + coin.symbol + ')'" />
        <polyline :points="linePoints(coin.history)" fill="none" :stroke="coin.color" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"/>
      </svg>
    </div>
  </div>
</template>

<script>
export default {
  name: 'CoinCard',
  props: {
    coin: Object,
    active: Boolean
  },
  emits: ['click'],
  methods: {
    fmt(n) {
      if (n >= 1e6) return (n / 1e6).toFixed(2) + 'M'
      if (n >= 1e3) return (n / 1e3).toFixed(2) + 'K'
      return n.toFixed(2)
    },
    linePoints(h) {
      const mn = Math.min(...h), mx = Math.max(...h), range = mx - mn || 1
      return h.map((v, i) => `${(i / (h.length - 1)) * 100},${30 - ((v - mn) / range) * 26}`).join(' ')
    },
    areaPoints(h) {
      const mn = Math.min(...h), mx = Math.max(...h), range = mx - mn || 1
      const pts = h.map((v, i) => `${(i / (h.length - 1)) * 100},${30 - ((v - mn) / range) * 26}`)
      return `0,30 ${pts.join(' ')} 100,30`
    }
  }
}
</script>

<style scoped>
.coin-card {
  background: #0e0e1c;
  border: 1px solid #ffffff0c;
  border-radius: 20px;
  padding: 22px 20px 16px;
  cursor: pointer;
  transition: transform 0.22s ease, border-color 0.22s ease, box-shadow 0.22s ease;
  position: relative;
  overflow: hidden;
}
.coin-card::after {
  content: '';
  position: absolute;
  inset: 0;
  background: radial-gradient(ellipse at top left, var(--accent)18, transparent 65%);
  opacity: 0;
  transition: opacity 0.3s;
}
.coin-card:hover, .coin-card.active {
  border-color: var(--accent);
  transform: translateY(-4px);
  box-shadow: 0 8px 32px var(--accent)22;
}
.coin-card:hover::after, .coin-card.active::after { opacity: 1; }

.card-top { display: flex; justify-content: space-between; align-items: center; margin-bottom: 8px; }
.coin-icon { font-size: 26px; }
.coin-symbol {
  font-family: 'Space Mono', monospace;
  font-size: 10px; letter-spacing: 2px;
  color: #55556e;
  background: #ffffff08;
  padding: 3px 8px; border-radius: 8px;
}
.coin-name { font-size: 13px; color: #55556e; margin-bottom: 10px; font-weight: 600; letter-spacing: 0.5px; }
.coin-price { font-size: 22px; font-weight: 800; letter-spacing: -0.5px; margin-bottom: 5px; }
.coin-change { font-family: 'Space Mono', monospace; font-size: 12px; margin-bottom: 14px; }
.up { color: #00e676; }
.down { color: #ff4560; }
.sparkline { height: 40px; }
.sparkline svg { width: 100%; height: 100%; }
</style>
