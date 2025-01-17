<template>
  <div class="system-role-container layout-padding">
    <div class="system-role-padding layout-padding-auto layout-padding-view">
      <div class="system-user-search mb15">
        <el-form ref="tableSearchRef" :model="searchState.form" :rules="searchState.rules">
          <el-row>
            <el-col :xs="8" :sm="12" :md="8" :lg="6" :xl="4" class="mb20">
              <el-form-item :label="t('message.app.name')" prop="appName">
                <el-select v-model="searchState.form.appId" filterable placeholder="" size="default" style="width: 90%">
                  <el-option
                    v-for="item in appState.list"
                    :key="item.id"
                    :label="item.label"
                    :value="item.id"
                    @click="onSearch(tableSearchRef)"
                  />
                </el-select>
              </el-form-item>
            </el-col>
            <el-col :xs="8" :sm="12" :md="8" :lg="6" :xl="4" class="mb20">
              <el-form-item :label="t('message.app.name')" prop="name">
                <el-input v-model="searchState.form.name" size="default"
                          style="width: 95%"></el-input>
              </el-form-item>
            </el-col>
            <el-col :xs="8" :sm="12" :md="8" :lg="6" :xl="4" class="mb20">
              <el-button size="default" type="primary" class="ml10"
                         @click="onSearch(tableSearchRef)">
                <el-icon>
                  <ele-Search/>
                </el-icon>
                {{ $t('message.commonBtn.query') }}
              </el-button>
              <el-button size="default" type="primary" class="ml10"
                         @click="onReset()">
                <el-icon>
                  <ele-RefreshRight/>
                </el-icon>
                {{ $t('message.commonBtn.reset') }}
              </el-button>
              <el-button size="default" type="success" class="ml10" @click="onOpenAddRole('add')">
                <el-icon>
                  <ele-FolderAdd/>
                </el-icon>
                {{ $t('message.commonBtn.add') }}
              </el-button>
            </el-col>
          </el-row>
        </el-form>
      </div>
      <el-table :data="state.tableData.data" v-loading="state.tableData.loading"
                style="width: 100%">
        <el-table-column prop="jobId" :label="t('message.job.instance.jobId')" show-overflow-tooltip></el-table-column>
        <el-table-column prop="status" :label="t('message.job.instance.status')" show-overflow-tooltip></el-table-column>
        <el-table-column prop="workerAddress" :label="t('message.job.instance.workerAddress')" show-overflow-tooltip></el-table-column>
        <el-table-column prop="executeTime" :label="t('message.job.instance.executeTime')" show-overflow-tooltip></el-table-column>
        <el-table-column prop="completeTime" :label="t('message.job.instance.completeTime')" show-overflow-tooltip></el-table-column>
        <el-table-column prop="lastReportTime" :label="t('message.job.instance.lastReportTime')" show-overflow-tooltip></el-table-column>
        <el-table-column prop="createTime" :label="t('message.job.instance.createTime')" show-overflow-tooltip></el-table-column>
        <el-table-column label="操作" width="260">
          <template #default="scope">
            <el-button type="primary" size="default" @click="onOpenEditRole('update',scope.row)">
              <el-icon>
                <ele-Edit/>
              </el-icon>
              {{ $t('message.commonBtn.update') }}
            </el-button>
            <el-button type="success" size="default" @click="onOpenEditRole('update',scope.row)">
              <el-icon>
                <ele-Edit/>
              </el-icon>
              {{ $t('message.job.job.instanceBtn') }}
            </el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
        @size-change="onHandleSizeChange"
        @current-change="onHandleCurrentChange"
        class="mt15"
        :pager-count="5"
        :page-sizes="[10, 20, 30]"
        v-model:current-page="state.tableData.param.pageNum"
        background
        v-model:page-size="state.tableData.param.pageSize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="state.tableData.total"
      >
      </el-pagination>
    </div>
    <JobDrawer ref="JobDrawerRef" @refresh="getTableData()"/>
  </div>
</template>

<script setup lang="ts" name="systemRole">
import {defineAsyncComponent, reactive, onMounted, ref} from 'vue';
import {ElMessageBox, ElMessage, FormInstance} from 'element-plus';
import {useI18n} from 'vue-i18n';
import {Local} from '/@/utils/storage';
import {useAppApi} from "/@/api/app";
import {useRouter} from "vue-router";
import {useJobApi, useJobInstanceApi} from "/@/api/job";
import {formatDateByTimestamp} from "/@/utils/formatTime";


// router
const router = useRouter();


// 定义变量内容
const {t} = useI18n();

// 定义接口
const instanceApi = useJobInstanceApi();
const appApi = useAppApi();

// 定义变量内容
const tableSearchRef = ref<FormInstance>();

