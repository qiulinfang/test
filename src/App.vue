<template>
  <div class="common-layout">
    <el-container>
      <el-header class="header">
        <el-image style="width: 120px; height: 50px" :src="imgUrl" :fit="fit" />
      </el-header>
      <el-breadcrumb class="breadcrumb" :separator-icon="ArrowRight">
        <el-breadcrumb-item>主页</el-breadcrumb-item>
        <el-breadcrumb-item>主目录分类</el-breadcrumb-item>
      </el-breadcrumb>
      <el-main class="main" style="padding: 0;">
        <el-divider content-position="left">功能面板</el-divider>
        <div class="filter">
          <div class="searchScope">
            <div>
              搜索：<el-input v-model="directorySearchKeyword" style="width: 200px;" placeholder="搜索主目录"
                :prefix-icon="Search" />
            </div>
            <div class="m-4" style="margin-left: 20px;">
              状态:
              <el-cascader v-model="statusSearchKeyword" :options="options" style="width: 200px;" clearable />
            </div>
          </div>
          <el-button type="primary" @click="filterData">确认筛选</el-button>
        </div>
        <div class="tableContainer">
          <el-table :data="displayedData" stripe style="width: 100%" :header-cell-style="headerCellStyle"
            @row-click="handleRowClick" border>
            <el-table-column prop="id" label="ID" width="180" align="center" />
            <el-table-column label="主目录分类" width="220" align="center">
              <template #default="{ row }">
                {{ row.mainDirectoryCategoryE }},{{ row.mainDirectoryCategoryC }}
              </template>
            </el-table-column>
            <el-table-column prop="shoppingSiteSorting" label="购物网站排序" width="180" align="center" sortable />
            <el-table-column prop="status" label="状态" align="center">
              <template #default="{ row }">
                {{ row.status === '1' ? "启用" : "禁用" }}
              </template>
            </el-table-column>
            <el-table-column label="操作" min-width="120" align="center">
              <template #default="{ row }">
                <el-button type="primary" size="small" @click="editData(row)">修改</el-button>
                <el-button type="danger" size="small" @click="deleteData(row.id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
          <div class="tableFooter">
            <div class="demo-pagination-block">
              <el-pagination v-model:current-page="currentPage" v-model:page-size="pageSize" :page-sizes="pageSizes"
                :size="size" :disabled="disabled" :background="background"
                layout="total, sizes, prev, pager, next, jumper" :total="total" />
            </div>
            <el-button type="primary" @click="addData" class="addDataButton">新增</el-button>
          </div>
        </div>
        <el-dialog v-model="dialogVisible">
          <el-form :model="form" label-width="auto" style="max-width: 700px" require-asterisk-position="right">
            <el-form-item label="主目录分类名称（中文）" required="true">
              <el-input maxlength="50" show-word-limit v-model="form.mainDirectoryCategoryE" />
            </el-form-item>
            <el-form-item label="主目录分类名称（英文）" required="true">
              <el-input maxlength="50" show-word-limit v-model="form.mainDirectoryCategoryC" />
            </el-form-item>
            <el-form-item label="购物网站排序" required="true">
              <el-input-number v-model="form.shoppingSiteSorting" class="mx-4" :min="1" 
                controls-position="right" @change="handleChange" />
            </el-form-item>
            <el-form-item label="状态" required="true">
              <el-radio-group v-model="form.status" class="ml-4">
                <el-radio value="1" size="large">启用</el-radio>
                <el-radio value="2" size="large">禁用</el-radio>
              </el-radio-group>
            </el-form-item>
          </el-form>
          <template #footer>
            <span class="dialog-footer">
              <el-button @click="dialogVisible = false">取 消</el-button>
              <el-button type="primary" @click="submitForm">提 交</el-button>
            </span>
          </template>
        </el-dialog>
      </el-main>
    </el-container>
  </div>

</template>

<script setup>
import { ref, reactive, onMounted, computed,watch  } from 'vue'
import axios from 'axios';
import myImage from '@/assets/logo.png';
import { ArrowRight, ArrowDown, Search } from '@element-plus/icons-vue'

//头部
const imgUrl = ref(myImage);

//过滤器数据
const directorySearchKeyword = ref('')
const statusSearchKeyword = ref([])
const filteredData = ref([])

