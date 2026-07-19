<template>
  <router-link :to="to" :class="['server-card', { 'server-card-ring': isRingChart }]" :data-region="regionCode">
    <template v-if="isRingChart">
      <div class="server-card-ring-header">
        <div class="server-card-ring-main">
          <div class="server-card-ring-title-row">
            <span v-if="regionCode && regionCode !== 'xx'" class="server-card-ring-flag-wrap">
              <img class="flag-img server-card-ring-flag" :src="getPublicAssetUrl('flags/' + regionCode + '.svg')" :alt="regionCode">
            </span>
            <span v-else class="server-card-ring-flag-wrap">
              <span class="flag-fallback server-card-ring-flag-fallback">🏳️</span>
            </span>
            <span class="server-card-ring-name">{{ server.name }}</span>
          </div>
          <div class="server-card-ring-meta">
            <span class="server-card-ring-os">
              <OsIcon :os="server.os" />
              <span>{{ server.os || 'N/A' }}</span>
            </span>
            <span class="server-card-ring-dot">•</span>
            <span class="server-card-ring-uptime">{{ uptimeText }}</span>
          </div>
        </div>
        <div class="server-card-ring-actions">
          <span class="server-card-ring-status" :style="statusBadgeStyle">{{ statusText }}</span>
        </div>
      </div>

      <div class="server-card-ring-divider"></div>

      <div class="server-card-ring-metrics">
        <div class="metric-ring-item">
          <div class="metric-ring-chart" :style="getRingStyle(cpuPercent, 'var(--accent-blue)')">
            <span class="metric-ring-track"></span>
            <span class="metric-ring-progress"></span>
            <span class="metric-ring-center">{{ roundedPercent(cpuPercent) }}%</span>
          </div>
          <div class="metric-ring-label">CPU</div>
          <div class="metric-ring-subtext">{{ cpuPercent.toFixed(1) }}%</div>
        </div>

        <div class="metric-ring-item">
          <div class="metric-ring-chart" :style="getRingStyle(ramPercent, 'var(--accent-blue)')">
            <span class="metric-ring-track"></span>
            <span class="metric-ring-progress"></span>
            <span class="metric-ring-center">{{ roundedPercent(ramPercent) }}%</span>
          </div>
          <div class="metric-ring-label">RAM</div>
          <div class="metric-ring-subtext">{{ ramUsageText }}</div>
        </div>

        <div class="metric-ring-item">
          <div class="metric-ring-chart" :style="getRingStyle(diskPercent, 'var(--accent-blue)')">
            <span class="metric-ring-track"></span>
            <span class="metric-ring-progress"></span>
            <span class="metric-ring-center">{{ roundedPercent(diskPercent) }}%</span>
          </div>
          <div class="metric-ring-label">Disk</div>
          <div class="metric-ring-subtext">{{ diskUsageText }}</div>
        </div>
      </div>

      <div class="server-card-network-panel">
        <div class="server-card-network-row">
          <span class="server-card-network-label">{{ trans.networkTraffic }}</span>
          <span class="server-card-network-values">
            <span class="server-card-speed-up">↑ {{ netOutSpeed }}/s</span>
            <span class="server-card-speed-down">↓ {{ netInSpeed }}/s</span>
          </span>
        </div>
        <div class="server-card-network-row">
          <span class="server-card-network-label">{{ trans.totalTraffic }}</span>
          <span class="server-card-network-values server-card-total-values">
            <span>↑ {{ totalTx }}</span>
            <span>↓ {{ totalRx }}</span>
          </span>
        </div>
        <div v-if="sysConfig.show_tf && trafficLimitSummary" class="server-card-limit-section">
          <div class="server-card-limit-header">
            <span>SUM Limit</span>
            <span>{{ trafficLimitText }} | {{ trafficLimitPercentText }}%</span>
          </div>
          <div class="server-card-limit-bar">
            <div class="server-card-limit-fill" :style="{ width: Math.min(100, trafficUsagePercent) + '%' }"></div>
          </div>
        </div>
      </div>
    </template>

    <template v-else>
      <div class="server-card-header">
        <div class="server-identity">
          <span v-if="regionCode && regionCode !== 'xx'" class="country-os-icons">
            <img class="flag-img" :src="getPublicAssetUrl('flags/' + regionCode + '.svg')" :alt="regionCode">
            <OsIcon :os="server.os" />
          </span>
          <span v-else class="country-os-icons">
            <span class="flag-fallback">🏳️</span>
            <OsIcon :os="server.os" />
          </span>
          <span class="server-name">{{ server.name }}</span>
        </div>
        <span class="status-label" :style="{ color: statusColor, borderColor: statusColor }">{{ statusText }}</span>
      </div>
      <div class="server-meta">
        <div class="card-meta">
          <div v-if="sysConfig.show_price && server.price" class="card-meta-item">💰 {{ server.price }}</div>
          <div v-if="sysConfig.show_expire && server.expire_date" class="card-meta-item">📅 <span :class="{ 'expired': isExpired }">{{ expireText }}</span></div>
        </div>
        <div class="card-badges">
          <span v-for="(tag, index) in tagList" :key="tag" :class="['badge', 'badge-tag', tagColorClass(index)]">{{ tag }}</span>
          <span v-if="server.ip_v4 === '1' && server.ip_v6 === '1'" class="badge badge badge-v4-v6">IPv4/6</span>
          <template v-else>
            <span v-if="server.ip_v4 === '1'" class="badge badge-v4">IPv4</span>
            <span v-if="server.ip_v6 === '1'" class="badge badge-v6">IPv6</span>
          </template>
        </div>
      </div>
      <div class="server-stats">
        <div class="stat-row">
          <span class="stat-key">CPU</span>
          <div class="stat-bar-container">
            <div class="stat-bar-fill" :style="{ width: cpuPercent + '%', background: 'var(--accent-cyan)' }"></div>
          </div>
          <span class="stat-value">{{ cpuPercent.toFixed(1) }}%</span>
        </div>
        <div class="stat-row">
          <span class="stat-key">RAM</span>
          <div class="stat-bar-container">
            <div class="stat-bar-fill" :style="{ width: ramPercent + '%', background: 'var(--accent-purple)' }"></div>
          </div>
          <span class="stat-value">{{ ramPercent.toFixed(2) }}%</span>
        </div>
        <div class="stat-row">
          <span class="stat-key">DISK</span>
          <div class="stat-bar-container">
            <div class="stat-bar-fill" :style="{ width: diskPercent + '%', background: 'var(--accent-green)' }"></div>
          </div>
          <span class="stat-value">{{ diskPercent.toFixed(2) }}%</span>
        </div>
        <div class="stat-row" v-if="sysConfig.show_tf && server.traffic_limit">
          <span class="stat-key">USE</span>
          <div class="stat-bar-container">
            <div class="stat-bar-fill" :style="{ width: Math.min(100, trafficUsagePercent) + '%', background: 'var(--accent-blue)' }"></div>
          </div>
          <span class="stat-value">{{ trafficUsagePercentText }}%</span>
        </div>
        <div class="stat-row">
          <span class="stat-key">NET</span>
          <span class="net-down">▼ {{ netInSpeed }}/s</span>
          <span class="net-up">▲ {{ netOutSpeed }}/s</span>
        </div>
        <div class="stat-row">
          <span class="stat-key">TRF</span>
          <span class="net-down">▼ {{ totalRx }}</span>
          <span class="net-up">▲ {{ totalTx }}</span>
          <span v-if="sysConfig.show_tf && server.traffic_limit" class="stat-limit">/ 📦 {{ formatBytes(server.traffic_limit * 1024 * 1024 * 1024) }}</span>
        </div>
        <div v-if="sysConfig.show_time" class="stat-row stat-time-row">
          <span class="stat-key">TIME</span>
          <span class="stat-time-value">{{ dataTimeText }}</span>
        </div>
      </div>
      <div class="ping-panel">
        <div class="ping-item">
          <span class="ping-label">CT</span>
          <span class="ping-value" :style="{ color: getPingColor(server.ping_ct) }">{{ !isPingValid(server.ping_ct) ? trans.timeout : server.ping_ct + 'ms' }}</span>
        </div>
        <div class="ping-item">
          <span class="ping-label">CU</span>
          <span class="ping-value" :style="{ color: getPingColor(server.ping_cu) }">{{ !isPingValid(server.ping_cu) ? trans.timeout : server.ping_cu + 'ms' }}</span>
        </div>
        <div class="ping-item">
          <span class="ping-label">CM</span>
          <span class="ping-value" :style="{ color: getPingColor(server.ping_cm) }">{{ !isPingValid(server.ping_cm) ? trans.timeout : server.ping_cm + 'ms' }}</span>
        </div>
        <div class="ping-item">
          <span class="ping-label">BD</span>
          <span class="ping-value" :style="{ color: getPingColor(server.ping_bd) }">{{ !isPingValid(server.ping_bd) ? trans.timeout : server.ping_bd + 'ms' }}</span>
        </div>
      </div>
    </template>
  </router-link>
