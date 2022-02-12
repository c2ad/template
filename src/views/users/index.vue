<template>
  <el-card>
    <!-- 头部 -->
    <el-row :gutter="20" class="header">
      <el-col :span="7">
        <el-input :placeholder="$t('table.placeholder')" clearable v-model="queryForm.query"></el-input>
      </el-col>
      <el-button type="primary" :icon="Search" @click="initGetUsersList">{{ $t('table.search') }}</el-button>
      <el-button type="primary" @click="handleDialogValue()">{{ $t('table.adduser') }}</el-button>
    </el-row>
    <!-- 表格 -->
    <el-table :data="tableData" stripe style="width: 100%">
      <el-table-column :prop="item.prop" :width="item.width" :label="$t(`table.${item.label}`)" v-for="(item, index) in options" :key="index">
        <template v-slot="{ row }" v-if="item.prop === 'mg_state'">
          <el-switch v-model="row.mg_state" @change="changeState(row)" />
        </template>
        <template v-slot="{ row }" v-else-if="item.prop === 'create_time'">
          {{ $filters.filterTimes(row.create_time) }}
        </template>
        <template #default="{ row }" v-else-if="item.prop === 'action'">
          <el-button type="primary" size="small" :icon="Edit" @click="handleDialogValue(row)"></el-button>
          <el-button type="warning" size="small" :icon="Setting"></el-button>
          <el-button type="danger" size="small" :icon="Delete" @click="delUser(row)"></el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页器 -->
    <el-pagination
      v-model:currentPage="queryForm.pagenum"
      v-model:page-size="queryForm.pagesize"
      :page-sizes="[5, 10, 15, 20]"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total"
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
    >
    </el-pagination>
  </el-card>
  <Dialog v-model="dialogVisible" :dialogTitle="dialogTitle" :dialogTableValue="dialogTableValue" v-if="dialogVisible" @initUserList="initGetUsersList" />
</template>

<script setup>
import { Search, Edit, Setting, Delete } from '@element-plus/icons-vue'
import { reactive, ref } from 'vue'
import { getUser, changeUserState, deleteUser } from '@/api/users'
import { options } from './options'
import { ElMessage, ElMessageBox } from 'element-plus'
import { useI18n } from 'vue-i18n'
import Dialog from './components/dialog.vue'
import { isNull } from '@/utils/filters'

const i18n = useI18n()

const queryForm = reactive({
  query: '',
  pagenum: 1,
  pagesize: 5
})
const total = ref(0)

const dialogTitle = ref('')
const dialogVisible = ref(false)
const dialogTableValue = ref({})
const handleDialogValue = (row) => {
  if (isNull(row)) {
    dialogTitle.value = '添加用户'
    dialogTableValue.value = {}
  } else {
    dialogTitle.value = '编辑用户'
    dialogTableValue.value = JSON.parse(JSON.stringify(row))
  }
  dialogVisible.value = true
}

const tableData = ref([])
const initGetUsersList = async () => {
  const res = await getUser(queryForm)
  tableData.value = res.users
  total.value = res.total
}
initGetUsersList()

const handleSizeChange = (pageSize) => {
  queryForm.pagenum = 1
  queryForm.pagesize = pageSize
  initGetUsersList()
}

const handleCurrentChange = (pageNum) => {
  queryForm.pagenum = pageNum
  initGetUsersList()
}

const changeState = async (info) => {
  await changeUserState(info.id, info.mg_state)
  ElMessage({
    message: i18n.t('message.updeteSuccess'),
    type: 'success'
  })
}

const delUser = (row) => {
  ElMessageBox.confirm(i18n.t('dialog.deleteTitle'), {
    confirmButtonText: 'OK',
    cancelButtonText: 'Cancel',
    type: 'warning'
  })
    .then(async () => {
      await deleteUser(row.id)
      ElMessage({
        type: 'success',
        message: 'Delete completed'
      })
      initGetUsersList()
    })
    .catch(() => {
      ElMessage({
        type: 'info',
        message: 'Delete canceled'
      })
    })
}
</script>

<style lang="scss" scoped>
.header {
  padding-bottom: 16px;
  box-sizing: border-box;
}
::v-deep .el-input__suffix {
  align-items: center;
}
::v-deep .el-pagination {
  padding-top: 16px;
  box-sizing: border-box;
  justify-content: flex-end;
}
</style>
