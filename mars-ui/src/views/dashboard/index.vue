<template>
  <div class="page-container">
    <!-- 欢迎区域 -->
    <div class="welcome-section">
      <!-- 左侧欢迎信息 -->
      <div class="welcome-info">
        <div class="welcome-header">
          <n-avatar round :size="56" :src="userStore.avatar || undefined">
            {{ userStore.nickname?.charAt(0) || 'U' }}
          </n-avatar>
          <div class="welcome-text">
            <h1 class="welcome-title">
              {{ getGreeting() }}，{{ userStore.nickname }} 👋
            </h1>
            <p class="welcome-desc">
              这是您的管理控制台，您可以在这里管理系统的各项功能
            </p>
          </div>
        </div>
        <div class="welcome-time">
          <div class="time-display">{{ currentTime }}</div>
          <div class="date-display">{{ currentDate }}</div>
        </div>
      </div>
      <!-- 右侧轮播Banner -->
      <div class="welcome-banner">
        <n-carousel autoplay :interval="5000" dot-type="line" show-arrow="hover" class="banner-carousel">
          <div v-for="(banner, index) in banners" :key="index" class="banner-item"
               :style="{ background: banner.bgColor }">
            <div class="banner-content">
              <div class="banner-text">
                <h3 class="banner-title">{{ banner.title }}</h3>
                <p class="banner-subtitle">{{ banner.subtitle }}</p>
              </div>
              <div class="banner-icon">
                <n-icon :size="64" :color="banner.iconColor">
                  <component :is="banner.icon"/>
                </n-icon>
              </div>
            </div>
          </div>
        </n-carousel>
      </div>
    </div>

    <!-- 统计卡片 -->
    <div class="stat-cards">
      <n-card v-for="stat in stats" :key="stat.title" class="stat-card">
        <div class="stat-content">
          <div class="stat-icon" :style="{ background: stat.bgColor }">
            <n-icon size="24" :color="stat.color">
              <component :is="stat.icon"/>
            </n-icon>
          </div>
          <div class="stat-info">
            <n-skeleton v-if="loading" :width="60" :height="28"/>
            <div v-else class="stat-value">{{ stat.value }}</div>
            <div class="stat-title">{{ stat.title }}</div>
          </div>
        </div>
      </n-card>
    </div>

    <!-- 项目仪表盘区域 -->
    <n-grid :x-gap="20" :cols="2" class="dashboard-section">
      <!-- 我的项目仪表盘 -->
      <n-gi>
        <n-card title="我的项目" class="dashboard-card">
          <div class="project-dashboard">
            <n-data-table
              :columns="projectColumns"
              :data="myProjects"
              :loading="loading"
              :row-key="(row: Project) => row.id"
              size="small"
            >
              <template #header-extra>
                <n-button size="small" type="primary" @click="router.push('/project')">
                  查看全部
                </n-button>
              </template>
            </n-data-table>
          </div>
        </n-card>
      </n-gi>

      <!-- 下属项目仪表盘 -->
      <n-gi>
        <n-card title="下属项目" class="dashboard-card">
          <div class="subordinate-dashboard">
            <n-data-table
              :columns="subordinateColumns"
              :data="subordinateProjects"
              :loading="loading"
              :row-key="(row: SubordinateProject) => row.id"
              size="small"
            >
              <template #header-extra>
                <n-button size="small" type="primary" @click="router.push('/project')">
                  查看全部
                </n-button>
              </template>
            </n-data-table>
          </div>
        </n-card>
      </n-gi>
    </n-grid>

    <!-- 告警信息和快捷入口 -->
    <n-grid :x-gap="20" :cols="2" class="bottom-section">
      <!-- 告警信息 -->
      <n-gi>
        <n-card title="项目告警" class="alert-card">
          <div class="alert-dashboard">
            <!-- 告警统计 -->
            <div class="alert-stats">
              <div class="alert-stat-item">
                <div class="alert-stat-value">{{ alerts.filter(a => a.level === 'high').length }}</div>
                <div class="alert-stat-label">紧急告警</div>
              </div>
              <div class="alert-stat-item">
                <div class="alert-stat-value">{{ alerts.filter(a => a.level === 'medium').length }}</div>
                <div class="alert-stat-label">重要告警</div>
              </div>
              <div class="alert-stat-item">
                <div class="alert-stat-value">{{ alerts.filter(a => a.status === '未处理').length }}</div>
                <div class="alert-stat-label">未处理</div>
              </div>
              <div class="alert-stat-item">
                <div class="alert-stat-value">{{ alerts.length }}</div>
                <div class="alert-stat-label">总计</div>
              </div>
            </div>
            
            <!-- 告警列表 -->
            <n-data-table
              :columns="alertColumns"
              :data="alerts"
              :loading="loading"
              :row-key="(row: Alert) => row.id"
              size="small"
              :max-height="300"
            >
              <template #header-extra>
                <n-button size="small" type="primary" @click="router.push('/project')">
                  查看全部
                </n-button>
              </template>
            </n-data-table>
          </div>
        </n-card>
      </n-gi>

      <!-- 快捷入口 -->
      <n-gi>
        <n-card title="快捷入口" class="shortcuts-card">
          <div class="shortcuts-grid">
            <div
                v-for="shortcut in shortcuts"
                :key="shortcut.path"
                class="shortcut-item"
                @click="router.push(shortcut.path)"
            >
              <div class="shortcut-icon" :style="{ background: shortcut.bgColor }">
                <n-icon size="24" :color="shortcut.color">
                  <component :is="shortcut.icon"/>
                </n-icon>
              </div>
              <div class="shortcut-name">{{ shortcut.name }}</div>
            </div>
          </div>
        </n-card>
      </n-gi>
    </n-grid>
  </div>