</template>

<script setup>
import { computed } from 'vue'
import OsIcon from './OsIcon.vue'
import { formatBytes, getFlagRegionCode, isServerOnline } from '../utils/api'
import { getPublicAssetUrl } from '../utils/config'
import { currentLang, useTranslation } from '../utils/i18n'
import { PING } from '../utils/constants'
import { normalizeTimestamp, formatDateTime } from '../utils/time.js'

const props = defineProps({
  server: {
    type: Object,
    required: true
  },
  sysConfig: {
    type: Object,
    default: () => ({
      show_price: true,
      show_expire: true,
      show_tf: true,
      show_time: true,
      card_chart_type: 'bar'
    })
  },
  to: {
    type: String,
    default: ''
  }
})

const trans = useTranslation()

const isRingChart = computed(() => props.sysConfig.card_chart_type === 'ring')

const currentTime = computed(() => {
  const ts = Number(props.server.current_timestamp)
  if (Number.isFinite(ts) && ts > 0) {
    return ts < 10000000000 ? ts * 1000 : ts
  }
  return Date.now()
})

const regionCode = computed(() => getFlagRegionCode(props.server.region))

const isOnline = computed(() => isServerOnline(props.server, currentTime.value))

const statusColor = computed(() => isOnline.value ? 'var(--accent-green)' : 'var(--accent-red)')
const statusText = computed(() => isOnline.value ? trans.value.online : trans.value.offline)
const statusBadgeStyle = computed(() => ({
  backgroundColor: statusColor.value,
  color: isOnline.value ? '#08110d' : '#ffffff'
}))

