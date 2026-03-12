<template>
  <div class="chart-wrap">
    <div class="chart-header">
      <div class="chart-info">
        <div class="chart-name">{{ coin.name }}</div>
        <div class="chart-sub">Últimos 30 ticks · en vivo</div>
      </div>
      <div class="chart-right">
        <div class="chart-price" :style="{ color: coin.color }">${{ fmt(coin.price) }}</div>
        <div class="chart-change" :class="coin.change >= 0 ? 'up' : 'down'">
          {{ coin.change >= 0 ? '+' : '' }}{{ coin.change.toFixed(2) }}%
        </div>
      </div>
    </div>
    <div class="canvas-wrap">
      <canvas ref="canvas"></canvas>
    </div>
  </div>
</template>

<script>
import { markRaw } from 'vue'
import { Chart, LineController, LineElement, PointElement, LinearScale, CategoryScale, Filler, Tooltip } from 'chart.js'
Chart.register(LineController, LineElement, PointElement, LinearScale, CategoryScale, Filler, Tooltip)

export default {
  name: 'PriceChart',
  props: { coin: Object },
  data() { return { chart: null } },
  methods: {
    fmt(n) {
      if (n >= 1e6) return (n / 1e6).toFixed(2) + 'M'
      if (n >= 1e3) return (n / 1e3).toFixed(2) + 'K'
      return n.toFixed(2)
    },
    build() {
      // Destroy any existing chart on the canvas
      const existing = Chart.getChart(this.$refs.canvas)
      if (existing) existing.destroy()
      if (this.chart) { this.chart.destroy(); this.chart = null }

      const ctx = this.$refs.canvas.getContext('2d')
      const g = ctx.createLinearGradient(0, 0, 0, 260)
      // Use coin color directly (raw string, not reactive)
      const color = String(this.coin.color)
      g.addColorStop(0, color + '44')
      g.addColorStop(1, color + '00')

      // Pass a plain copy of history to avoid Vue Proxy conflicts with Chart.js
      const historyData = Array.from(this.coin.history).map(Number)
      const labels = historyData.map((_, i) => i + 1)

      // markRaw prevents Vue from making the chart instance reactive
      this.chart = markRaw(new Chart(ctx, {
        type: 'line',
        data: {
          labels,
          datasets: [{
            data: historyData,
            borderColor: color,
            borderWidth: 2.5,
            backgroundColor: g,
            fill: true,
            tension: 0.45,
            pointRadius: 0,
            pointHoverRadius: 6,
            pointHoverBackgroundColor: '#fff',
            pointHoverBorderColor: color,
            pointHoverBorderWidth: 2,
          }]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          animation: { duration: 500 },
          plugins: {
            legend: { display: false },
            tooltip: {
              backgroundColor: '#12121f',
              borderColor: color,
              borderWidth: 1,
              padding: 10,
              titleColor: '#666',
              bodyColor: '#fff',
              bodyFont: { family: 'Space Mono' },
              callbacks: { label: c => ' $' + this.fmt(c.raw) }
            }
          },
          scales: {
            x: { display: false },
            y: {
              display: true,
              position: 'right',
              grid: { color: '#ffffff08' },
              ticks: {
                color: '#444',
                font: { family: 'Space Mono', size: 10 },
                callback: v => '$' + this.fmt(v)
              }
            }
          }
        }
      }))
    },
    update() {
      if (!this.chart) return
      // Pass plain copy to avoid Vue Proxy issues
      this.chart.data.datasets[0].data = Array.from(this.coin.history).map(Number)
      this.chart.update('none')
    }
  },
  watch: {
    'coin.symbol'() { this.$nextTick(() => this.build()) },
    'coin.history'() { this.update() }
  },
  mounted() { this.$nextTick(() => this.build()) },
  beforeUnmount() { if (this.chart) this.chart.destroy() }
}
</script>

<style scoped>
.chart-wrap {
  background: #0e0e1c;
  border: 1px solid #ffffff0c;
  border-radius: 20px;
  padding: 28px 28px 20px;
  height: 100%;
  display: flex;
  flex-direction: column;
}
.chart-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 24px;
}
.chart-name { font-size: 22px; font-weight: 800; letter-spacing: -0.5px; }
.chart-sub { font-family: 'Space Mono', monospace; font-size: 11px; color: #44445a; margin-top: 4px; }
.chart-right { text-align: right; }
.chart-price { font-family: 'Space Mono', monospace; font-size: 20px; font-weight: 700; }
.chart-change { font-family: 'Space Mono', monospace; font-size: 12px; margin-top: 4px; }
.up { color: #00e676; }
.down { color: #ff4560; }
.canvas-wrap { flex: 1; position: relative; min-height: 260px; }
canvas { position: absolute; inset: 0; width: 100% !important; height: 100% !important; }
</style>
