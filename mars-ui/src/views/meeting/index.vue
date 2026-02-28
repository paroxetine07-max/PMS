<template>
  <div class="page-container">
    <!-- 会议管理 -->
    <n-card class="meeting-card" size="small">
      <!-- 搜索表单 -->
      <div class="search-form">
        <n-form inline :model="searchForm" label-placement="left">
          <n-form-item label="会议名称">
            <n-input v-model:value="searchForm.title" placeholder="请输入会议名称" clearable />
          </n-form-item>
          <n-form-item label="状态">
            <n-select
              v-model:value="searchForm.status"
              placeholder="请选择状态"
              :options="statusOptions"
              clearable
              style="width: 120px"
            />
          </n-form-item>
          <n-form-item label="日期范围">
            <n-date-picker
              v-model:value="searchForm.dateRange"
              type="daterange"
              clearable
              style="width: 220px"
            />
          </n-form-item>
          <n-form-item>
            <n-space>
              <n-button type="primary" @click="handleSearch">
                <template #icon><n-icon><SearchOutline /></n-icon></template>
                搜索
              </n-button>
              <n-button @click="handleReset">
                <template #icon><n-icon><RefreshOutline /></n-icon></template>
                重置
              </n-button>
            </n-space>
          </n-form-item>
        </n-form>
      </div>

      <!-- 工具栏 -->
      <div class="table-toolbar">
        <n-space>
          <n-button type="primary" @click="handleAddMeeting">
            <template #icon><n-icon><AddOutline /></n-icon></template>
            创建周会
          </n-button>
          <n-button v-if="checkedRowKeys.length > 0" type="error" @click="handleBatchDelete">
            <template #icon><n-icon><TrashOutline /></n-icon></template>
            批量删除({{ checkedRowKeys.length }})
          </n-button>
        </n-space>
      </div>

      <!-- 表格 -->
      <n-data-table
        :columns="columns"
        :data="tableData"
        :loading="loading"
        :row-key="(row: Meeting) => row.id"
        v-model:checked-row-keys="checkedRowKeys"
        remote
      />

      <div class="pagination-container" style="display: flex; justify-content: flex-end; margin-top: 12px">
        <n-pagination
          v-model:page="pagination.page"
          v-model:page-size="pagination.pageSize"
          :item-count="pagination.itemCount"
          :page-sizes="[10, 20, 50, 100]"
          show-size-picker
          show-quick-jumper
          @update:page="handlePageChange"
          @update:page-size="handlePageSizeChange"
        >
          <template #prefix>
            共 {{ pagination.itemCount }} 条
          </template>
        </n-pagination>
      </div>
    </n-card>

    <!-- 新增/编辑会议弹窗 -->
    <n-modal
      v-model:show="modalVisible"
      :title="modalTitle"
      preset="card"
      style="width: 800px"
      :mask-closable="false"
    >
      <n-form
        ref="formRef"
        :model="formData"
        :rules="rules"
        label-placement="left"
        label-width="100"
        class="modal-form"
      >
        <n-grid :cols="2" :x-gap="16">
          <n-gi>
            <n-form-item label="会议名称" path="title">
              <n-input v-model:value="formData.title" placeholder="请输入会议名称" />
            </n-form-item>
          </n-gi>
          <n-gi>
            <n-form-item label="会议类型" path="type">
              <n-select
                v-model:value="formData.type"
                :options="meetingTypeOptions"
                placeholder="请选择会议类型"
              />
            </n-form-item>
          </n-gi>
          <n-gi>
            <n-form-item label="开始时间" path="startTime">
              <n-date-picker
                v-model:value="formData.startTime"
                type="datetime"
                placeholder="请选择开始时间"
              />
            </n-form-item>
          </n-gi>
          <n-gi>
            <n-form-item label="结束时间" path="endTime">
              <n-date-picker
                v-model:value="formData.endTime"
                type="datetime"
                placeholder="请选择结束时间"
              />
            </n-form-item>
          </n-gi>
          <n-gi>
            <n-form-item label="会议地点" path="location">
              <n-input v-model:value="formData.location" placeholder="请输入会议地点" />
            </n-form-item>
          </n-gi>
          <n-gi>
            <n-form-item label="主持人" path="host">
              <n-input v-model:value="formData.host" placeholder="请输入主持人" />
            </n-form-item>
          </n-gi>
          <n-gi span="2">
            <n-form-item label="参会人员" path="attendees">
              <n-select
                v-model:value="formData.attendees"
                multiple
                :options="userOptions"
                placeholder="请选择参会人员"
                filterable
                style="width: 100%"
              />
            </n-form-item>
          </n-gi>
          <n-gi span="2">
            <n-form-item label="关联项目" path="projects">
              <n-select
                v-model:value="formData.projects"
                multiple
                :options="projectOptions"
                placeholder="请选择关联项目"
                filterable
                style="width: 100%"
              />
            </n-form-item>
          </n-gi>
          <n-gi span="2">
            <n-form-item label="会议议程">
              <n-input
                v-model:value="formData.agenda"
                type="textarea"
                :rows="4"
                placeholder="请输入会议议程，每项一行"
              />
            </n-form-item>
          </n-gi>
        </n-grid>
      </n-form>
      <template #footer>
        <n-space justify="end">
          <n-button @click="modalVisible = false">取消</n-button>
          <n-button type="primary" :loading="submitLoading" @click="handleSubmit">确定</n-button>
        </n-space>
      </template>
    </n-modal>

    <!-- 会议详情弹窗 -->
    <n-modal
      v-model:show="detailVisible"
      title="会议详情"
      preset="card"
      style="width: 900px"
      :mask-closable="false"
    >
      <div class="meeting-detail">
        <n-descriptions title="基本信息" :column="2">
          <n-descriptions-item label="会议名称">{{ detailData.title }}</n-descriptions-item>
          <n-descriptions-item label="会议类型">{{ getMeetingTypeLabel(detailData.type) }}</n-descriptions-item>
          <n-descriptions-item label="开始时间">{{ detailData.startTime }}</n-descriptions-item>
          <n-descriptions-item label="结束时间">{{ detailData.endTime }}</n-descriptions-item>
          <n-descriptions-item label="会议地点">{{ detailData.location }}</n-descriptions-item>
          <n-descriptions-item label="主持人">{{ detailData.host }}</n-descriptions-item>
          <n-descriptions-item label="参会人员" :span="2">{{ detailData.attendees?.join(', ') }}</n-descriptions-item>
          <n-descriptions-item label="关联项目" :span="2">{{ detailData.projects?.join(', ') }}</n-descriptions-item>
        </n-descriptions>

        <div class="section-title">会议议程</div>
        <div class="agenda-content">
          <n-list>
            <n-list-item v-for="(item, index) in detailData.agenda?.split('\n')" :key="index">
              {{ item }}
            </n-list-item>
          </n-list>
        </div>

        <div class="section-title">会议纪要</div>
        <div class="minutes-content">
          <n-editor
            v-model:value="detailData.minutes"
            :content-style="{ height: '300px' }"
          />
        </div>

        <div class="section-title">行动项</div>
        <div class="action-items">
          <n-data-table
            :columns="actionItemColumns"
            :data="detailData.actionItems"
            :row-key="(row: ActionItem) => row.id"
          />
          <n-button type="primary" size="small" @click="handleAddActionItem" style="margin-top: 12px">
            <template #icon><n-icon><AddOutline /></n-icon></template>
            添加行动项
          </n-button>
        </div>
      </div>
      <template #footer>
        <n-space justify="end">
          <n-button @click="detailVisible = false">关闭</n-button>
          <n-button type="primary" @click="handleSaveMinutes">保存纪要</n-button>
        </n-space>
      </template>
    </n-modal>

    <!-- 添加行动项弹窗 -->
    <n-modal
      v-model:show="actionItemModalVisible"
      title="添加行动项"
      preset="card"
      style="width: 500px"
      :mask-closable="false"
    >
      <n-form
        ref="actionItemFormRef"
        :model="actionItemForm"
        :rules="actionItemRules"
        label-placement="left"
        label-width="80"
        class="modal-form"
      >
        <n-form-item label="任务内容" path="content">
          <n-input v-model:value="actionItemForm.content" placeholder="请输入任务内容" />
        </n-form-item>
        <n-form-item label="负责人" path="assignee">
          <n-select
            v-model:value="actionItemForm.assignee"
            :options="userOptions"
            placeholder="请选择负责人"
          />
        </n-form-item>
        <n-form-item label="截止时间" path="deadline">
          <n-date-picker
            v-model:value="actionItemForm.deadline"
            type="datetime"
            placeholder="请选择截止时间"
          />
        </n-form-item>
        <n-form-item label="优先级" path="priority">
          <n-select
            v-model:value="actionItemForm.priority"
            :options="priorityOptions"
            placeholder="请选择优先级"
          />
        </n-form-item>
      </n-form>
      <template #footer>
        <n-space justify="end">
          <n-button @click="actionItemModalVisible = false">取消</n-button>
          <n-button type="primary" @click="handleSubmitActionItem">确定</n-button>
        </n-space>
      </template>
    </n-modal>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, h, onMounted, computed, type FormInst, type FormRules } from 'vue'
