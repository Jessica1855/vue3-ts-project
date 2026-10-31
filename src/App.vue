<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from 'vue'
import * as echarts from 'echarts'

const currentTime = ref('')
const updateTime = () => {
  const now = new Date()
  currentTime.value = now.toLocaleString('zh-CN', { hour12: false })
}
setInterval(updateTime, 1000)

// --- 模拟实时交易滚动数据 ---
const orderList = ref([
  { time: '15:08:01', user: '138****1234', item: 'iPhone 15 Pro Max', price: '9999' },
  { time: '15:07:45', user: '159****5678', item: '戴森吹风机', price: '2990' },
  { time: '15:07:12', user: '186****9012', item: '雅诗兰黛小棕瓶', price: '850' },
  { time: '15:06:55', user: '135****3456', item: 'MacBook Pro 14寸', price: '14999' },
  { time: '15:06:20', user: '188****7890', item: 'Nike 运动鞋', price: '1299' },
])
const updateOrder = () => {
  const names = ['华为 Mate 60', 'SK-II 神仙水', '索尼降噪耳机', '海尔冰箱', '大疆无人机']
  const randomUser = Math.floor(Math.random() * 9000 + 1000)
  const newOrder = {
    time: new Date().toLocaleTimeString('zh-CN', { hour12: false }),
    user: `1${Math.floor(Math.random() * 9 + 1)}****${randomUser}`,
    item: names[Math.floor(Math.random() * names.length)],
    price: Math.floor(Math.random() * 10000 + 500).toString(),
  }
  orderList.value.unshift(newOrder)
  if (orderList.value.length > 5) orderList.value.pop()
}

// 图表 Refs
const trendChartRef = ref<HTMLElement | null>(null)
const pieChartRef = ref<HTMLElement | null>(null)
const channelChartRef = ref<HTMLElement | null>(null)
const radarChartRef = ref<HTMLElement | null>(null)

let trendChart: echarts.ECharts | null = null
let pieChart: echarts.ECharts | null = null
let channelChart: echarts.ECharts | null = null
let radarChart: echarts.ECharts | null = null

