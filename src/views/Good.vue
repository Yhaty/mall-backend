<template>
  <el-card class="swiper-container">
    <template #header>
      <div class="header">
        <el-button type="primary" @click="handleAdd"
          ><el-icon><Plus /></el-icon>新增商品</el-button
        >
      </div>
    </template>
    <el-table
      :load="()=>state.loading"
      :data="state.tableData"
      tooltip-effect="dark"
      style="width: 100%"
    >
      <el-table-column prop="goodsId" label="商品编号"> </el-table-column>
      <el-table-column prop="goodsName" label="商品名"> </el-table-column>
      <el-table-column prop="goodsIntro" label="商品简介"> </el-table-column>
      <el-table-column label="商品图片" width="150px">
        <template #default="scope">
          <img
            style="width: 100px; height: 100px"
            :key="scope.row.goodsId"
            :src="$filters.prefix(scope.row.goodsCoverImg)"
            alt="商品主图"
          />
        </template>
      </el-table-column>
      <el-table-column prop="stockNum" label="商品库存"> </el-table-column>
      <el-table-column prop="sellingPrice" label="商品售价"> </el-table-column>
      <el-table-column label="上架状态">
        <template #default="scope">
          <span style="color: green" v-if="scope.row.goodsSellStatus == 0"
            >销售中</span
          >
          <span style="color: red" v-else>已下架</span>
        </template>
      </el-table-column>

      <el-table-column label="操作" width="100">
        <template #default="scope">
          <a
            style="cursor: pointer; margin-right: 10px"
            @click="handleEdit(scope.row.goodsId)"
            >修改</a
          >
          <a
            style="cursor: pointer; margin-right: 10px"
            v-if="scope.row.goodsSellStatus == 0"
            @click="handleStatus(scope.row.goodsId, 1)"
            >下架</a
          >
          <a
            style="cursor: pointer; margin-right: 10px"
            v-else
            @click="handleStatus(scope.row.goodsId, 0)"
            >上架</a
          >
        </template>
      </el-table-column>
    </el-table>
    <!--总数超过一页，再展示分页器-->
    <el-pagination
      background
      layout="prev, pager, next"
      :total="state.total"
      :page-size="state.pageSize"
      :current-page="state.currentPage"
      @current-change="changePage"
    />
  </el-card>
</template>

<script setup>
import { onMounted, reactive, ref } from 'vue'
import axios from '@/utils/axios'
import { ElMessage } from 'element-plus'
import { useRouter } from 'vue-router'

const router = useRouter() // 获取路由实例，内涵路由相关的各种方法。
const state = reactive({
  loading: false, // 列表数据接口返回前的 loadinig
  tableData: [], // 数据列表
  total: 0, // 总条数
  currentPage: 1, // 当前页
  pageSize: 10 // 分页大小
})
// 初始化钩子函数
onMounted(() => {
  getGoodList()
})
// 获取轮播图列表
const getGoodList = () => {
  state.loading = true
  axios.get('/goods/list', {
    params: {
      pageNumber: state.currentPage, // 当前页
      pageSize: state.pageSize, // 每页数量
    }
  }).then(res => {
    state.tableData = res.list // 列表数据
    state.total = res.totalCount // 数据总条数
    state.currentPage = res.currPage // 当前页
    state.loading = false // 数据成功返回后，将列表 loading 清除
    const main = document.querySelector('.main')
    main.scrollTop = 0
  })
}
// 添加商品，跳转到 /add 路径下
const handleAdd = () => {
  router.push({ path: '/add' })
}
// 编辑商品，带 id 跳转 /add 路径
const handleEdit = (id) => {
  router.push({ path: '/add', query: { id } })
}
// 翻页方法
const changePage = (val) => {
  state.currentPage = val
  getGoodList()
}
// 上下架方法
const handleStatus = (id, status) => {
  axios.put(`/goods/status/${status}`, {
    ids: id ? [id] : []
  }).then(() => {
    ElMessage.success('修改成功')
    getGoodList()
  })
}
</script>