import { NButton, NTag, NSpace, NPagination, NGrid, NGi, useMessage, useDialog, type DataTableColumns } from 'naive-ui'
import { SearchOutline, RefreshOutline, AddOutline, TrashOutline, CalendarOutline, ClockOutline, MapPinOutline, UserOutline, UsersOutline, FolderOutline, FileTextOutline, CheckmarkCircleOutline } from '@vicons/ionicons5'

// 类型定义
interface Meeting {
  id: number
  title: string
  type: string
  startTime: Date | null
  endTime: Date | null
  location: string
  host: string
  attendees: string[]
  projects: string[]
  agenda: string
  minutes: string
  status: number
  createTime: string
  actionItems: ActionItem[]
}

interface ActionItem {
  id: number
  content: string
  assignee: string
  deadline: Date | null
  priority: string
  status: number
}

const message = useMessage()
const dialog = useDialog()

// ==================== 搜索表单 ====================
const searchForm = reactive({
  title: '',
  status: null as number | null,
  dateRange: null as [string, string] | null
})

const statusOptions = [
  { label: '待召开', value: 0 },
  { label: '进行中', value: 1 },
  { label: '已结束', value: 2 }
]

// ==================== 表格 ====================
const tableData = ref<Meeting[]>([])
const loading = ref(false)
const checkedRowKeys = ref<number[]>([])
const pagination = reactive({
  page: 1,
  pageSize: 10,
  itemCount: 0
})

