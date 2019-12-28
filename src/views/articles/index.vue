<template>
  <el-card class="articles">
    <bread-crumb slot="header">
      <template slot="title">文章列表</template>
    </bread-crumb>
    <!-- el-row 行 -->
    <el-row class="searchTool">
      <el-col :span="2">
        <span>文章状态</span>
      </el-col>
      <el-col :span="18">
        <!--
              0-草稿，1-待审核，2-审核通过，3-审核失败，4-已删除，不传为全部
        -->
        <!-- 单选框组 -->
        <!-- 第一种方式用  @change="changeCondition" 第二种方式 用watch-->
        <el-radio-group @change="changeCondition" v-model="formData.status">
          <!-- 全部这个5是默认的,在传参的时候判断一下 是不是5 如果是5 就传个null -->
          <el-radio :label="5">全部</el-radio>
          <el-radio :label="0">草稿</el-radio>
          <el-radio :label="1">待审核</el-radio>
          <el-radio :label="2">审核通过</el-radio>
          <el-radio :label="3">审核失败</el-radio>
        </el-radio-group>
      </el-col>
    </el-row>
    <el-row class="searchTool">
      <el-col :span="2">
        <span>频道列表</span>
      </el-col>
      <el-col :span="18">
        <el-select @change="changeCondition" v-model="formData.channel_id">
          <!-- 循环生成多个el-option -->
          <!-- label 指的是el-option的显示值
          value 指的是el-option 的存储值-->
          <el-option v-for="item in channels" :key="item.id" :label="item.name" :value="item.id"></el-option>
        </el-select>
      </el-col>
    </el-row>
    <el-row class="searchTool">
      <el-col :span="2">
        <span>时间选择</span>
      </el-col>
      <el-col :span="18">
        <el-date-picker
          @change="changeCondition"
          value-format="yyyy-MM-dd"
          v-model="formData.dataRange"
          type="daterange"
          range-separator="-"
          start-placeholder="开始日期"
          end-placeholder="结束日期"
        ></el-date-picker>
      </el-col>
    </el-row>
    <el-row class="total">
      <!-- 主体 -->
      <span>共找到{{ page.total }} 条符合条件的内容</span>
    </el-row>
    <!-- 循环模板 -->
    <el-row
      v-for="item in list"
      :key="item.id.toString()"
      class="article-item"
      type="flex"
      justify="space-between"
    >
      <!-- 左 -->
      <el-col :span="10">
        <el-row type="flex">
          <img :src="item.cover.images.length ? item.cover.images[0]: defaultImg" alt />
          <div class="tianwang">
            <span>{{ item.title }}</span>
            <!-- 过滤器不但可以在插值表达式中使用 还可以在v-bind表达式中使用 -->
            <el-tag :type="item.status | filterType" class="tag">{{ item.status | filterStatus}}</el-tag>
            <span class="date">{{ item.pubdate }}</span>
          </div>
        </el-row>
      </el-col>
      <!-- 右 -->
      <el-col :span="6">
        <el-row class="right" type="flex" justify="end">
          <span>
            <i class="el-icon-edit"></i>修改
          </span>
          <span @click="delArticle(item.id)">
            <i class="el-icon-delete"></i>删除
          </span>
        </el-row>
      </el-col>
    </el-row>
    <!-- 分页 -->
    <el-row type="flex" justify="center" align="middle" style="height:60px">
      <el-pagination
        background
        layout="prev, pager, next"
        :total="page.total"
        :current-page="page.currentPage"
        :page-size="page.pageSize"
        @current-change="changePage"
      ></el-pagination>
    </el-row>
  </el-card>
</template>
<script>
export default {
  data () {
    return {
      formData: {
        status: 5, // 状态
        channel_id: null, // 默认是空
        dataRange: [] // 默认为空数组
      },
      channels: [], // 定义一个channels 接受频道
      list: [], // 接收文章列表数据
      defaultImg: require('../../assets/img/2.jpg'),
      page: {
        currentPage: 1, // 当前页码为1条
        pageSize: 10, // 最低10条
        total: 0
      }
    }
  },
  filters: {
    // 处理显示状态
    filterStatus (value) {
      // value 是过滤器前面表达式计算得到的值
      switch (value) {
        case 0:
          return '草稿'
        case 1:
          return '待审核'
        case 2:
          return '已发表'
        case 3:
          return '审核失败'
        default:
          break
      }
    },
    filterType (value) {
      // value 是过滤器前面表达式计算得到的值
      switch (value) {
        case 0:
          return 'warning'
        case 1:
          return 'info'
        case 2:
          return ''
        case 3:
          return 'danger'
        default:
          break
      }
    }
  },
  methods: {
    // 删除文章
    delArticle (id) {
      this.$confirm('是否删除？').then(() => {
        this.$axios({
          method: 'delete',
          url: `/articles/${id.toString()}`
        }).then(() => {
          this.$message({
            type: 'success',
            message: '删除成功'
          })
          alert(123)
          this.page.currentPage = 1 // 如果想回第一页就赋值唯一
          this.getConditionArticle()
        })
      })
    },
    changePage (newPage) {
      // 赋值当前页码
      this.page.currentPage = newPage
      this.getConditionArticle()
    },
    // 改变条件
    changeCondition () {
      // 组装条件
      this.page.currentPage = 1 // 强制将页码回到第一页
      this.getConditionArticle()
    },
    getConditionArticle () {
      // alert(1)
      let params = {
        page: this.page.currentPage, // 分页信息
        per_page: this.page.pageSize, // 分页信息
        status: this.formData.status === 5 ? null : this.formData.status, // 不传为全部，5代表全部
        channels_id: this.formData.channel_id, // 频道
        begin_pubdate: this.formData.dataRange.length
          ? this.formData.dataRange[0]
          : null, // 起始时间
        end_pubdate:
          this.formData.dataRange.length > 1 ? this.formData.dataRange[1] : null // 截止时间
      }
      this.getArticles(params)
    },
    // 获取频道
    getChannels () {
      this.$axios({
        url: 'channels'
      }).then(result => {
        this.channels = result.data.channels // 获取频道数据
      })
    },
    // 获取文章列表数据
    getArticles (params) {
      this.$axios({
        url: '/articles', // 请求地址
        params
      }).then(result => {
        this.list = result.data.results // 接收文章列表数据
        this.page.total = result.data.total_count // 文章总数
      })
    }
  },
  created () {
    this.getChannels() // 调用获取频道数据
    this.getArticles({ page: 1, per_page: 10 }) // 调用读取文章列表
  }
}
</script>

<style lang="less" scoped>
.articles {
  .searchTool {
    height: 60px;
    padding-left: 50px;
    margin: 0 auto;
  }
  .total {
    margin: 60px 0px;
    height: 35px;
    border-bottom: 1px dashed #ccc;
  }
  .article-item {
    margin: 20px 0px;
    padding: 1ex;
    border-bottom: 1px solid #f2f3f5;
    img {
      width: 180px;
      height: 100px;
      margin-right: 10px;
      border-radius: 5px;
    }
    .tianwang {
      height: 100px;
      display: flex;
      flex-direction: column;
      justify-content: space-between;
      .tag {
        max-width: 58px;
      }
      .data {
        color: #999;
        font-size: 12px;
      }
    }
    .right {
      span {
        margin-left: 8px;
        font-size: 12px;
        cursor: pointer;
      }
    }
  }
}
</style>