// 引入组件
const JobDrawer = defineAsyncComponent(() => import('/@/views/job/instance/drawer.vue'));

// 定义变量内容
const nsDialogRef = ref();
const JobDrawerRef = ref();

onMounted(()=>{
  initAppList();
});

const appState = reactive<any>({
  list: [],
});

const searchState = reactive({
  form: {
    appId: '',
    namespaceId: Local.get("nid"),
    name: ''
  },
  rules: {
  },
});

const state = reactive<JobInstanceState>({
  tableData: {
    data: [],
    total: 0,
    loading: false,
    param: {
      pageNum: 1,
      pageSize: 20,
    },
  },
});
// 初始化表格数据
const getTableData = async () => {
  state.tableData.loading = true;
  let data = await instanceApi.getList({
    namespaceId: Local.get("nid"),
    appId: searchState.form.appId,
    name: searchState.form.name,
    page: state.tableData.param.pageNum,
    size: state.tableData.param.pageSize,
  });

  // 清空列表数据
  state.tableData.data = [];
  data.list.forEach(function (item: Object) {
    state.tableData.data.push({
      id: item['id'],
      workflowId: item['workflowId'],
      namespaceId: item['namespaceId'],
      appId: item['appId'],
      jobId: item['jobId'],
      processorType: item['processorType'],
      processorInfo: item['processorInfo'],
      executeType: item['executeType'],
      paramsType: item['paramsType'],
      params: item['params'],
      extendParamsType: item['extendParamsType'],
      extendParams: item['extendParams'],
      timeExpressionType: item['timeExpressionType'],
      timeExpression: item['timeExpression'],
      executeStrategy: item['executeStrategy'],
      failRetryTimes: item['failRetryTimes'],
      failRetryInterval: item['failRetryInterval'],
      concurrency: item['concurrency'],
      status: item['status'],
      workerAddress: item['workerAddress'],
      executeTime: formatDateByTimestamp(item['executeTime']),
      completeTime: formatDateByTimestamp(item['completeTime']),
      lastReportTime: formatDateByTimestamp(item['lastReportTime']),
      createTime: formatDateByTimestamp(item['createTime']),
    })
  });

  state.tableData.total = data.total;
  setTimeout(() => {
    state.tableData.loading = false;
  }, 500);
};

const initAppList = async ()=>{
  let data = await appApi.getList({
    namespaceId: searchState.form.namespaceId,
    page: 1,
    size: 30,
  });

  appState.list = [];
  data.list.forEach(function (item: Object) {
    // 列表数据
    appState.list.push({
      id: item['id'],
      label: item['name']
    })
  });
};

const onSearch = (formEl: FormInstance | undefined) => {
  if (!formEl) return;
  formEl.validate((valid: boolean) => {
    if (valid) {
      getTableData();
    } else {
      return false;
    }
  });
};

const onReset = () => {
  searchState.form.name = '';
  searchState.form.appId = '';
  getTableData();
};

const onMoreCommand = (command :string, row :RowJobType)=>{
  if (command === 'execute'){
    nsDialogRef.value.openDialog(row);
    return ;
  }

  if (command === 'copy'){
    JobDrawerRef.value.openDrawer('copy', searchState.form.appId, row);
    return ;
  }

  if (command === 'delete'){
    ElMessage.success('delete');
    return ;
  }
};

// 打开新增角色弹窗
const onOpenAddRole = (type: string) => {
  JobDrawerRef.value.openDrawer(type, searchState.form.appId);
  // router.push({
  //   path: '/admin/job/page',
  //   params: {
  //     type: type
  //   }
  // })
};
// 打开修改角色弹窗
const onOpenEditRole = (type: string, row: Object) => {
  JobDrawerRef.value.openDrawer(type, searchState.form.appId, row);
};
// 删除角色
const onDel = (row: RowNamespaceType) => {
  ElMessageBox.confirm(`此操作将永久删除角色名称：“${row.name}”，是否继续?`, '提示', {
    confirmButtonText: '确认',
    cancelButtonText: '取消',
    type: 'warning',
  })
    .then(async () => {
      await instanceApi.delete({
        "id": row.id,
      });

      await getTableData();
      ElMessage.success('删除成功');
    })
    .catch(() => {
    });
};
// 分页改变
const onHandleSizeChange = (val: number) => {
  state.tableData.param.pageSize = val;
  getTableData();
};
// 分页改变
const onHandleCurrentChange = (val: number) => {
  state.tableData.param.pageNum = val;
  getTableData();
};
// 页面加载时
onMounted(() => {
  getTableData();
});
</script>

<style scoped lang="scss">
.system-role-container {
  .system-role-padding {
    padding: 15px;

    .el-table {
      flex: 1;
    }
  }
}
</style>