</template>

<script setup lang="ts">
import {ref, onMounted, onUnmounted, markRaw, h} from 'vue'
import {useRouter} from 'vue-router'
import {
  PersonOutline,
  PeopleOutline,
  MenuOutline,
  ShieldCheckmarkOutline,
  LogoGithub,
  LogoGitlab,
  LogoWechat,
  Globe,
  Mail,
  Star,
  Refresh,
  DocumentText,
  SettingsOutline,
  TimerOutline,
  ServerOutline,
  RocketOutline,
  SparklesOutline,
  CodeSlashOutline,
  CloudOutline,
  ChatbubbleOutline,
  AlertCircleOutline,
  CheckmarkCircleOutline,
  AlertTriangleOutline,
  FolderOutline,
  UserOutline,
  CalendarOutline
} from '@vicons/ionicons5'
import {useUserStore} from '@/stores/user'
import {dashboardApi} from '@/api/system'
import {NTag, type DataTableColumns} from 'naive-ui'

const router = useRouter()
const userStore = useUserStore()

const currentTime = ref('')
const currentDate = ref('')
const loading = ref(true)

// 类型定义
interface Project {
  id: number
  name: string
  status: string
  progress: number
  deadline: string
  manager: string
}

interface SubordinateProject {
  id: number
  name: string
  status: string
  progress: number
  deadline: string
  owner: string
  manager: string
}

interface Alert {
  id: number
  title: string
  level: string
  message: string
  time: string
  status: string
}

// 获取问候语
function getGreeting() {
  const hour = new Date().getHours()
  if (hour < 6) return '夜深了'
  if (hour < 9) return '早上好'
  if (hour < 12) return '上午好'
  if (hour < 14) return '中午好'
  if (hour < 18) return '下午好'
  if (hour < 22) return '晚上好'
  return '夜深了'
}

// 轮播Banner数据
const banners = [
  {
    title: '周报系统',
    subtitle: '周报管理',
    bgColor: 'linear-gradient(135deg, #4facfe 0%, #00f2fe 100%)',
    icon: markRaw(RocketOutline),
    iconColor: 'rgba(255,255,255,0.3)'
  },]