const clampPercent = (value) => {
  const num = Number(value)
  if (!Number.isFinite(num)) return 0
  return Math.max(0, Math.min(100, num))
}

const cpuPercent = computed(() => clampPercent(Number.parseFloat(props.server.cpu || 0) || 0))
const ramPercent = computed(() => {
  const total = Number.parseFloat(props.server.ram_total) || 0
  if (total > 0) {
    return clampPercent(((Number.parseFloat(props.server.ram_used) || 0) / total) * 100)
  }
  return 0
})
const diskPercent = computed(() => {
  const total = Number.parseFloat(props.server.disk_total) || 0
  if (total > 0) {
    return clampPercent(((Number.parseFloat(props.server.disk_used) || 0) / total) * 100)
  }
  return 0
})

const getTrafficUsageBytes = () => {
  const rx = Number.parseFloat(props.server.net_rx_monthly) || 0
  const tx = Number.parseFloat(props.server.net_tx_monthly) || 0
  const calcType = props.server.traffic_calc_type || 'total'
  if (calcType === 'dl') return rx
  if (calcType === 'ul') return tx
  return rx + tx
}

const trafficLimitSummary = computed(() => {
  const limitGb = Number.parseFloat(props.server.traffic_limit) || 0
  if (limitGb <= 0) return null
  const limitBytes = limitGb * 1024 * 1024 * 1024
  const usedBytes = getTrafficUsageBytes()
  return {
    usedBytes,
    limitBytes,
    percent: (usedBytes / limitBytes) * 100
  }
})

const trafficUsagePercent = computed(() => trafficLimitSummary.value ? trafficLimitSummary.value.percent : 0)
const trafficUsagePercentText = computed(() => trafficUsagePercent.value.toFixed(1))
const trafficLimitPercentText = computed(() => {
  if (!trafficLimitSummary.value) return '0.000'
  return trafficUsagePercent.value < 0.1 ? trafficUsagePercent.value.toFixed(3) : trafficUsagePercent.value.toFixed(1)
})
const trafficLimitText = computed(() => {
  if (!trafficLimitSummary.value) return ''
  return `${formatBytes(trafficLimitSummary.value.usedBytes)} / ${formatBytes(trafficLimitSummary.value.limitBytes)}`
})

const tagList = computed(() => String(props.server.tags || '')
  .split(',')
  .map(tag => tag.trim())
  .filter(Boolean)
)
const tagColorClass = (index) => `tag-color-${index % 6}`