const meetingTypeOptions = [
  { label: '周会', value: 'weekly' },
  { label: '月会', value: 'monthly' },
  { label: '临时会议', value: 'temporary' },
  { label: '其他', value: 'other' }
]

const priorityOptions = [
  { label: '高', value: 'high' },
  { label: '中', value: 'medium' },
  { label: '低', value: 'low' }
]

// 模拟用户数据
const userOptions = ref([
  { label: '张三', value: '张三' },
  { label: '李四', value: '李四' },
  { label: '王五', value: '王五' },
  { label: '赵六', value: '赵六' }
])

// 模拟项目数据
const projectOptions = ref([
  { label: '项目A', value: '项目A' },
  { label: '项目B', value: '项目B' },
  { label: '项目C', value: '项目C' }
])

const columns: DataTableColumns<Meeting> = [
  { type: 'selection' },
  { title: 'ID', key: 'id', width: 60 },
  { title: '会议名称', key: 'title', width: 200 },
  { 
    title: '会议类型', 
    key: 'type', 
    width: 100, 
    render(row) {
      const typeMap: Record<string, { type: 'info' | 'success' | 'warning'; label: string }> = {
        weekly: { type: 'info', label: '周会' },
        monthly: { type: 'success', label: '月会' },
        temporary: { type: 'warning', label: '临时会议' },
        other: { type: 'default', label: '其他' }
      }
      const t = typeMap[row.type || 'other'] || { type: 'default', label: row.type || '未知' }
      return h(NTag, { type: t.type, size: 'small' }, { default: () => t.label })
    }
  },
  { title: '开始时间', key: 'startTime', width: 170 },
  { title: '结束时间', key: 'endTime', width: 170 },
  { title: '会议地点', key: 'location', width: 120 },
  { title: '主持人', key: 'host', width: 100 },
  { 
    title: '项目名称', 
    key: 'projects', 
    width: 150, 
    render(row) {
      if (!row.projects || row.projects.length === 0) {
        return '-'
      }
      return row.projects.join(', ')
    }
  },
  { 
    title: '状态', 
    key: 'status', 
    width: 80, 
    render(row) {
      const statusMap: Record<number, { type: 'success' | 'error' | 'warning' | 'info'; label: string }> = {
        0: { type: 'info', label: '待召开' },
        1: { type: 'warning', label: '进行中' },
        2: { type: 'success', label: '已结束' }
      }
      const status = statusMap[row.status] || { type: 'info', label: '未知' }
      return h(NTag, { type: status.type, size: 'small' }, { default: () => status.label })
    }
  },
  { title: '创建时间', key: 'createTime', width: 170 },
  {
    title: '操作',
    key: 'actions',
    width: 160,
    fixed: 'right',
    render(row) {
      const buttons = []
      buttons.push(
        h(NButton, { size: 'small', onClick: () => handleViewDetail(row) }, { default: () => '查看' })
      )
      buttons.push(
        h(NButton, { size: 'small', onClick: () => handleEdit(row) }, { default: () => '编辑' })
      )
      buttons.push(
        h(NButton, { size: 'small', type: 'error', onClick: () => handleDelete(row) }, { default: () => '删除' })
      )
      return h(NSpace, null, { default: () => buttons })
    }
  }
]

