<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
      <el-breadcrumb-item>添加商品</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-alert title="添加商品信息" type="info" center show-icon :closable="false"></el-alert>
      <!-- 步骤 -->
      <el-steps :active="+activeIndex" process-status="finish" finish-status="success" align-center>
        <el-step title="基本信息"></el-step>
        <el-step title="商品参数"></el-step>
        <el-step title="商品属性"></el-step>
        <el-step title="商品照片"></el-step>
        <el-step title="商品内容"></el-step>
        <el-step title="完成"></el-step>
      </el-steps>
      <el-form
        ref="addFormRef"
        :model="addForm"
        label-width="80px"
        :rules="addRules"
        label-position="top"
      >
        <el-tabs
          v-model="activeIndex"
          :tab-position="'left'"
          :before-leave="tabLeave"
          @tab-click="tabClicked"
        >
          <el-tab-pane label="基本信息" name="0">
            <el-form-item label="商品名称" prop="goods_name">
              <el-input v-model="addForm.goods_name"></el-input>
            </el-form-item>
            <el-form-item label="商品价格" prop="goods_price">
              <el-input v-model="addForm.goods_price"></el-input>
            </el-form-item>
            <el-form-item label="商品重量" prop="goods_weight">
              <el-input v-model="addForm.goods_weight"></el-input>
            </el-form-item>
            <el-form-item label="商品数量" prop="goods_number">
              <el-input v-model="addForm.goods_number"></el-input>
            </el-form-item>
            <el-form-item label="商品分类" prop="goods_cat">
              <el-cascader
                :options="cateList"
                :props="cateProps"
                v-model="addForm.goods_cat"
                @change="handleChange"
              ></el-cascader>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品参数" name="1">
            <el-form-item :label="item.attr_name" v-for="item in manyTableData" :key="item.attr_id" size="mini">
              <el-checkbox-group v-model="item.attr_vals" size="mini">
                <el-checkbox id="el-checkbox" :label="it"  v-for="(it, i) in item.attr_vals" :key="i" border></el-checkbox>
              </el-checkbox-group>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品属性" name="2"><el-form-item :label="item.attr_name" v-for="item in onlyTableData" :key="item.attr_id" size="mini">
              <el-input v-model="item.attr_vals"></el-input>
            </el-form-item></el-tab-pane>
          <el-tab-pane label="商品图片" name="3">
            <el-upload
              :action="uploadUrl"
              :on-preview="handlePreview"
              :on-remove="handleRemove"
              list-type="picture"
              :headers="uploadHeader"
              :on-success="handleSuccess">
              <el-button size="small" type="primary">点击上传</el-button>
            </el-upload>
          </el-tab-pane>
          <el-tab-pane label="商品内容" name="4"><quill-editor v-model="addForm.goods_instroduce"></quill-editor>
          <el-button id="addGoods" type="primary" @click="addGoods">添加商品</el-button></el-tab-pane>
        </el-tabs>
      </el-form>
    </el-card>
    <!-- 对话框 -->
    <el-dialog
      title="图片预览"
      :visible.sync="previewVisible"
      width="50%">
      <img :src="previewPath">
    </el-dialog>
  </div>
</template>

<script>
import _ from 'lodash'

export default {
  data () {
    return {
      activeIndex: '0',
      addForm: {
        goods_name: '',
        goods_price: 0,
        goods_weight: 0,
        goods_number: 0,
        goods_cat: [],
        pics: [],
        goods_instroduce: '',
        attrs: []
      },
      addRules: {
        goods_name: [
          { required: true, message: '请输入商品名称', trigger: 'blur' }
        ],
        goods_price: [
          { required: true, message: '请输入商品价格', trigger: 'blur' }
        ],
        goods_weight: [
          { required: true, message: '请输入商品重量', trigger: 'blur' }
        ],
        goods_number: [
          { required: true, message: '请输入商品数量', trigger: 'blur' }
        ],
        goods_cat: [
          { required: true, message: '请选择商品分类', trigger: 'blur' }
        ]
      },
      cateList: [],
      cateProps: {
        label: 'cat_name',
        value: 'cat_id',
        children: 'children'
      },
      manyTableData: [],
      onlyTableData: [],
      uploadUrl: 'http://192.168.31.195:8888/api/private/v1/upload',
      uploadHeader: { Authorization: sessionStorage.getItem('token') },
      previewPath: '',
      previewVisible: false
    }
  },
  created () {
    this.getCatelist()
  },
  methods: {
    async getCatelist () {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('获取分类列表失败')
      }
      this.cateList = res.data
    },
    handleChange () {
      if (this.addForm.goods_cat.length !== 3) {
        this.addForm.goods_cat = []
      }
    },
    tabLeave (activeName, oldActiveName) {
      let isLeave = true
      this.$refs.addFormRef.validate(valid => {
        if (!valid) {
          this.$message.error('请填写完整的基本信息')
          isLeave = false
        }
      })
      return isLeave
    },
    async tabClicked () {
      if (this.activeIndex === '1') {
        const {
          data: res
        } = await this.$http.get(
          `categories/${this.addForm.goods_cat[2]}/attributes`,
          { params: { sel: 'many' } }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('获取动态参数列表失败')
        }
        res.data.forEach(item => {
          item.attr_vals = item.attr_vals.length === 0 ? [] : item.attr_vals.split(' ')
        })
        this.manyTableData = res.data
      } else if (this.activeIndex === '2') {
        const {
          data: res
        } = await this.$http.get(
          `categories/${this.addForm.goods_cat[2]}/attributes`,
          { params: { sel: 'only' } }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('获取静态属性列表失败')
        }
        res.data.forEach(item => {
          item.attr_vals = item.attr_vals.length === 0 ? [] : item.attr_vals.split(' ')
        })
        this.onlyTableData = res.data
      }
    },
    handlePreview (file) {
      this.previewPath = file.response.data.url
      this.previewVisible = true
    },
    handleRemove (file) {
      const filePath = file.response.data.tmp_path
      const i = this.addForm.pics.findIndex(x => x.pic === filePath)
      this.addForm.pics.splice(i, 1)
    },
    handleSuccess (response) {
      const picInfo = { pic: response.data.tmp_path }
      this.addForm.pics.push(picInfo)
    },
    async addGoods () {
      const form = _.cloneDeep(this.addForm)
      form.goods_cat = this.addForm.goods_cat.join(',')

      this.manyTableData.forEach(item => {
        const newInfo = { attr_id: item.attr_id, attr_value: item.attr_vals.join(' ') }
        this.addForm.attrs.push(newInfo)
      })

      this.onlyTableData.forEach(item => {
        const newInfo = { attr_id: item.attr_id, attr_value: item.attr_vals.join(' ') }
        this.addForm.attrs.push(newInfo)
      })
      form.attrs = this.addForm.attrs
      const { data: res } = await this.$http.post('goods', form)
      if (res.meta.status !== 201) {
        return this.$message.error('添加商品失败')
      }
      this.$message.success('添加商品成功')
      this.$router.push('/goods')
    }
  }
}
</script>

<style lang="less" scoped>
.el-steps {
  margin: 15px 0;
}

.el-checkbox-group{
  margin-left: 5px;
}

//此处为id选择器
#el-checkbox{
  margin:0 8px
}

img{
  width: 50%;
}

#addGoods{
  margin-top:15px;
}

</style>
