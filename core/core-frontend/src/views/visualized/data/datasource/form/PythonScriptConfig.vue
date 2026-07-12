<script setup lang="ts">
import { ref, onMounted } from 'vue'
import {
  uploadPythonScript,
  listPythonScripts,
  deletePythonScript,
  associatePythonScript,
  getDatasourcePythonScript,
  removeDatasourcePythonScript,
  listPythonScriptLogs
} from '@/api/datasource'
import { ElMessage, ElMessageBox } from 'element-plus-secondary'

const props = defineProps({
  datasourceId: {
    type: [String, Number],
    required: true
  }
})

const scripts = ref([])
const currentScriptId = ref(null)
const uploadLoading = ref(false)
const logs = ref([])
const fileInput = ref(null)

const loadScripts = async () => {
  const data = await listPythonScripts()
  scripts.value = data || []
}

const loadAssociation = async () => {
  if (!props.datasourceId || props.datasourceId === '0') return
  const data = await getDatasourcePythonScript(props.datasourceId)
  currentScriptId.value = data?.id || null
}

const loadLogs = async () => {
  if (!props.datasourceId || props.datasourceId === '0') return
  const data = await listPythonScriptLogs({ datasourceId: props.datasourceId })
  logs.value = data || []
}

const handleFileChange = async (event: Event) => {
  const target = event.target as HTMLInputElement
  const file = target.files?.[0]
  if (!file) return
  if (!file.name.toLowerCase().endsWith('.py')) {
    ElMessage.warning('仅支持上传 .py 文件')
    return
  }
  const formData = new FormData()
  formData.append('file', file)
  uploadLoading.value = true
  try {
    await uploadPythonScript(formData)
    ElMessage.success('上传成功')
    await loadScripts()
  } catch (e) {
    console.error(e)
  } finally {
    uploadLoading.value = false
    target.value = ''
  }
}

const handleDelete = async (script: any) => {
  try {
    await ElMessageBox.confirm('确认删除该脚本？', '提示', { type: 'warning' })
    await deletePythonScript(script.id)
    ElMessage.success('删除成功')
    await loadScripts()
    await loadAssociation()
  } catch (e) {
    // cancelled
  }
}

const handleAssociate = async () => {
  if (!props.datasourceId || props.datasourceId === '0') {
    ElMessage.warning('请先保存数据源')
    return
  }
  await associatePythonScript({
    datasourceId: props.datasourceId,
    scriptId: currentScriptId.value,
    enabled: 1
  })
  ElMessage.success('关联成功')
  await loadAssociation()
}

const handleRemove = async () => {
  await removeDatasourcePythonScript(props.datasourceId)
  currentScriptId.value = null
  ElMessage.success('已取消关联')
  await loadAssociation()
}

onMounted(() => {
  loadScripts()
  loadAssociation()
  loadLogs()
})

const formatTime = (time: number) => {
  if (!time) return '-'
  return new Date(time).toLocaleString()
}
</script>

<template>
  <div class="python-script-config">
    <div class="section">
      <div class="section-title">Python 导入后脚本</div>
      <div class="upload-row">
        <input
          ref="fileInput"
          type="file"
          accept=".py"
          style="display: none"
          @change="handleFileChange"
        />
        <ElButton :loading="uploadLoading" @click="fileInput?.click()">上传脚本</ElButton>
      </div>

      <div class="associate-row">
        <ElSelect v-model="currentScriptId" placeholder="选择脚本" clearable style="width: 240px">
          <ElOption
            v-for="script in scripts"
            :key="script.id"
            :label="script.name"
            :value="script.id"
          />
        </ElSelect>
        <ElButton type="primary" @click="handleAssociate">保存关联</ElButton>
        <ElButton v-if="currentScriptId" @click="handleRemove">取消关联</ElButton>
      </div>

      <div v-if="scripts.length" class="script-list">
        <div v-for="script in scripts" :key="script.id" class="script-item">
          <span>{{ script.name }}</span>
          <ElButton type="danger" link @click="handleDelete(script)">删除</ElButton>
        </div>
      </div>
    </div>

    <div class="section">
      <div class="section-title">执行日志</div>
      <ElTable :data="logs" stripe max-height="300">
        <ElTableColumn prop="engineTableName" label="引擎表" show-overflow-tooltip />
        <ElTableColumn prop="status" label="状态" width="100">
          <template #default="{ row }">
            <ElTag
              :type="
                row.status === 'Success'
                  ? 'success'
                  : row.status === 'Failed' || row.status === 'Timeout'
                  ? 'danger'
                  : 'info'
              "
              >{{ row.status }}</ElTag
            >
          </template>
        </ElTableColumn>
        <ElTableColumn prop="createTime" label="执行时间" width="160">
          <template #default="{ row }">{{ formatTime(row.createTime) }}</template>
        </ElTableColumn>
        <ElTableColumn prop="errorMsg" label="错误信息" show-overflow-tooltip />
      </ElTable>
    </div>
  </div>
</template>

<style scoped lang="less">
.python-script-config {
  .section {
    margin-bottom: 16px;
  }
  .section-title {
    font-weight: 500;
    margin-bottom: 8px;
  }
  .upload-row,
  .associate-row {
    display: flex;
    gap: 8px;
    margin-bottom: 12px;
    align-items: center;
  }
  .script-list {
    border: 1px solid #e4e7ed;
    border-radius: 4px;
    padding: 8px;
  }
  .script-item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 4px 0;
  }
}
</style>
