<template>
    <div>
      <!--卡片组件-->
      <el-card class="card_box">
        <!--利用布局组件调试每一栏的间隔-->
        <el-row :gutter="20">
          <el-col :span="8">
            <!--input表单-->
            <el-input placeholder="请输入内容" v-model="queryInfo.query" @clear="getGoodsList" class="input-with-select" clearable>
              <el-button slot="append" @click="getGoodsList" icon="el-icon-search"></el-button>
            </el-input>
          </el-col>
          <el-col :span="4">
            <!--button按钮-->
            <el-button type="primary">添加商品</el-button>
          </el-col>
        </el-row>
        <!--表格组件-->
        <el-table
          :data="GoodsList.goods"
          border
          stripe
          style="width: 100%">
          <el-table-column
            type="index"
            label="#">
          </el-table-column>
          <el-table-column
            prop="goods_name"
            label="商品名称">
          </el-table-column>
          <el-table-column
            prop="goods_price"
            label="商品价格（元）">
          </el-table-column>
          <el-table-column
            prop="goods_weight"
            label="商品重量">
          </el-table-column>
          <el-table-column
            prop="add_time"
            label="创建时间">
            <template #default="{ row }">
              {{ new Date(row.add_time).toLocaleString() }}
            </template>
          </el-table-column>
          <el-table-column
            prop="address"
            label="操作">
            <template #default="{ row }">
              <el-button @click="showEditDialog(row.goods_id)" type="primary" icon="el-icon-edit" size="mini"></el-button>
              <el-button @click="removeGoodsBgyId(row.goods_id)" type="danger" icon="el-icon-delete" size="mini"></el-button>
            </template>
          </el-table-column>
        </el-table>
        <!--分页组件-->
        <el-pagination
          @size-change="handleSizeChangeGoods"
          @current-change="handleCurrentChangeGoods"
          :current-page="queryInfo.pagenum"
          :page-sizes="[5, 10, 15, 20, 25, 30]"
          :page-size="queryInfo.pagesize"
          layout="total, sizes, prev, pager, next, jumper"
          :total="totalGoods">
        </el-pagination>
      </el-card>
      <!--修改用户的对话框-->
      <el-dialog
        title="修改商品"
        :visible.sync="editGoodsDialogVisible"
        width="50%"
        @close="editGoodsDialogClosed">
        <!--表单结构-->
        <el-form :model="editGoodsForm" :rules="editGoodsFormRules" ref="editGoodsRef" label-width="100px" hide-required-asterisk>
          <el-form-item label="商品名称" prop="goods_name">
            <el-input v-model="editGoodsForm.goods_name"></el-input>
          </el-form-item>
          <el-form-item label="商品价格" prop="goods_price">
            <el-input v-model="editGoodsForm.goods_price"></el-input>
          </el-form-item>
          <el-form-item label="商品数量" prop="goods_number">
            <el-input v-model="editGoodsForm.goods_number"></el-input>
          </el-form-item>
          <el-form-item label="商品重量" prop="goods_weight">
            <el-input v-model="editGoodsForm.goods_weight"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="editGoodsDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editGoodsInfo">确 定</el-button>
        </span>
      </el-dialog>
    </div>
</template>

<script>
    export default {
        name: 'Goods',
      data () {
        const checkSZ = (rule, value, callback) => {
          // 设置数字的正则表达式
          const regId = /^\d+$|^\d+[.]?\d+$/
          if (regId.test(value)) {
            // 说明是正确的数字
            return callback()
          }
          callback(new Error('请输入正确的数字'))
        }
        return {
          // 修改表单验证对象
          editGoodsFormRules: {
            goods_name:[
              { required: true, message: '请输入商品名称', trigger: 'blur' },
              { min: 5, max: 20, message: '长度在 2 到 20 个字符', trigger: 'blur' }
            ],
            goods_price:[
              { required: true, message: '请输入商品价格', trigger: 'blur' },
              { validator: checkSZ, trigger: 'blur' }
            ],
            goods_number:[
              { required: true, message: '请输入商品名称', trigger: 'blur' },
              { validator: checkSZ, trigger: 'blur' }
            ],
            goods_weight:[
              { required: true, message: '请输入商品名称', trigger: 'blur' },
              { validator: checkSZ, trigger: 'blur' }
            ]
          },
          // 修改表单数据对象
          editGoodsForm: {},
          // 控制修改用户的弹框的显示和隐藏
          editGoodsDialogVisible: false,
          GoodsList: [],
          totalGoods: 0,
          // 分类的参数对象
          queryInfo: {
            query:'', // 查询字符串
            type: '',
            pagenum: 1,
            pagesize: 10
          }
        }
      },
      created () {
        this.getGoodsList()
      },
      methods: {
        // 点击修改  有问题
        editGoodsInfo () {
          this.$refs.editGoodsRef.validate(async valid => {
            if (!valid) return
            // 发送请求
            const { data: res } = await this.$http.put(`goods/${this.editGoodsForm.goods_id}`, {
              goods_id: this.editGoodsForm.goods_id,
              goods_name: this.editGoodsForm.goods_name,
              goods_price: this.editGoodsForm.goods_price,
              goods_number: this.editGoodsForm.goods_number,
              goods_weight: this.editGoodsForm.goods_weight
            })
            if (res.meta.status !== 201) return this.$message.error(res.meta.msg)
            // 成功
            // 重新获取用户列表数据数据
            this.getGoodsList()
            // 关闭修改用户框
            this.editGoodsDialogVisible = false
            this.$message.success('修改成功')
          })
        },
        // 监听修改对话框关闭的事件
        editGoodsDialogClosed () {
          this.$refs.editGoodsRef.resetFields()
        },
        // 点击显示修改的对话框
        async showEditDialog (id) {
          // 根据id查询用户信息
          const { data: res } = await this.$http.get(`goods/${id}`)
          console.log(res)
          if (res.meta.status !== 200) return this.$message.error('查询失败')
          // 成功
          this.editGoodsForm = res.data
          // 显示弹框
          this.editGoodsDialogVisible = true
        },
        // 点击删除单个商品
        async removeGoodsBgyId (id) {
          // 消息确认框的组件
          const confirmRes = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning',
            center: true
          }).catch(err => err)
          // console.log(confirmRes)
          // 如果点击的是确定按钮，confirmRes; confirm
          // 如果点击的是确定按钮，confirmRes; cancel
          if (confirmRes === 'confirm') {
            const { data: res } = await this.$http.delete(`goods/${id}`)
            if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
            // 成功
            this.getGoodsList()
          }
        },
        // 获取商品列表
        async getGoodsList() {
          const { data: res } = await this.$http.get('goods', {
            params: this.queryInfo
          })
          if (res.meta.status !== 200) return this.$message.error('获取失败')
          // 成功
          this.GoodsList = res.data
          this.totalGoods = res.data.total
        },
        // 监听pagesize改变事件
        handleSizeChangeGoods (newSize) {
          this.queryInfo.pagesize = newSize
          // 重新获取数据
          this.getGoodsList()
        },
        // 监听page改变事件
        handleCurrentChangeGoods (newPage) {
          this.queryInfo.pagenum = newPage
          // 重新获取数据
          this.getGoodsList()
        }
      }
    }
</script>

<style scoped>

</style>