// 统计数据
const stats = ref([
  {
    title: '用户总数',
    value: 0,
    icon: markRaw(PersonOutline),
    color: '#111827',
    bgColor: '#F3F4F6'
  },
  {
    title: '角色数量',
    value: 0,
    icon: markRaw(PeopleOutline),
    color: '#059669',
    bgColor: '#D1FAE5'
  },
  {
    title: '菜单数量',
    value: 0,
    icon: markRaw(MenuOutline),
    color: '#2563EB',
    bgColor: '#DBEAFE'
  },
  {
    title: '权限数量',
    value: 0,
    icon: markRaw(ShieldCheckmarkOutline),
    color: '#D97706',
    bgColor: '#FEF3C7'
  }
])

// 我的项目数据
const myProjects = ref<Project[]>([
  {
    id: 1,
    name: '项目A',
    status: '进行中',
    progress: 75,
    deadline: '2026-03-15',
    manager: '张三'
  },
  {
    id: 2,
    name: '项目B',
    status: '待开始',
    progress: 0,
    deadline: '2026-04-10',
    manager: '张三'
  },
  {
    id: 3,
    name: '项目C',
    status: '已完成',
    progress: 100,
    deadline: '2026-02-20',
    manager: '张三'
  }
])

// 下属项目数据
const subordinateProjects = ref<SubordinateProject[]>([
  {
    id: 4,
    name: '项目D',
    status: '进行中',
    progress: 45,
    deadline: '2026-03-20',
    owner: '李四',
    manager: '张三'
  },
  {
    id: 5,
    name: '项目E',
    status: '进行中',
    progress: 60,
    deadline: '2026-03-25',
    owner: '王五',
    manager: '张三'
  },
  {
    id: 6,
    name: '项目F',
    status: '待开始',
    progress: 0,
    deadline: '2026-04-05',
    owner: '赵六',
    manager: '张三'
  }
])

// 告警信息数据
const alerts = ref<Alert[]>([
  {
    id: 1,
    title: '项目A超时告警',
    level: 'high',
    message: '项目A已超时2天，原计划截止日期为2026-02-26，负责人：张三',
    time: '2026-02-28 14:30:00',
    status: '未处理'
  },
  {
    id: 2,
    title: '任务B超时告警',
    level: 'medium',
    message: '任务B已超时1天，负责人：李四，所属项目：项目A',
    time: '2026-02-28 13:15:00',
    status: '已处理'
  },
  {
    id: 3,
    title: '项目C即将超时',
    level: 'medium',
    message: '项目C将在3天后超时，当前进度仅为60%，负责人：王五',
    time: '2026-02-28 11:45:00',
    status: '未处理'
  },
  {
    id: 4,
    title: '任务D超时告警',
    level: 'high',
    message: '任务D已超时3天，负责人：赵六，所属项目：项目B',
    time: '2026-02-27 16:20:00',
    status: '处理中'
  },
  {
    id: 5,
    title: '项目E进度滞后',
    level: 'medium',
    message: '项目E进度滞后，当前进度仅为40%，剩余时间不足',
    time: '2026-02-28 10:00:00',
    status: '未处理'
  },
  {
    id: 6,
    title: '任务F即将超时',
    level: 'low',
    message: '任务F将在1天后超时，负责人：张三',
    time: '2026-02-28 09:30:00',
    status: '未处理'
  }
])