const netInSpeed = computed(() => formatBytes(props.server.net_in_speed))
const netOutSpeed = computed(() => formatBytes(props.server.net_out_speed))
const totalRx = computed(() => formatBytes(props.server.net_rx_monthly))
const totalTx = computed(() => formatBytes(props.server.net_tx_monthly))

const formatUptime = (bootTime, nowTs = Date.now()) => {
  if (!bootTime) return 'N/A'

  let bootTimeMs = null
  if (typeof bootTime === 'string' && !/^\d+$/.test(bootTime)) {
    const parsed = new Date(bootTime)
    if (!Number.isNaN(parsed.getTime())) {
      bootTimeMs = parsed.getTime()
    }
  } else {
    const timestamp = Number.parseInt(bootTime)
    if (Number.isFinite(timestamp)) {
      bootTimeMs = timestamp < 1000000000000 ? timestamp * 1000 : timestamp
    }
  }

  if (!bootTimeMs) return 'N/A'

  const diffMs = nowTs - bootTimeMs
  if (diffMs < 0) return 'N/A'

  const totalSeconds = Math.floor(diffMs / 1000)
  const days = Math.floor(totalSeconds / 86400)
  const hours = Math.floor((totalSeconds % 86400) / 3600)
  const minutes = Math.floor((totalSeconds % 3600) / 60)
  const seconds = totalSeconds % 60
  const zh = currentLang.value === 'zh'
  const parts = []

  if (days > 0) parts.push(zh ? `${days}天` : `${days}d`)
  if (hours > 0) parts.push(zh ? `${hours}小时` : `${hours}h`)
  if (minutes > 0) parts.push(zh ? `${minutes}分` : `${minutes}m`)
  if (seconds > 0 || parts.length === 0) parts.push(zh ? `${seconds}秒` : `${seconds}s`)

  return parts.slice(0, 3).join(' ')
}

const uptimeText = computed(() => formatUptime(props.server.boot_time, currentTime.value))

const formatMetricUsage = (used, total) => `${formatBytes((Number(used) || 0) * 1024 * 1024)} / ${formatBytes((Number(total) || 0) * 1024 * 1024)}`
const ramUsageText = computed(() => formatMetricUsage(props.server.ram_used, props.server.ram_total))
const diskUsageText = computed(() => formatMetricUsage(props.server.disk_used, props.server.disk_total))

const dataTimeText = computed(() => {
  const reportTimestamp = normalizeTimestamp(props.server.report_timestamp ?? props.server.last_updated)
  if (!isOnline.value) return formatDateTime(reportTimestamp)

  const displayTimestamp = normalizeTimestamp(
    props.server.display_timestamp ?? props.server.sample_timestamp ?? props.server.timestamp ?? reportTimestamp
  )
  const sampleTimestamp = normalizeTimestamp(
    props.server.sample_timestamp ?? props.server.timestamp ?? displayTimestamp
  )
  const lagSeconds = displayTimestamp && sampleTimestamp
    ? Math.max(0, Math.floor((displayTimestamp - sampleTimestamp) / 1000))
    : 0
  return `${formatDateTime(sampleTimestamp)}${lagSeconds > 0 ? ` (+${lagSeconds}s)` : ''}`
})

const isExpired = computed(() => {
  const expTime = new Date(props.server.expire_date).getTime()
  return !isNaN(expTime) && expTime < currentTime.value
})

const expireText = computed(() => {
  const expTime = new Date(props.server.expire_date).getTime()
  if (isNaN(expTime)) return ''
  const diff = expTime - currentTime.value
  const days = Math.ceil(diff / (1000 * 3600 * 24))
  return days > 0 ? `${days}${trans.value.expireDays}` : trans.value.expired
})

const getRingStyle = (value, color) => ({
  '--ring-value': `${clampPercent(value)}`,
  '--ring-color': color
})

const roundedPercent = (value) => Math.round(clampPercent(value))

const isPingValid = (ping) => {
  if (ping === null || ping === undefined || ping === '' || ping === '0') {
    return false
  }
  const val = parseInt(ping)
  return val > 0
}

const getPingColor = (ping) => {
  if (!isPingValid(ping)) return 'var(--accent-red)'
  const val = parseInt(ping)
  if (val < PING.GOOD_THRESHOLD) return 'var(--accent-green)'
  if (val < PING.WARNING_THRESHOLD) return 'var(--accent-yellow)'
  return 'var(--accent-red)'
}
</script>
