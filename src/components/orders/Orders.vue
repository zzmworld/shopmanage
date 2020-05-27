<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片 -->
    <el-card>
      <el-row>
        <el-col :span="10">
          <el-input
            v-model="queryInfo.query"
            placeholder="请输入内容"
            clearable
            @clear="getOrderList"
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="getOrderList"
            ></el-button>
          </el-input>
        </el-col>
      </el-row>
      <el-table :data="orderList" border stripe>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column prop="order_id" label="订单编号"></el-table-column>
        <el-table-column prop="order_price" label="订单价格"></el-table-column>
        <el-table-column prop="pay_status" label="是否付款">
          <template slot-scope="scope">
            <el-tag type="success" v-if="scope.row.pay_status === '1'"
              >已付款</el-tag
            >
            <el-tag type="danger" v-else>未付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="is_send" label="是否发货">
          <template slot-scope="scope">
            <el-tag type="success" v-if="scope.row.is_send === '是'"
              >已发货</el-tag
            >
            <el-tag type="danger" v-else>未发货</el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="create_time" label="下单时间" min-width="100px">
          <template slot-scope="scope">{{
            (scope.row.create_time * 1000) | dateFormat
          }}</template>
        </el-table-column>
        <el-table-column label="操作" min-width="225px">
          <template slot-scope="scope">
            <el-button
              type="primary"
              size="mini"
              icon="el-icon-edit"
              @click="showAddress(scope.row.order_id)"
              >修改地址</el-button
            >
            <el-button
              type="success"
              size="mini"
              icon="el-icon-location"
              @click="showLocation(scope.row.order_id)"
              >物流信息</el-button
            >
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
    </el-card>
    <!-- 对话框 -->
    <el-dialog
      title="修改地址"
      width="50%"
      :visible.sync="addressVisible"
      @close="addressClosed"
    >
      <el-form
        ref="addressFormRef"
        :model="addressForm"
        label-width="100px"
        :rules="orderRules"
      >
        <el-form-item label="省/市/区/县" prop="address1">
          <el-cascader
            :options="cityData"
            v-model="addressForm.address1"
          ></el-cascader>
        </el-form-item>
        <el-form-item label="详细地址" prop="address2">
          <el-input v-model="addressForm.address2"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer">
        <el-button @click="addressVisible = false">取 消</el-button>
        <el-button type="primary" @click="addressVisible = false"
          >确 定</el-button
        >
      </span>
    </el-dialog>
    <!-- 物流 -->
    <el-dialog
      title="物流信息"
      width="50%"
      :visible.sync="locationVisible"
    >
      <el-timeline>
        <el-timeline-item
          v-for="(activity, index) in locationInfo"
          :key="index"
          :timestamp="activity.time"
        >
          {{ activity.context }}
        </el-timeline-item>
      </el-timeline>
    </el-dialog>
  </div>
</template>

<script>
import cityData from './citydata.js'

export default {
  data () {
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 5
      },
      total: 0,
      orderList: [],
      addressVisible: false,
      locationVisible: false,
      locationInfo: [],
      addressForm: {
        address1: [],
        address2: ''
      },
      locationForm: {},
      orderRules: {
        address1: [
          { required: true, message: '请选择省/市/区/县', trigger: 'blur' }
        ],
        address2: [
          { required: true, message: '请填写详细地址', trigger: 'blur' }
        ]
      },
      cityData
    }
  },
  created () {
    this.getOrderList()
  },
  methods: {
    async getOrderList () {
      const { data: res } = await this.$http.get('orders', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取订单列表失败')
      }
      this.total = res.data.total
      this.orderList = res.data.goods
    },
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getOrderList()
    },
    handleCurrentChange (newNum) {
      this.queryInfo.pagenum = newNum
      this.getOrderList()
    },
    showAddress () {
      this.addressVisible = true
    },
    addressClosed () {
      this.$refs.addressFormRef.resetFields()
    },
    async showLocation () {
      const { data: res } = await this.$http.get('/kuaidi/1106975712662')
      if (res.meta.status !== 200) {
        return this.$message.error('获取物流信息失败')
      }
      this.locationInfo = res.data
      this.locationVisible = true
    }
  }
}
</script>

<style lang="less" scoped></style>
