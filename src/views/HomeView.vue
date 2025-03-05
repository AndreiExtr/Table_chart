<template>
  <div class="home">
    <table class="home__table">
      <thead>
        <tr class="header">
          <th class="col-width-1 col-color-1">Показатель</th>
          <th class="col-width-2 col-color-2">Текущий день</th>
          <th class="col-width-3">Вчера</th>
          <th class="col-width-4">Этот день недели</th>
        </tr>
      </thead>
      <tbody>
        <template v-for="(row, index) in tableData" :key="index">
          <tr @click="showChart(row, index)">
            <td class="col-width-1 col-color-1">{{ row.name }}</td>
            <td class="col-width-2 col-color-2">{{ formatNumber(row.currentDay) }}</td>
            <td :class="['col-width-3 col-color-1', getCellClass(row.currentDay, row.yesterday)]">
              {{ formatNumber(row.yesterday)}}
              <span v-if="row.yesterday !== 0" :style="{ color: getPercentageColor(row.currentDay, row.yesterday) }" class="percent-change">
                ({{ calculatePercentage(row.currentDay, row.yesterday) }})
              </span>
            </td>
            <td :class="['col-width-4 col-color-1', getCellClass(row.currentDay, row.weekday)]">
              {{ formatNumber(row.weekday)}}
            </td>
          </tr>
          <tr v-if="selectedRowIndex === index" class="with-chart">
            <td colspan="4">
              <div class="chart-container">
                <apexchart v-if="chartData" type="line" :options="chartOptions" :series="chartData"></apexchart>
              </div>
            </td>
          </tr>
        </template>
      </tbody>
    </table>
  </div>
</template>

<script>
import { defineComponent } from 'vue'
import ApexCharts from 'vue3-apexcharts'

export default defineComponent({
  name: 'HomeView',
  components: {
    apexchart: ApexCharts
  },
  data () {
    return {
      tableData: [
        { name: 'Выручка', currentDay: 15000, yesterday: 14000, weekday: 16000 },
        { name: 'Наличные', currentDay: 8000, yesterday: 7500, weekday: 8200 },
        { name: 'Безналичный расчет', currentDay: 7000, yesterday: 7000, weekday: 7000 },
        { name: 'Кредитная карта', currentDay: 3000, yesterday: 2800, weekday: 3200 },
        { name: 'Средний чек, руб', currentDay: 1200, yesterday: 1150, weekday: 1200 },
        { name: 'Средний гость, руб', currentDay: 800, yesterday: 800, weekday: 800 },
        { name: 'Удаление из чека (после оплаты), руб', currentDay: 200, yesterday: 180, weekday: 220 },
        { name: 'Удаление из чека (до оплаты), руб', currentDay: 150, yesterday: 178, weekday: 100 },
        { name: 'Кол-во чеков', currentDay: 50, yesterday: 48, weekday: 52 },
        { name: 'Кол-во гостей', currentDay: 100, yesterday: 123, weekday: 105 }
      ],
      selectedRowIndex: null,
      chartData: null,
      chartOptions: {
        chart: {
          id: 'basic-line',
          width: '100%',
          height: 300,
          toolbar: {
            show: true
          }
        },
        xaxis: {
          categories: ['Текущий день', 'Вчера', 'Этот день недели']
        },
        yaxis: {
          title: {
            text: 'Сумма (руб)'
          }
        },
        title: {
          text: 'График выручки по времени',
          align: 'center'
        },
        responsive: [{
          breakpoint: 480,
          options: {
            chart: {
              width: '100%'
            }
          }
        }]
      }
    }
  },
  methods: {
    showChart (row, index) {
      // Если строка уже выбрана, скрываем график
      if (this.selectedRowIndex === index) {
        this.selectedRowIndex = null
        this.chartData = null
        return
      }

      this.selectedRowIndex = index

      this.chartData = [
        {
          name: row.name,
          data: [row.currentDay, row.yesterday, row.weekday]
        }
      ]
    },
    getCellClass (currentValue, compareValue) {
      if (currentValue > compareValue) {
        return 'green'
      } else if (currentValue < compareValue) {
        return 'red'
      }
      return ''
    },
    formatNumber (number) {
      return number.toLocaleString('ru-RU')
    },
    calculatePercentage (current, previous) {
      if (previous === 0) return '—'
      const percent = ((current / previous) * 100 - 100).toFixed(1)
      return percent > 0 ? `+${percent}%` : `${percent}%`
    },
    getPercentageColor (current, previous) {
      if (current > previous) {
        return 'green'
      } else if (current < previous) {
        return 'red'
      }
      return '#646464'
    }
  }
})
</script>

<style lang="scss">
$border-color: #ddd;
$text-color: #646464;
$header-table-color: #f4f4f4;
$second-color: #e3f9ff;

.home {
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: center;
  height: 100vh;
  overflow-y: scroll;
  padding-right: 17px;
  box-sizing: border-box;

  &__table {
    width: 100%;
    max-width: 1200px;
    margin-top: 50px;
    border-collapse: separate;
    border-spacing: 2px 2px;
    table-layout: fixed;

    .col-width-1 {
      width: 400px;
      text-align: left;
    }

    .col-width-2,
    .col-width-3,
    .col-width-4 {
      width: 150px;
    }

    .col-color-1 {
      background-color: $header-table-color;
    }

    .col-color-2 {
      background-color: $second-color;
    }

    th,
    td {
      padding: 16px;
      text-align: left;
      font-size: 16px;
      font-weight: 400;
      color: $text-color;
    }

    /* Ховер на строки таблицы */
    tr:not(.with-chart):not(.header):hover td {
      background-color: #e7e7e7 !important;
      cursor: pointer;
    }

    th {
      text-align: center;
      background-color: $header-table-color;
    }

    td:not(.col-width-1) {
      text-align: right;
    }

    /* Стили для ячеек */
    .green {
      background-color: #d4eddac3; /* Зеленый цвет для увеличения */
    }

    .red {
      background-color: #f8d7dab9; /* Красный цвет для уменьшения */
    }

    .chart-container {
      width: 100%;
      overflow: hidden;
      height: auto;
    }
  }
}
</style>
