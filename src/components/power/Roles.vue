<template>
    <div>
      <!--卡片组件-->
      <el-card>
        <el-button type="primary">添加角色</el-button>
        <!--角色表格区域-->
        <el-table :data="roleList" border stripe>
          <el-table-column type="expand">
            <!--渲染对应的权限列表-->
            <template #default="{ row }">
              <el-row v-for="(item1, i1) in row.children" :key="item1.id" :class="['bdBottom','itemsCenter',i1==0?'bdTop':'']">
                <el-col :span="5">
                  <!--一级权限-->
                  <el-tag closable @close="removeRights(row, item1.id)">{{ item1.authName }}</el-tag>
                  <i class="el-icon-caret-right"></i>
                </el-col>
                <el-col :span="19">
                  <!--通过for循环嵌套的渲染二级权限-->
                  <el-row v-for="(item2, i2) in item1.children" :key="item2.id" :class="['itemsCenter',i2==0?'':'bdTop' ]">
                    <!--二级权限-->
                    <el-col :span="6">
                      <el-tag closable @close="removeRights(row, item2.id)"  type="success">{{ item2.authName }}</el-tag>
                      <i class="el-icon-caret-right"></i>
                    </el-col>
                    <!--三级权限-->
                    <el-col :span="18">
                      <el-tag closable @close="removeRights(row, item3.id)"  v-for="item3 in item2.children" type="warning" :key="item3.id">{{ item3.authName }}</el-tag>
                    </el-col>
                  </el-row>
                </el-col>
              </el-row>
            </template>
          </el-table-column>
          <el-table-column type="index" label="#"></el-table-column>
          <el-table-column prop="roleName" label="角色名称"></el-table-column>
          <el-table-column prop="roleDesc" label="角色描述"></el-table-column>
          <el-table-column label="操作">
            <template #default="{ row }">
              <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(row.id)">编辑</el-button>
              <el-button @click="removeRoleById(row.id)" type="danger" icon="el-icon-delete" size="mini" >删除</el-button>
              <el-button type="warning" icon="el-icon-setting" size="mini" @click="showRightDialog(row)">分配权限</el-button>
            </template>
          </el-table-column>

        </el-table>
      </el-card>

      <!--修改角色的对话框-->
      <el-dialog
        title="修改角色"
        :visible.sync="editDialogVisible"
        width="50%"
        @close="editDialogClosed">
        <!--表单结构-->
        <el-form :model="editForm"  :rules="editFormRules" ref="editRef" label-width="70px" hide-required-asterisk>
          <!--prop="roleName" : 验证规则的属性-->
          <el-form-item label="角色名称" prop="roleName">
            <el-input v-model="editForm.roleName"></el-input>
          </el-form-item>
          <el-form-item label="角色描述" prop="roleDesc">
            <el-input v-model="editForm.roleDesc"></el-input>
          </el-form-item>
        </el-form>

        <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary"  @click="editUserInfo()">确 定</el-button>
        </span>
      </el-dialog>
      <!--分配权限的对话框-->
      <el-dialog
        title="分配权限"
        :visible.sync="setRightDialogVisible"
        width="50%"
        @close="rightDialogClosed">
        <!--树形控件的组件-->
        <!--default-expand-all : 默认展开所有节点-->
        <!--default-checked-keys: 默认选中展开的节点的 以key选中-->
        <el-tree
          ref="treeRef"
          :data="rightsList"
          show-checkbox
          node-key="id"
          default-expand-all
          :default-checked-keys="defaultKeys"
          :props="treeProps">
        </el-tree>
        <span slot="footer" class="dialog-footer">
    <el-button @click="setRightDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="saveRights">确 定</el-button>
  </span>
      </el-dialog>
    </div>
</template>