onMounted(() => {
  updateTime()

  // 定时器：图表数据波动 + 订单滚动
  const timer = setInterval(() => {
    updateOrder()
    if (trendChart) {
      const newData = Array.from({ length: 7 }, () => Math.floor(Math.random() * 5000) + 1000)
      trendChart.setOption({ series: [{ data: newData }] })
    }
  }, 3000)

  // --- 1. 双轴折柱混合图 (销售趋势) ---
  if (trendChartRef.value) {
    trendChart = echarts.init(trendChartRef.value)
    trendChart.setOption({
      // 标题
      title: {
        text: '近7日销售与转化趋势',
        textStyle: { color: '#fff', fontSize: 16 },
        left: 'center',
        top: 0,
      },
      tooltip: { trigger: 'axis', axisPointer: { type: 'cross' } },
      // 【修改这里】：图例往下移到 top: 35
      legend: { data: ['销售额(万)', '转化率(%)'], textStyle: { color: '#a0a0a0' }, top: 35 },
      // 【修改这里】：把 grid 的 top 调大，强行给标题和图例留出空间 (top: '25%')
      grid: { left: '5%', right: '5%', bottom: '5%', top: '25%', containLabel: true },
      xAxis: [
        {
          type: 'category',
          data: ['周一', '周二', '周三', '周四', '周五', '周六', '周日'],
          axisLabel: { color: '#a0a0a0' },
          axisLine: { lineStyle: { color: '#333' } },
        },
      ],
      yAxis: [
        {
          type: 'value',
          name: '销售额',
          nameTextStyle: { color: '#a0a0a0' },
          axisLabel: { color: '#a0a0a0' },
          splitLine: { lineStyle: { color: '#333', type: 'dashed' } },
        },
        {
          type: 'value',
          name: '转化率',
          nameTextStyle: { color: '#a0a0a0' },
          axisLabel: { color: '#a0a0a0', formatter: '{value}%' },
          splitLine: { show: false },
        },
      ],
      series: [
        {
          name: '销售额(万)',
          type: 'bar',
          barWidth: '30%',
          data: [3200, 4500, 3800, 6200, 5800, 8100, 9500],
          itemStyle: {
            color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
              { offset: 0, color: '#00f2fe' },
              { offset: 1, color: 'rgba(0, 242, 254, 0.1)' },
            ]),
          },
        },
        {
          name: '转化率(%)',
          type: 'line',
          yAxisIndex: 1,
          smooth: true,
          data: [12, 15, 11, 18, 16, 22, 25],
          itemStyle: { color: '#ff00cc' },
          lineStyle: { width: 3 },
        },
      ],
    })
  }

  // --- 2. 品类占比 (环形图) ---
  if (pieChartRef.value) {
    pieChart = echarts.init(pieChartRef.value)
    pieChart.setOption({
      // 标题
      title: {
        text: '品类销售占比',
        textStyle: { color: '#fff', fontSize: 14 },
        left: 'center',
        top: 0,
      },
      tooltip: { trigger: 'item' },
      // 【修改这里】：图例放回底部，去掉之前设置的垂直排列
      legend: {
        bottom: '0%',
        itemWidth: 10,
        itemHeight: 10,
        textStyle: { color: '#a0a0a0', fontSize: 12 },
      },
      series: [
        {
          type: 'pie',
          radius: ['35%', '55%'],
          // 【修改这里】：把饼图整体往上移 (y轴改为45%)，给底部图例留出空间
          center: ['50%', '45%'],
          avoidLabelOverlap: false,
          itemStyle: { borderRadius: 5, borderColor: '#111', borderWidth: 2 },
          label: { show: false },
          data: [
            { value: 1048, name: '数码家电', itemStyle: { color: '#ff00cc' } },
            { value: 735, name: '服饰美妆', itemStyle: { color: '#00f2fe' } },
            { value: 580, name: '食品生鲜', itemStyle: { color: '#f9ca24' } },
            { value: 484, name: '家居日用', itemStyle: { color: '#686de0' } },
          ],
        },
      ],
    })
  }

  // --- 3. 渠道转化率 (横向柱状图) ---
  if (channelChartRef.value) {
    channelChart = echarts.init(channelChartRef.value)
    channelChart.setOption({
      title: {
        text: '流量渠道转化率',
        textStyle: { color: '#fff', fontSize: 14 },
        left: 'center',
        top: 0,
      },
      tooltip: { trigger: 'axis', axisPointer: { type: 'shadow' } },
      // 【修改这里】：修改 grid 的 top，把图表往下压，给标题留空间
      grid: { left: '3%', right: '10%', bottom: '5%', top: '30px', containLabel: true },
      xAxis: {
        type: 'value',
        axisLabel: { color: '#a0a0a0' },
        splitLine: { lineStyle: { color: '#333', type: 'dashed' } },
      },
      yAxis: {
        type: 'category',
        data: ['自然搜索', '直通车', '微信私域', '抖音直播'],
        axisLabel: { color: '#a0a0a0', fontSize: 12 },
        axisLine: { lineStyle: { color: '#333' } },
      },
      series: [
        {
          type: 'bar',
          data: [12.5, 8.2, 18.6, 15.3],
          barWidth: '40%',
          itemStyle: {
            color: new echarts.graphic.LinearGradient(1, 0, 0, 0, [
              { offset: 0, color: '#00f2fe' },
              { offset: 1, color: '#4facfe' },
            ]),
            borderRadius: [0, 5, 5, 0],
          },
        },
      ],
    })
  }

  // --- 4. 用户画像 (雷达图) ---
  if (radarChartRef.value) {
    radarChart = echarts.init(radarChartRef.value)
    radarChart.setOption({
      // 标题固定在顶部
      title: {
        text: '核心用户画像',
        textStyle: { color: '#fff', fontSize: 14 },
        left: 'center',
        top: 0,
      },
      tooltip: {},
      radar: {
        indicator: [
          { name: '18-25岁', max: 100 },
          { name: '26-35岁', max: 100 },
          { name: '36-45岁', max: 100 },
          { name: '高净值', max: 100 },
          { name: '复购率高', max: 100 },
        ],
        // 【修改这里】：中心点往下移（y轴改为60%），半径缩小（50%）
        center: ['50%', '60%'],
        radius: '55%',
        axisName: { color: '#a0a0a0', fontSize: 12 },
        splitLine: { lineStyle: { color: '#333' } },
        splitArea: { show: false },
      },
      series: [
        {
          type: 'radar',
          data: [
            {
              value: [60, 90, 45, 80, 70],
              name: '用户占比',
              areaStyle: { color: 'rgba(255, 0, 204, 0.3)' },
              itemStyle: { color: '#ff00cc' },
              lineStyle: { width: 2 },
            },
          ],
        },
      ],
    })
  }

  // 缩放自适应
  window.addEventListener('resize', () => {
    trendChart?.resize()
    pieChart?.resize()
    channelChart?.resize()
    radarChart?.resize()
  })

  onBeforeUnmount(() => {
    clearInterval(timer)
    trendChart?.dispose()
    pieChart?.dispose()
    channelChart?.dispose()
    radarChart?.dispose()
  })
})
</script>