const actionItemColumns: DataTableColumns<ActionItem> = [
  { title: '任务内容', key: 'content', width: 300 },
  { title: '负责人', key: 'assignee', width: 100 },
  { title: '截止时间', key: 'deadline', width: 170 },
  { 
    title: '优先级', 
    key: 'priority', 
    width: 80, 
    render(row) {
      const priorityMap: Record<string, { type: 'error' | 'warning' | 'success'; label: string }> = {
        high: { type: 'error', label: '高' },
        medium: { type: 'warning', label: '中' },
        low: { type: 'success', label: '低' }
      }
      const p = priorityMap[row.priority || 'medium'] || { type: 'default', label: row.priority || '未知' }
      return h(NTag, { type: p.type, size: 'small' }, { default: () => p.label })
    }
  },
  { 
    title: '状态', 
    key: 'status', 
    width: 80, 
    render(row) {
      const statusMap: Record<number, { type: 'success' | 'info'; label: string }> = {
        0: { type: 'info', label: '待完成' },
        1: { type: 'success', label: '已完成' }
      }
      const status = statusMap[row.status] || { type: 'info', label: '未知' }
      return h(NTag, { type: status.type, size: 'small' }, { default: () => status.label })
    }
  },
  {
    title: '操作',
    key: 'actions',
    width: 80,
    fixed: 'right',
    render(row) {
      return h(NButton, { size: 'small', type: 'success', onClick: () => handleCompleteActionItem(row) }, { default: () => '完成' })
    }
  }
]

// ==================== 弹窗 ====================
const modalVisible = ref(false)
const modalTitle = ref('创建周会')
const formRef = ref<FormInst | null>(null)
const submitLoading = ref(false)

const formData = reactive<Meeting>({
  id: 0,
  title: '',
  type: 'weekly',
  startTime: null,
  endTime: null,
  location: '',
  host: '',
  attendees: [],
  projects: [],
  agenda: '',
  minutes: '',
  status: 0,
  createTime: '',
  actionItems: []
})

const rules: FormRules = {
  title: [{ required: true, message: '请输入会议名称', trigger: 'blur' }],
  startTime: [{ required: true, message: '请选择开始时间', trigger: 'blur' }],
  endTime: [{ required: true, message: '请选择结束时间', trigger: 'blur' }],
  host: [{ required: true, message: '请输入主持人', trigger: 'blur' }]
}

// 会议详情
const detailVisible = ref(false)
const detailData = reactive<Meeting>({
  id: 0,
  title: '',
  type: '',
  startTime: '',
  endTime: '',
  location: '',
  host: '',
  attendees: [],
  projects: [],
  agenda: '',
  minutes: '',
  status: 0,
  createTime: '',
  actionItems: []
})

// 行动项
const actionItemModalVisible = ref(false)
const actionItemFormRef = ref<FormInst | null>(null)
const actionItemForm = reactive<ActionItem>({
  id: 0,
  content: '',
  assignee: '',
  deadline: null,
  priority: 'medium',
  status: 0
})