<script>
    export default {
        name: 'Roles',
      data() {
          return {
            // 控制分配权限对话框的显示和隐藏
            setRightDialogVisible: false,
            // 树形控件的数据源
            // 经过测试，只需要获取权限3的所有id,并放到数组里即
            rightsList: [],
            // 默认选中展开的节点的 以key选中
            defaultKeys: [],
            // 指定节点的字段
            treeProps: {
              label: 'authName',
              children: 'children'
            },
            // 存储角色id
            roleId: '',
            // 控制修改弹框的显示和隐藏
            editDialogVisible: false,
            roleList:[],
            // 修改表单数据对象
            editForm:{},
            // 修改表单的效验规则
            editFormRules: {
              roleName:[
                { required: true, message: '角色名称', trigger: 'blur' }
              ],
              roleDesc:[
                { required: true, message: '请输入手机号', trigger: 'blur' }
              ]
            }
          }
      },
      created() {
          this.getRoleList()
      },
      methods:{
          // 点击显示分配权限对话框
        async showRightDialog(role) {
          // 保存当前角色id
          this.roleId = role.id
          // 请求数据
          const { data: res } = await this.$http.get('rights/tree')
          if (res.meta.status !== 200) return this.$message.error('获取失败')
          // 成功
          this.rightsList = res.data
          // 调用，@params: a.对应的权限列表 b.数组
          this.getrights3Keys(role, this.defaultKeys)
          this.setRightDialogVisible = true
        },
        // 定义一个递归方法，用来获取所有权限3的id, 追加到defaultKeys
        getrights3Keys(node, arr) {
          // 判断是不是三级节点,根据有没有children字段即可
          if (!node.children) return arr.push(node.id)
          // 不是三级节点，再遍历的去调用自己即可
          node.children.forEach(item => {
            this.getrights3Keys(item, arr)
          })
        },
        // 监听分配权限对话框关闭事件
        rightDialogClosed() {
          this.defaultKeys = []
        },
        // 点击提交分配权限
        async saveRights() {
          // 获取一二三级所有权限id,利用扩展运算符合并数组
          const keys = [
            ...this.$refs.treeRef.getCheckedKeys(),
            ...this.$refs.treeRef.getHalfCheckedKeys()
          ].join(',')
          // 发请求
          const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, {
            rids: keys
          })
          if (res.meta.status !== 200) return this.$message.error('分配权限失败')
          // 成功
          this.getRoleList()
          this.setRightDialogVisible = false
        },
        // 获取角色列表数据
        async getRoleList() {
          const { data: res } = await this.$http.get('roles')
          if (res.meta.status !== 200) return this.$message.error('获取角色失败')
          // 成功
          this.roleList = res.data
        },
        // 点击删除对应的权限
        async removeRights(role, rightId) {
          const confirmRes = await this.$confirm('此操作将永久删除, 是否继续?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          }).catch(err => err)

          if (confirmRes !== 'confirm') return this.$message.info('取消了删除')
          // 发请求
          const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
          if (res.meta.status !== 200) return this.$message.eror('删除权限失败')
          // 删除成功, 不建议刷新数据, 利用返回给我们的修改之后的数据赋值给data的中参数即可
          console.log(res)
          console.log(role)
          role.children = res.data
        },
        // 监听修改对话框关闭的事件
        editDialogClosed() {
          this.$refs.editRef.resetFields()
        },
        // 点击删除角色
        async removeRoleById(id) {
          // 消息确认框的组件
          const confirmRes = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          }).catch(err => err)
          if (confirmRes === 'confirm') {
            const { data: res } = await this.$http.delete(`roles/${id}`)
            if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
            // 成功 , 刷新数据
            this.getRoleList()
          }
        },
        // 点击显示修改对话框
        async showEditDialog(id) {
          // 根据id查询用户信息
          const { data :res } = await this.$http.get(`roles/${id}`)
          if (res.meta.status !== 200) return this.$message.error('查询失败')
          // 成功
          this.editForm = res.data
          // 显示弹框
          this.editDialogVisible = true
        },
        // 点击编辑角色
        editUserInfo() {
          this.$refs.editRef.validate(async valid => {
            if (!valid) return
            // 发请求
            const { data: res } = await this.$http.put(`roles/${this.editForm.roleId}`, {
              roleName: this.editForm.roleName,
              roleDesc: this.editForm.roleDesc
            })
            if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
            // 成功
            // 重新获取用户列表数据
            this.getRoleList()
            // 关闭对话框
            this.editDialogVisible = false
            this.$message.success('更新成功')
          })
        }
      }
    }
</script>

<style scoped>
  .el-tag{
    margin: 8px;
  }
  .bdBottom{
    border-bottom: 1px solid #eee;
  }
  .bdTop{
    border-top: 1px solid #eee;
  }
  .itemsCenter{
    display: flex;
    align-items: center;
  }
</style>