// 项目列配置
const projectColumns: DataTableColumns<Project> = [
  {
    title: '项目名称',
    key: 'name',
    width: 150
  },
  {
    title: '状态',
    key: 'status',
    width: 100,
    render(row) {
      const statusMap: Record<string, { type: string; label: string }> = {
        '进行中': { type: 'warning', label: '进行中' },
        '待开始': { type: 'info', label: '待开始' },
        '已完成': { type: 'success', label: '已完成' }
      }
      const status = statusMap[row.status] || { type: 'default', label: row.status }
      return h(NTag, { type: status.type }, { default: () => status.label })
    }
  },
  {
    title: '进度',
    key: 'progress',
    width: 120,
    render(row) {
      return h('div', {
        style: {
          width: '100%',
          height: '6px',
          backgroundColor: '#e5e7eb',
          borderRadius: '3px',
          overflow: 'hidden'
        }
      }, [
        h('div', {
          style: {
            width: `${row.progress}%`,
            height: '100%',
            backgroundColor: row.progress === 100 ? '#10b981' : '#3b82f6',
            transition: 'width 0.3s ease'
          }
        }),
        h('div', {
          style: {
            marginTop: '4px',
            fontSize: '12px',
            color: '#6b7280'
          }
        }, `${row.progress}%`)
      ])
    }
  },
  {
    title: '截止日期',
    key: 'deadline',
    width: 120
  }
]

// 下属项目列配置
const subordinateColumns: DataTableColumns<SubordinateProject> = [
  {
    title: '项目名称',
    key: 'name',
    width: 150
  },
  {
    title: '负责人',
    key: 'owner',
    width: 100
  },
  {
    title: '状态',
    key: 'status',
    width: 100,
    render(row) {
      const statusMap: Record<string, { type: string; label: string }> = {
        '进行中': { type: 'warning', label: '进行中' },
        '待开始': { type: 'info', label: '待开始' },
        '已完成': { type: 'success', label: '已完成' }
      }
      const status = statusMap[row.status] || { type: 'default', label: row.status }
      return h(NTag, { type: status.type }, { default: () => status.label })
    }
  },
  {
    title: '进度',
    key: 'progress',
    width: 120,
    render(row) {
      return h('div', {
        style: {
          width: '100%',
          height: '6px',
          backgroundColor: '#e5e7eb',
          borderRadius: '3px',
          overflow: 'hidden'
        }
      }, [
        h('div', {
          style: {
            width: `${row.progress}%`,
            height: '100%',
            backgroundColor: row.progress === 100 ? '#10b981' : '#3b82f6',
            transition: 'width 0.3s ease'
          }
        }),
        h('div', {
          style: {
            marginTop: '4px',
            fontSize: '12px',
            color: '#6b7280'
          }
        }, `${row.progress}%`)
      ])
    }
  },
  {
    title: '截止日期',
    key: 'deadline',
    width: 120
  }
]

// 告警信息列配置
const alertColumns: DataTableColumns<Alert> = [
  {
    title: '告警类型',
    key: 'type',
    width: 100,
    render(row) {
      if (row.title.includes('项目')) {
        return h(NTag, { type: 'info' }, { default: () => '项目' })
      } else if (row.title.includes('任务')) {
        return h(NTag, { type: 'warning' }, { default: () => '任务' })
      } else {
        return h(NTag, { type: 'default' }, { default: () => '其他' })
      }
    }
  },
  {
    title: '告警标题',
    key: 'title',
    ellipsis: true
  },
  {
    title: '严重程度',
    key: 'level',
    width: 100,
    render(row) {
      const levelMap: Record<string, { type: string; label: string }> = {
        'high': { type: 'error', label: '紧急' },
        'medium': { type: 'warning', label: '重要' },
        'low': { type: 'info', label: '一般' }
      }
      const level = levelMap[row.level] || { type: 'default', label: row.level }
      return h(NTag, { type: level.type }, { default: () => level.label })
    }
  },
  {
    title: '详细信息',
    key: 'message',
    ellipsis: true
  },
  {
    title: '告警时间',
    key: 'time',
    width: 150
  },
  {
    title: '处理状态',
    key: 'status',
    width: 100,
    render(row) {
      const statusMap: Record<string, { type: string; label: string }> = {
        '未处理': { type: 'error', label: '未处理' },
        '已处理': { type: 'success', label: '已处理' },
        '处理中': { type: 'warning', label: '处理中' }
      }
      const status = statusMap[row.status] || { type: 'default', label: row.status }
      return h(NTag, { type: status.type }, { default: () => status.label })
    }
  }
]