const options = [
  {
    value: '3',
    label: '全选',
  },
  {
    value: '1',
    label: '启用',
  },
  {
    value: '2',
    label: '禁用',
  },
]

//表格数据和行为
const tableData = ref([
])


const headerCellStyle = () => ({
  backgroundColor: 'red',
  color: 'black'
});

const fetchData = async () => {
  try {
    const response = await axios.get('http://localhost:3000/posts');
    tableData.value = response.data;
  } catch (err) {
    console.error("Error fetching data:", err);
  }
}

const deleteData = async (id) => {
  try {
    await axios.delete(`http://localhost:3000/posts/${id}`);
    await fetchData();
    filterData();
  } catch (err) {
    console.error("Error fetching data:", err);
  }
}

const addData = async () => {
  resetForm();
  dialogVisible.value = true;
}

const editData = (row) => {
  form.id = row.id;
  form.mainDirectoryCategoryE = row.mainDirectoryCategoryE;
  form.mainDirectoryCategoryC = row.mainDirectoryCategoryC;
  form.shoppingSiteSorting = row.shoppingSiteSorting;
  form.status = row.status;
  
  dialogVisible.value = true;
};

const filterData = () => {
  if (directorySearchKeyword.value === '' && (typeof statusSearchKeyword.value === 'undefined' || statusSearchKeyword.value.length === 0)) {
    filteredData.value = tableData.value
  } else if (statusSearchKeyword.value[0] === '3') {
    filteredData.value = tableData.value.filter(item => {
      return (item.mainDirectoryCategoryE === directorySearchKeyword.value ||
        item.mainDirectoryCategoryC === directorySearchKeyword.value
      )
    });
  } else {
    filteredData.value = tableData.value.filter(item => {
      return (item.mainDirectoryCategoryE === directorySearchKeyword.value ||
        item.mainDirectoryCategoryC === directorySearchKeyword.value
      ) && item.status === statusSearchKeyword.value[0]
    });
  }
  total.value = filteredData.value.length;
}

const resetForm = () => {
  form.id = null;
  form.mainDirectoryCategoryE = '';
  form.mainDirectoryCategoryC = '';
  form.shoppingSiteSorting = '';
  form.status = '';
};

// 分页器数据
const total = ref(filteredData.value.length)
const pageSizes = [10, 50, 100]
const currentPage = ref(1)
const pageSize = ref(10)
const size = ref('default')
const disabled = ref(false)
const background = ref(false)
const displayedData = computed(() => {
  const start = (currentPage.value - 1) * pageSize.value;
  const end = start + pageSize.value;
  return filteredData.value.slice(start, end);
});

// 模态框数据和行为
const dialogVisible = ref(false)
const form = reactive({
  id: null,
  mainDirectoryCategoryE: '',
  mainDirectoryCategoryC: '',
  shoppingSiteSorting: '',
  status: '',
})

const submitForm = async () => {
  try {
    if (form.id) {
      // 修改现有记录
      await axios.put(`http://localhost:3000/posts/${form.id}`, form);
    } else {
      // 添加新记录
      const { id, ...newObj } = form;
      await axios.post('http://localhost:3000/posts', newObj);
    }
    dialogVisible.value = false;
  } catch (error) {
    console.error(error);
  }
  await fetchData();
  filterData();
}

//其他
onMounted(async () => {
  await fetchData();
  filterData();
});
</script>

<style scoped>
.header {
  background-color: black;
}

.filter {
  display: flex;
  justify-content: space-between;
  margin-bottom: 10px;
}

.searchScope {
  display: flex;
  justify-content: space-between;
}

.breadcrumb {
  display: flex;
  align-items: center;
  height: 30px;
  background-color: rgb(221, 221, 221);
}


.checkbox {
  display: flex;
  flex-direction: column;
}

.tableFooter {
  display: flex;
  justify-content: center;
  margin-top: 50px;
}

.example-showcase .el-dropdown-link {
  cursor: pointer;
  color: var(--el-color-primary);
  display: flex;
  align-items: center;
}

.demo-pagination-block {
  padding-top: 10px;
  border-top: 1px solid gainsboro;
}

.addDataButton {
  position: absolute;
  right: 0;
}
</style>