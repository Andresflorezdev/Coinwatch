<template>
  <div class="stats-panel">
    <div class="panel-section">
      <div class="section-label">MARKET STATS</div>
      <div v-for="stat in stats" :key="stat.label" class="stat-row">
        <span class="stat-label">{{ stat.label }}</span>
        <span class="stat-val" :style="{ color: stat.color || '#e8e8f0' }">{{ stat.value }}</span>
      </div>
    </div>

    <div class="panel-section">
      <div class="section-label">HOLDINGS</div>
      <div v-for="coin in coins" :key="coin.symbol" class="holding-row" :style="{ '--c': coin.color }">
        <div class="h-left">
          <div class="h-dot" :style="{ background: coin.color }"></div>
          <div>
            <div class="h-name">{{ coin.name }}</div>
            <div class="h-amount">{{ coin.amount }} {{ coin.symbol }}</div>
          </div>
        </div>
        <div class="h-right">
          <div class="h-value">${{ fmt(coin.price * coin.amount) }}</div>
          <div class="h-bar-wrap"><div class="h-bar" :style="{ width: barWidth(coin) + '%', background: coin.color }"></div></div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'StatsPanel',
  props: { coin: Object, coins: Array },
  computed: {
    stats() {
      const c = this.coin
      return [
        { label: 'Market Cap',  value: '$' + this.fmt(c.price * 19_000_000) },
        { label: '24h Volume',  value: '$' + this.fmt(c.price * 420_000) },
        { label: '24h High',    value: '$' + this.fmt(c.price * 1.042) },
        { label: '24h Low',     value: '$' + this.fmt(c.price * 0.961) },
        { label: 'Variación',   value: (c.change >= 0 ? '+' : '') + c.change.toFixed(2) + '%', color: c.change >= 0 ? '#00e676' : '#ff4560' },
      ]
    },
    totalHoldings() {
      return this.coins.reduce((s, c) => s + c.price * c.amount, 0)
    }
  },
  methods: {
    fmt(n) {
      if (n >= 1e6) return (n / 1e6).toFixed(2) + 'M'
      if (n >= 1e3) return (n / 1e3).toFixed(2) + 'K'
      return n.toFixed(2)
    },
    barWidth(coin) {
      return Math.round((coin.price * coin.amount / this.totalHoldings) * 100)
    }
  }
}
</script>

<style scoped>
.stats-panel {
  background: #0e0e1c;
  border: 1px solid #ffffff0c;
  border-radius: 20px;
  padding: 28px;
  display: flex;
  flex-direction: column;
  gap: 32px;
  height: 100%;
}
.section-label {
  font-family: 'Space Mono', monospace;
  font-size: 10px; letter-spacing: 3px; color: #33334a;
  margin-bottom: 16px;
}
.stat-row {
  display: flex; justify-content: space-between; align-items: center;
  padding: 10px 0; border-bottom: 1px solid #ffffff07;
}
.stat-label { font-size: 13px; color: #44445a; }
.stat-val { font-family: 'Space Mono', monospace; font-size: 12px; }

.holding-row {
  padding: 12px 0; border-bottom: 1px solid #ffffff07;
}
.h-left { display: flex; align-items: center; gap: 12px; margin-bottom: 8px; }
.h-dot { width: 8px; height: 8px; border-radius: 50%; flex-shrink: 0; }
.h-name { font-size: 14px; font-weight: 700; }
.h-amount { font-family: 'Space Mono', monospace; font-size: 10px; color: #44445a; margin-top: 2px; }
.h-right {}
.h-value { font-family: 'Space Mono', monospace; font-size: 13px; text-align: right; margin-bottom: 6px; }
.h-bar-wrap { background: #ffffff08; border-radius: 4px; height: 3px; }
.h-bar { height: 100%; border-radius: 4px; transition: width 0.8s ease; }
</style>
