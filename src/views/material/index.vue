<template>
  <el-card v-loading="loading">
    <bread-crumb slot="header">
      <template slot="title">素材管理</template>
    </bread-crumb>
    <el-row type="flex" justify="end">
      <el-upload :http-request="uploadImg" :show-file-list="false" action>
        <el-button size="small" type="primary">点击上传</el-button>
      </el-upload>
    </el-row>
    <!-- 素材 -->
    <el-tabs v-model="activeName" @tab-click="changeTab">
      <el-tab-pane label="全部素材" name="all">
        <!-- 全部素材内容 定制 -->
        <div class="img-list">
          <!-- v-for -->
          <el-card class="img-card" v-for="item in list" :key="item.id">
            <img :src="item.url" alt />
            <!-- v-bind:style   根据收藏状态决定显示图标的颜色 -->
            <el-row class="operate" type="flex" align="middle" justify="space-around">
              <i
                @click="collectOrCancel(item)"
                :style="{color:item.is_collected ? 'red':''}"
                class="el-icon-star-on"
              ></i>
              <i @click="delMaterial(item.id)" class="el-icon-delete-solid"></i>
            </el-row>
          </el-card>
        </div>
        <el-row type="flex" justify="center" style="height:80px" align="middle">
          <el-pagination
            background
            layout="prev, pager, next"
            :total="page.total"
            :current-page="page.currentPage"
            :page-size="page.pageSize"
            @current-change="changePage"
          ></el-pagination>
        </el-row>
      </el-tab-pane>
      <el-tab-pane label="收藏素材" name="collect">
        <!-- 收藏素材内容 -->
        <div class="img-list">
          <!-- v-for -->
          <el-card class="img-card" v-for="item in list" :key="item.id">
            <img :src="item.url" alt />
          </el-card>
        </div>
        <el-row type="flex" justify="center" style="height:80px" align="middle">
          <el-pagination
            background
            layout="prev, pager, next"
            :total="page.total"
            :current-page="page.currentPage"
            :page-size="page.pageSize"
            @current-change="changePage"
          ></el-pagination>
        </el-row>
      </el-tab-pane>
    </el-tabs>
  </el-card>
</template>

<script>
export default {
  data () {
    return {
      loading: false,
      activeName: 'all', // 默认选中全部
      list: [], // 接收全部数据
      page: {
        currentPage: 1,
        pageSize: 8,
        total: 0
      }
    }
  },
  methods: {
    // 定义一个删除方法
    delMaterial (id) {
      this.$confirm('您确定要删除该素材吗').then(() => {
        // 只有点击了确定 才会执行
        // 调用删除接口
        this.$axios({
          url: `/user/images/${id}`,
          method: 'delete'
        }).then(() => {
          // 重新拉取
          this.getAllMaterial() // 重新加载数据
        })
      })
    },
    // 收藏或者取消收藏
    collectOrCancel (row) {
      // 调用  收藏或者取消收藏接口
      this.$axios({
        url: `/user/images/${row.id}`,
        method: 'put',
        data: {
          collect: !row.is_collected // 状态取反
        }
      }).then(() => {
        // 成功一定进入then
        this.getAllMaterial()
      })
    },
    uploadImg (params) {
      this.loading = true // 打开进度条
      let form = new FormData()
      form.append('image', params.file) // 添加文件到formData
      this.$axios({
        method: 'post',
        url: '/user/images',
        data: form
      }).then(result => {
        this.loading = false // 关闭进度条
        this.getAllMaterial()
      })
    },
    // 切换分页
    changePage (newPage) {
      this.page.currentPage = newPage // 得到最新页码
      this.getAllMaterial()
    },
    // 切换tab事件
    changeTab () {
      // 切换tab 应该把当前页码回到第一页 因为如果不重置第一页，就会找不到对应页码
      this.page.currentPage = 1
      this.getAllMaterial()
    },
    // 获取所有素材
    getAllMaterial () {
      this.$axios({
        url: '/user/images',
        params: {
          collect: this.activeName === 'collect',
          page: this.page.currentPage,
          per_page: this.page.pageSize
        }
      }).then(result => {
        this.list = result.data.results
        this.page.total = result.data.total_count // 获取总条数
      })
    }
  },
  created () {
    // 获取所有的素材
    this.getAllMaterial() // 第一次进入activeName 为all实际上是查的全部的数据
  }
}
</script>

<style lang="less" scoped>
.img-list {
  display: flex;
  flex-wrap: wrap;
  .img-card {
    width: 200px;
    height: 250px;
    margin: 20px 50px;
    position: relative;
  }
  img {
    height: 100%;
    width: 100%;
  }
  .operate {
    width: 100%;
    font-size: 20px;
    position: absolute;
    bottom: 0;
    background-color: #f4f5f6;
    height: 30px;
    margin-left: -20px;
    i {
      cursor: pointer; //换成小手状态
    }
  }
}
</style>