const actionItemRules: FormRules = {
  content: [{ required: true, message: '请输入任务内容', trigger: 'blur' }],
  assignee: [{ required: true, message: '请选择负责人', trigger: 'blur' }],
  deadline: [{ required: true, message: '请选择截止时间', trigger: 'blur' }]
}

// ==================== 数据加载 ====================
async function loadData() {
  loading.value = true
  try {
    // 模拟数据
    const mockData: Meeting[] = [
      {
        id: 1,
        title: '周会 - 项目进度讨论',
        type: 'weekly',
        startTime: '2026-02-28 10:00:00',
        endTime: '2026-02-28 11:30:00',
        location: '会议室A',
        host: '张三',
        attendees: ['张三', '李四', '王五'],
        projects: ['项目A', '项目B'],
        agenda: '1. 项目A进度汇报\n2. 项目B问题讨论\n3. 下周工作计划',
        minutes: '本次会议讨论了项目A和项目B的进展情况，确定了下周的工作计划。',
        status: 2,
        createTime: '2026-02-27 09:00:00',
        actionItems: [
          {
            id: 1,
            content: '完成项目A的需求文档',
            assignee: '李四',
            deadline: '2026-03-05 18:00:00',
            priority: 'high',
            status: 0
          },
          {
            id: 2,
            content: '修复项目B的bug',
            assignee: '王五',
            deadline: '2026-03-03 18:00:00',
            priority: 'medium',
            status: 1
          }
        ]
      },
      {
        id: 2,
        title: '周会 - 团队建设讨论',
        type: 'weekly',
        startTime: '2026-02-21 10:00:00',
        endTime: '2026-02-21 11:00:00',
        location: '会议室B',
        host: '李四',
        attendees: ['张三', '李四', '王五', '赵六'],
        projects: [],
        agenda: '1. 团队建设活动讨论\n2. 员工福利方案',
        minutes: '本次会议讨论了团队建设活动方案和员工福利计划。',
        status: 2,
        createTime: '2026-02-20 14:00:00',
        actionItems: [
          {
            id: 3,
            content: '制定团队建设活动计划',
            assignee: '张三',
            deadline: '2026-02-25 18:00:00',
            priority: 'medium',
            status: 1
          }
        ]
      },
      {
        id: 3,
        title: '周会 - 技术分享',
        type: 'weekly',
        startTime: '2026-03-06 10:00:00',
        endTime: '2026-03-06 11:30:00',
        location: '会议室A',
        host: '王五',
        attendees: ['张三', '李四', '王五', '赵六'],
        projects: ['项目A', '项目C'],
        agenda: '1. 前端技术分享\n2. 后端架构讨论\n3. 技术难题解决',
        minutes: '',
        status: 0,
        createTime: '2026-02-28 10:00:00',
        actionItems: []
      }
    ]
    
    // 模拟搜索
    let filteredData = [...mockData]
    if (searchForm.title) {
      filteredData = filteredData.filter(item => item.title.includes(searchForm.title))
    }
    if (searchForm.status !== null) {
      filteredData = filteredData.filter(item => item.status === searchForm.status)
    }
    if (searchForm.dateRange) {
      const [start, end] = searchForm.dateRange
      filteredData = filteredData.filter(item => {
        const meetingDate = new Date(item.startTime).toDateString()
        const startDate = new Date(start).toDateString()
        const endDate = new Date(end).toDateString()
        return meetingDate >= startDate && meetingDate <= endDate
      })
    }
    
    // 模拟分页
    const start = (pagination.page - 1) * pagination.pageSize
    const end = start + pagination.pageSize
    tableData.value = filteredData.slice(start, end)
    pagination.itemCount = filteredData.length
  } catch (error) {
    console.error('加载数据失败:', error)
  } finally {
    loading.value = false
  }
}

// ==================== 操作方法 ====================
function handleSearch() {
  pagination.page = 1
  loadData()
}

function handleReset() {
  searchForm.title = ''
  searchForm.status = null
  searchForm.dateRange = null
  handleSearch()
}

function handlePageChange(page: number) {
  pagination.page = page
  loadData()
}