// 快捷入口
const shortcuts = [
  {
    name: '用户管理',
    path: '/system/user',
    icon: markRaw(PersonOutline),
    color: '#111827',
    bgColor: '#F3F4F6'
  },
  {
    name: '菜单管理',
    path: '/system/menu',
    icon: markRaw(MenuOutline),
    color: '#2563EB',
    bgColor: '#DBEAFE'
  },
  {
    name: '服务监控',
    path: '/monitor/server',
    icon: markRaw(ServerOutline),
    color: '#0891B2',
    bgColor: '#CFFAFE'
  },
  {
    name: '项目管理',
    path: '/project',
    icon: markRaw(FolderOutline),
    color: '#8B5CF6',
    bgColor: '#EDE9FE'
  },
  {
    name: '会议管理',
    path: '/meeting',
    icon: markRaw(CalendarOutline),
    color: '#EC4899',
    bgColor: '#FCE7F3'
  },
  {
    name: '系统设置',
    path: '/system/setting',
    icon: markRaw(SettingsOutline),
    color: '#6366F1',
    bgColor: '#EEF2FF'
  }
]

// 加载统计数据
async function loadStats() {
  try {
    loading.value = true
    const data = await dashboardApi.getStats()
    stats.value[0].value = data.userCount
    stats.value[1].value = data.roleCount
    stats.value[2].value = data.menuCount
    stats.value[3].value = data.permissionCount
  } catch (error) {
    console.error('加载统计数据失败', error)
  } finally {
    loading.value = false
  }
}

// 更新时间
function updateTime() {
  const now = new Date()
  currentTime.value = now.toLocaleTimeString('zh-CN', {hour12: false})
  currentDate.value = now.toLocaleDateString('zh-CN', {
    year: 'numeric',
    month: 'long',
    day: 'numeric',
    weekday: 'long'
  })
}

let timer: number
onMounted(() => {
  updateTime()
  timer = window.setInterval(updateTime, 1000)
  loadStats()
})

onUnmounted(() => {
  clearInterval(timer)
})
</script>

<style lang="scss" scoped>
// 欢迎区域
.welcome-section {
  display: flex;
  gap: 20px;
  margin-bottom: 20px;
}

.welcome-info {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  padding: 24px;
  background: #fff;
  border-radius: 12px;
  box-shadow: 0 1px 3px 0 rgb(0 0 0 / 0.1);
}

.welcome-header {
  display: flex;
  align-items: center;
  gap: 16px;
}

.welcome-text {
  flex: 1;
}

.welcome-title {
  font-size: 22px;
  font-weight: 700;
  color: #111827;
  margin: 0 0 6px 0;
}

.welcome-desc {
  font-size: 14px;
  color: #6B7280;
  margin: 0;
}

.welcome-time {
  display: flex;
  align-items: baseline;
  gap: 12px;
  margin-top: 16px;
}

.time-display {
  font-size: 36px;
  font-weight: 700;
  color: #111827;
  font-variant-numeric: tabular-nums;
}

.date-display {
  font-size: 14px;
  color: #6B7280;
}

// 轮播Banner
.welcome-banner {
  width: 380px;
  flex-shrink: 0;
}

.banner-carousel {
  height: 100%;
  border-radius: 12px;
  overflow: hidden;

  :deep(.n-carousel__slides) {
    height: 100%;
  }

  :deep(.n-carousel__slide) {
    height: 100%;
  }

  :deep(.n-carousel__dots) {
    bottom: 12px;
  }

  :deep(.n-carousel__dot) {
    background: rgba(255, 255, 255, 0.5);

    &.n-carousel__dot--active {
      background: #fff;
    }
  }

  :deep(.n-carousel__arrow) {
    background: rgba(255, 255, 255, 0.2);
    color: #fff;

    &:hover {
      background: rgba(255, 255, 255, 0.3);
    }
  }
}