<template>
  <div class="dashboard-container">
    <header class="header">
      <h1>企业全域销售数据监控中心</h1>
      <div class="time">{{ currentTime }}</div>
    </header>

    <!-- KPI 指标 -->
    <div class="kpi-row">
      <div class="kpi-card">
        <div class="kpi-title">今日总销售额</div>
        <div class="kpi-value">¥ 1,284,500</div>
        <div class="kpi-trend up">↑ 12.5%</div>
      </div>
      <div class="kpi-card">
        <div class="kpi-title">今日总订单量</div>
        <div class="kpi-value">8,432</div>
        <div class="kpi-trend up">↑ 8.2%</div>
      </div>
      <div class="kpi-card">
        <div class="kpi-title">活跃用户数</div>
        <div class="kpi-value">32,109</div>
        <div class="kpi-trend down">↓ 2.1%</div>
      </div>
      <div class="kpi-card">
        <div class="kpi-title">客单价</div>
        <div class="kpi-value">¥ 152.3</div>
        <div class="kpi-trend up">↑ 4.3%</div>
      </div>
    </div>

    <!-- 核心主体区域 (改为三列布局) -->
    <div class="main-grid">
      <!-- 左侧列：雷达图 + 实时交易滚动 -->
      <div class="column-left">
        <div class="chart-box tech-border">
          <div ref="radarChartRef" style="width: 100%; height: 100%"></div>
        </div>
        <div class="list-box tech-border">
          <h3>实时交易动态</h3>
          <ul>
            <li v-for="(order, index) in orderList" :key="index">
              <span class="time">{{ order.time }}</span>
              <span class="user">{{ order.user }}</span>
              <span class="item">{{ order.item }}</span>
              <span class="price">¥{{ order.price }}</span>
            </li>
          </ul>
        </div>
      </div>

      <!-- 中间列：主力大图 -->
      <div class="column-center">
        <div class="chart-box tech-border" style="height: 100%">
          <div ref="trendChartRef" style="width: 100%; height: 100%"></div>
        </div>
      </div>

      <!-- 右侧列：环形图 + 柱状图 -->
      <div class="column-right">
        <div class="chart-box tech-border">
          <div ref="pieChartRef" style="width: 100%; height: 100%"></div>
        </div>
        <div class="chart-box tech-border">
          <div ref="channelChartRef" style="width: 100%; height: 100%"></div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.dashboard-container {
  width: 100vw;
  height: 100vh;
  background-color: #0b0f19;
  color: #fff;
  padding: 15px;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  font-family: 'PingFang SC', sans-serif;
}
.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-bottom: 15px;
  border-bottom: 1px solid rgba(0, 242, 254, 0.2);
}
.header h1 {
  font-size: 24px;
  margin: 0;
  background: linear-gradient(90deg, #00f2fe, #4facfe);
  -webkit-background-clip: text;
  color: transparent;
  letter-spacing: 2px;
}
.header .time {
  font-size: 14px;
  color: #a0a0a0;
}

/* KPI */
.kpi-row {
  display: flex;
  gap: 15px;
  margin-top: 15px;
  height: 100px;
}
.kpi-card {
  flex: 1;
  background: rgba(255, 255, 255, 0.03);
  border: 1px solid rgba(0, 242, 254, 0.2);
  border-radius: 8px;
  padding: 10px;
  text-align: center;
  box-shadow: 0 0 10px rgba(0, 242, 254, 0.05) inset;
}
.kpi-title {
  font-size: 13px;
  color: #a0a0a0;
}
.kpi-value {
  font-size: 24px;
  font-weight: bold;
  margin: 5px 0;
  color: #fff;
}
.kpi-trend {
  font-size: 12px;
}
.kpi-trend.up {
  color: #1dd1a1;
}
.kpi-trend.down {
  color: #ff6b6b;
}

/* 主体网格布局 */
.main-grid {
  display: grid;
  grid-template-columns: 1fr 2fr 1fr;
  gap: 15px;
  margin-top: 15px;
  flex: 1;
}
.column-left,
.column-right {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

/* 通用科技感卡片 */
.chart-box,
.list-box {
  background: rgba(255, 255, 255, 0.02);
  border-radius: 8px;
  padding: 10px;
  position: relative;
  overflow: hidden;
}
.column-left .chart-box,
.column-left .list-box {
  flex: 1;
}

/* 纯CSS科技感四角边框 */
.tech-border::before,
.tech-border::after {
  content: '';
  position: absolute;
  width: 15px;
  height: 15px;
  border: 2px solid #00f2fe;
  pointer-events: none;
}
.tech-border::before {
  top: 0;
  left: 0;
  border-right: none;
  border-bottom: none;
}
.tech-border::after {
  bottom: 0;
  right: 0;
  border-left: none;
  border-top: none;
}

/* 实时交易列表样式 */
.list-box h3 {
  margin: 0 0 10px 0;
  font-size: 14px;
  color: #fff;
  font-weight: normal;
  text-align: center;
}
.list-box ul {
  list-style: none;
  padding: 0;
  margin: 0;
}
.list-box li {
  display: flex;
  justify-content: space-between;
  font-size: 12px;
  color: #a0a0a0;
  padding: 8px 0;
  border-bottom: 1px dashed #333;
}
.list-box li span.time {
  color: #686de0;
  width: 60px;
}
.list-box li span.user {
  color: #00f2fe;
  width: 80px;
}
.list-box li span.item {
  flex: 1;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
.list-box li span.price {
  color: #ff00cc;
  width: 50px;
  text-align: right;
  font-weight: bold;
}
</style>