function handlePageSizeChange(pageSize: number) {
  pagination.pageSize = pageSize
  pagination.page = 1
  loadData()
}

function handleAddMeeting() {
  modalTitle.value = '创建周会'
  Object.assign(formData, {
    id: 0,
    title: '',
    type: 'weekly',
    startTime: null,
    endTime: null,
    location: '',
    host: '',
    attendees: [],
    projects: [],
    agenda: '',
    minutes: '',
    status: 0,
    createTime: '',
    actionItems: []
  })
  modalVisible.value = true
}

function handleEdit(row: any) {
  modalTitle.value = '编辑会议'
  const meetingData = {
    ...row,
    startTime: row.startTime ? new Date(row.startTime) : null,
    endTime: row.endTime ? new Date(row.endTime) : null
  }
  Object.assign(formData, meetingData)
  modalVisible.value = true
}

async function handleSubmit() {
  try {
    await formRef.value?.validate()
    submitLoading.value = true
    
    // 模拟提交
    setTimeout(() => {
      message.success(formData.id ? '更新成功' : '创建成功')
      modalVisible.value = false
      loadData()
      submitLoading.value = false
    }, 500)
  } catch (error) {
    console.error('提交失败:', error)
  } finally {
    submitLoading.value = false
  }
}

function handleDelete(row: Meeting) {
  dialog.warning({
    title: '提示',
    content: `确定要删除会议"${row.title}"吗？`,
    positiveText: '确定',
    negativeText: '取消',
    onPositiveClick: () => {
      // 模拟删除
      message.success('删除成功')
      loadData()
    }
  })
}

function handleBatchDelete() {
  if (checkedRowKeys.value.length === 0) {
    message.warning('请选择要删除的会议')
    return
  }
  dialog.warning({
    title: '提示',
    content: `确定要删除选中的 ${checkedRowKeys.value.length} 个会议吗？`,
    positiveText: '确定',
    negativeText: '取消',
    onPositiveClick: () => {
      // 模拟删除
      message.success('删除成功')
      checkedRowKeys.value = []
      loadData()
    }
  })
}

function handleViewDetail(row: any) {
  const meetingData = {
    ...row,
    startTime: row.startTime ? new Date(row.startTime) : null,
    endTime: row.endTime ? new Date(row.endTime) : null,
    actionItems: row.actionItems.map((item: any) => ({
      ...item,
      deadline: item.deadline ? new Date(item.deadline) : null
    }))
  }
  Object.assign(detailData, meetingData)
  detailVisible.value = true
}

function handleSaveMinutes() {
  // 模拟保存
  message.success('会议纪要保存成功')
}

function handleAddActionItem() {
  Object.assign(actionItemForm, {
    id: 0,
    content: '',
    assignee: '',
    deadline: '',
    priority: 'medium',
    status: 0
  })
  actionItemModalVisible.value = true
}

async function handleSubmitActionItem() {
  try {
    await actionItemFormRef.value?.validate()
    
    // 模拟添加行动项
    const newActionItem: ActionItem = {
      ...actionItemForm,
      id: Date.now()
    }
    detailData.actionItems.push(newActionItem)
    actionItemModalVisible.value = false
    message.success('行动项添加成功')
  } catch (error) {
    console.error('提交失败:', error)
  }
}

function handleCompleteActionItem(row: ActionItem) {
  row.status = 1
  message.success('行动项已标记为完成')
}

function getMeetingTypeLabel(type: string): string {
  const typeMap: Record<string, string> = {
    weekly: '周会',
    monthly: '月会',
    temporary: '临时会议',
    other: '其他'
  }
  return typeMap[type] || type
}

onMounted(() => {
  loadData()
})
</script>

<style scoped>
.meeting-detail {
  .section-title {
    font-size: 16px;
    font-weight: 500;
    margin: 20px 0 10px 0;
    color: #374151;
  }
  
  .agenda-content,
  .minutes-content {
    background: #f9fafb;
    padding: 16px;
    border-radius: 8px;
    margin-bottom: 16px;
  }
  
  .action-items {
    margin-top: 16px;
  }
}

body.dark-theme {
  .meeting-detail {
    .section-title {
      color: #ffffffd1;
    }
    
    .agenda-content,
    .minutes-content {
      background: #18181c;
    }
  }
}
</style>