.banner-item {
  height: 100%;
  min-height: 140px;
  padding: 24px;
  display: flex;
  align-items: center;
}

.banner-content {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 100%;
}

.banner-text {
  flex: 1;
}

.banner-title {
  font-size: 20px;
  font-weight: 700;
  color: #fff;
  margin: 0 0 8px 0;
}

.banner-subtitle {
  font-size: 14px;
  color: rgba(255, 255, 255, 0.85);
  margin: 0;
}

.banner-icon {
  opacity: 0.6;
}

.stat-cards {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 20px;
  margin-bottom: 20px;
}

.stat-card {
  :deep(.n-card__content) {
    padding: 20px;
  }
}

.stat-content {
  display: flex;
  align-items: center;
  gap: 16px;
}

.stat-icon {
  width: 52px;
  height: 52px;
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.stat-value {
  font-size: 28px;
  font-weight: 700;
  color: #111827;
  line-height: 1;
}

.stat-title {
  font-size: 14px;
  color: #6B7280;
  margin-top: 4px;
}

.middle-section {
  margin-bottom: 20px;
}

.dashboard-section {
  margin-bottom: 20px;
}

.dashboard-card {
  height: 100%;

  :deep(.n-card__content) {
    padding: 0;
  }

  :deep(.n-data-table) {
    border-radius: 0;
  }

  :deep(.n-data-table__header) {
    border-bottom: 1px solid #e5e7eb;
  }

  :deep(.n-data-table__row) {
    cursor: pointer;

    &:hover {
      background: #f9fafb;
    }
  }
}

.alert-card {
  height: 100%;

  :deep(.n-card__content) {
    padding: 16px;
  }

  :deep(.n-data-table) {
    border-radius: 8px;
    margin-top: 16px;
  }

  :deep(.n-data-table__header) {
    border-bottom: 1px solid #e5e7eb;
  }

  :deep(.n-data-table__row) {
    cursor: pointer;

    &:hover {
      background: #f9fafb;
    }
  }
}

.alert-stats {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 16px;
  margin-bottom: 16px;
}

.alert-stat-item {
  background: #f9fafb;
  padding: 16px;
  border-radius: 8px;
  text-align: center;
}

.alert-stat-value {
  font-size: 24px;
  font-weight: 700;
  color: #111827;
  margin-bottom: 4px;
}

.alert-stat-label {
  font-size: 14px;
  color: #6B7280;
}

.shortcuts-card {
  height: 100%;
}

.shortcuts-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 12px;
}

.shortcut-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 16px;
  border-radius: 12px;
  cursor: pointer;
  transition: all 0.2s;

  &:hover {
    background: #F3F4F6;
  }
}

.shortcut-icon {
  width: 48px;
  height: 48px;
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 8px;
}

.shortcut-name {
  font-size: 13px;
  color: #374151;
  font-weight: 500;
}

.bottom-section {
  margin-bottom: 20px;
}

@media (max-width: 1200px) {
  .welcome-section {
    flex-direction: column;
  }

  .welcome-banner {
    width: 100%;
  }

  .banner-item {
    min-height: 120px;
  }

  .stat-cards {
    grid-template-columns: repeat(2, 1fr);
  }

  .dashboard-section,
  .bottom-section {
    :deep(.n-grid) {
      display: block;
    }

    :deep(.n-gi) {
      margin-bottom: 20px;
    }
  }
}

@media (max-width: 768px) {
  .welcome-header {
    flex-direction: column;
    text-align: center;
  }

  .welcome-time {
    flex-direction: column;
    align-items: center;
    gap: 4px;
  }

  .stat-cards {
    grid-template-columns: 1fr;
  }

  .shortcuts-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

</style>
