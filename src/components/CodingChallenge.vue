<template>
  <div>
    <h1>Options Profit Calculator</h1>
    <form @submit.prevent="calculateStrategy">
      <div v-for="(option, index) in options" :key="index" class="option-inputs">
        <label>
          Type:
          <select v-model="option.type">
            <option>call</option>
            <option>put</option>
          </select>
        </label>
        <label>
          Strike Price:
          <input type="number" v-model.number="option.strike" required>
        </label>
        <label>
          Premium:
          <input type="number" v-model.number="option.premium" required>
        </label>
        <label>
          Quantity:
          <input type="number" v-model.number="option.quantity" required>
        </label>
        <button type="button" @click="removeOption(index)">Remove</button>
      </div>
      <button type="button" @click="addOption">Add Option</button>
      <button type="submit">Calculate</button>
    </form>
    <div v-if="graphData">
      <p>Max Profit: {{ maxProfit }}</p>
      <p>Max Loss: {{ maxLoss }}</p>
      <p>Break Even Points: {{ breakEvenPoints.join(', ') }}</p>
      <LineChart :data="graphData" />
    </div>
  </div>
</template>

<script>
import { Line as LineChart } from 'vue-chartjs'
import { Chart as ChartJS, Title, Tooltip, Legend, LineElement, PointElement, LinearScale, CategoryScale } from 'chart.js'

ChartJS.register(Title, Tooltip, Legend, LineElement, PointElement, LinearScale, CategoryScale)

export default {
  name: 'OptionsProfitCalculator',
  components: {
    LineChart
  },
  data() {
    return {
      options: [{
        type: 'call',
        strike: 0,
        premium: 0,
        quantity: 1
      }],
      graphData: null,
      maxProfit: null,
      maxLoss: null,
      breakEvenPoints: []
    }
  },
  methods: {
    addOption() {
      this.options.push({
        type: 'call',
        strike: 0,
        premium: 0,
        quantity: 1
      })
    },
    removeOption(index) {
      this.options.splice(index, 1)
    },
    calculateStrategy() {
      const underlyingPrices = Array.from({ length: 101 }, (_, i) => i)
      const profits = underlyingPrices.map(price => this.calculateProfit(price))

      this.graphData = {
        labels: underlyingPrices,
        datasets: [
          {
            label: 'Profit/Loss',
            data: profits,
            borderColor: 'rgb(75, 192, 192)',
            tension: 0.1,
            fill: false
          }
        ]
      }

      this.calculateMaxProfitLoss(profits)
      this.calculateBreakEvenPoints(profits, underlyingPrices)
    },
    calculateProfit(price) {
      return this.options.reduce((acc, option) => {
        const { type, strike, premium, quantity } = option
        let profit = 0
        if (type === 'call') {
          profit = Math.max(price - strike, 0) - premium
        } else if (type === 'put') {
          profit = Math.max(strike - price, 0) - premium
        }
        return acc + profit * quantity
      }, 0)
    },
    calculateMaxProfitLoss(profits) {
      this.maxProfit = Math.max(...profits)
      this.maxLoss = Math.min(...profits)
    },
    calculateBreakEvenPoints(profits, underlyingPrices) {
      this.breakEvenPoints = []
      for (let i = 1; i < profits.length; i++) {
        if ((profits[i - 1] < 0 && profits[i] >= 0) || (profits[i - 1] > 0 && profits[i] <= 0)) {
          const x1 = underlyingPrices[i - 1]
          const x2 = underlyingPrices[i]
          const y1 = profits[i - 1]
          const y2 = profits[i]
          const breakEven = x1 - y1 * (x2 - x1) / (y2 - y1)
          this.breakEvenPoints.push(breakEven)
        }
      }
    }
  }
}
</script>

<style scoped>
form {
  margin-bottom: 20px;
}
.option-inputs {
  margin-bottom: 10px;
}
label {
  margin-right: 10px;
}
button {
  margin-right: 10px;
}
</style>
