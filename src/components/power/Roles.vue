<template>
    <div style="margin-top: 20px">
      <!--卡片组件-->
      <el-card>
        <el-button type="primary" style="margin-bottom: 20px" @click="dialogVisible = true">添加角色</el-button>
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
              <el-button @click="editRolesDialog(row.id)" type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
              <el-button @click="removeRolesBgyId(row.id)" type="danger" icon="el-icon-delete" size="mini">删除</el-button>
              <el-button @click="showRightDialog(row)" type="warning" icon="el-icon-setting" size="mini">分配权限</el-button>
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
        <el-form :model="editForm" :rules="editFormRules" ref="editRef" label-width="100px" hide-required-asterisk>
          <el-form-item label="id" prop="roleId">
            <el-input v-model="editForm.roleId" disabled></el-input>
          </el-form-item>
          <el-form-item label="角色名称" prop="roleName">
            <el-input v-model="editForm.roleName"></el-input>
          </el-form-item>
          <el-form-item label="角色描述" prop="roleDesc">
            <el-input v-model="editForm.roleDesc"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="editDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editRolesInfo">确 定</el-button>
        </span>
      </el-dialog>
      <!--添加角色的对话框-->
      <el-dialog
        title="修改角色"
        :visible.sync="dialogVisible"
        width="50%"
        @close="addDialogClosed">
        <el-form :model="addForm" :rules="addFormRules" ref="addRef" label-width="100px" hide-required-asterisk>
          <el-form-item label="角色名称" prop="roleName">
            <el-input v-model="addForm.roleName"></el-input>
          </el-form-item>
          <el-form-item label="角色描述" prop="roleDesc">
            <el-input v-model="addForm.roleDesc"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="dialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addUserInfo">确 定</el-button>
        </span>
      </el-dialog>
      <!--分配权限的对话框-->
      <el-dialog
        title="分配权限"
        :visible.sync="getRightDialogVisible"
        width="50%"
        @close="rightDialogClosd">
        <!--树形控件的组件-->
        <!--default-expand-all :默认 展开所有的节点-->
        <!--default-checked-kers : 默认选中展开的节点。以key选中-->
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
    <el-button @click="getRightDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="saveRights">确 定</el-button>
  </span>
      </el-dialog>
    </div>
</template>

<script>
    export default {
        name: 'Roles',
      data () {
        const checkId = (rule, value, callback) => {
            // 设置id的正则表达式
            const regId = /^\d+$|^\d+[.]?\d+$/
            if (regId.test(value)) {
              // 说明是正确的id
              return callback()
            }
            callback(new Error('请输入正确的id'))
          }
        return {
          // 定义id的校验规则
          roleList: [],
          // 控制修改角色的弹框的显示和隐藏
          editDialogVisible: false,
          // 修改表单数据对象
          editForm: {},
          // 修改表单验证对象
          editFormRules: {
            roleId:[
              { required: true, message: '请输入id', trigger: 'blur' },
              // 调用自定义的校验规则
              { validator: checkId, trigger: 'blur' }
            ],
            roleName:[
              { required: true, message: '请输入角色名称', trigger: 'blur' },
              { min: 2, max: 10, message: '长度在 2 到 10 个字符', trigger: 'blur' }
            ],
            roleDesc:[
              { required: true, message: '请输入角色描述', trigger: 'blur' },
              { min: 2, max: 10, message: '长度在 2 到 10 个字符', trigger: 'blur' }
            ]
          },
          // 控制添加角色的弹框的显示和隐藏
          dialogVisible: false,
          // 添加表单数据对象
          addForm: {},
          // 添加表单验证对象
          addFormRules: {
            roleName:[
              { required: true, message: '请输入角色名称', trigger: 'blur' },
              { min: 2, max: 10, message: '长度在 2 到 10 个字符', trigger: 'blur' }
            ],
            roleDesc:[
              { required: true, message: '请输入角色描述', trigger: 'blur' },
              { min: 2, max: 10, message: '长度在 2 到 10 个字符', trigger: 'blur' }
            ]
          },
          // 控制分配权限的弹框的显示和隐藏
          getRightDialogVisible: false,
          // 树形控件的数据源
          rightsList: [],
          // 默认选中展开的节点。以key选中
          defaultKeys: [],
          // 指定节点的字段
          treeProps: {
            label: 'authName',
            children: 'children'
          },
          // 存储当前角色id
          roleId: ''
        }
      },
      created () {
        this.getRoleList()
      },
      methods: {
        // 获取角色列表数据
        async getRoleList() {
          const { data: res } = await this.$http.get('roles')
          if (res.meta.status !== 200) return this.$message.error('获取角色失败')
          // 成功
          this.roleList = res.data
        },
        // 点击删除对应的权限
        async removeRights (role, rightId) {
          const confirmRes = await this.$confirm('此操作将永久删除, 是否继续?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          }).catch(err => err)
          if (confirmRes !== 'confirm') return this.$message.info('取消了删除')
          // 发送请求
          const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
          if (res.meta.status !== 200) return this.$message.eror('删除权限失败')
          // 删除成功, 不建议刷新数据, 利用返回给我们的修改之后的数据赋值给data的中参数即可
          role.children = res.data
        },
        // 点击删除单个角色
        async removeRolesBgyId (id) {
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
            const { data: res } = await this.$http.delete(`roles/${id}`)
            if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
            // 成功
            this.getRoleList()
          }
        },
        // 监听修改角色对话框关闭的事件
        editDialogClosed () {
          this.$refs.editRef.resetFields()
        },
        // 点击显示修改角色的对话框
        async editRolesDialog (id) {
          // 根据id查询用户信息
          const { data: res } = await this.$http.get(`roles/${id}`)
          if (res.meta.status !== 200) return this.$message.error('查询失败')
          // 成功
          this.editForm = res.data
          // 显示弹框
          this.editDialogVisible = true
        },
        // 点击修改角色
        editRolesInfo () {
          this.$refs.editRef.validate(async valid => {
            if (!valid) return
            // 发送请求
            const { data: res } = await this.$http.put(`roles/${this.editForm.roleId}`, {
              roleId: this.editForm.roleId,
              roleName: this.editForm.roleName,
              roleDesc: this.editForm.roleDesc
            })
            if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
            // 成功
            // 重新获取用户列表数据数据
            this.getRoleList()
            // 关闭修改用户框
            this.editDialogVisible = false
            this.$message.success('修改成功')
          })
        },
        // 监听添加角色对话框关闭的事件
        addDialogClosed () {
          this.$refs.addRef.resetFields()
        },
        // 点击按钮新增角色信息
        addUserInfo() {
          // 先验证所有的验证信息
          this.$refs.addRef.validate(async valid => {
            if (!valid) return
            // 发送请求
            const { data: res } = await this.$http.post('roles', this.addForm)
            if (res.meta.status !== 201) return this.$message.error(res.meta.msg)
            // 成功
            // 重新获取用户列表数据数据
            this.getRoleList()
            // 关闭添加用户框
            this.dialogVisible = false
          })
        },
        // 点击显示分配权限的对话框
        async showRightDialog (role) {
          // 保存当前角色id
          this.roleId = role.id
          // 请求数据
          const { data: res } = await this.$http.get('rights/tree')
          if (res.meta.status !== 200) return this.$message.error('获取失败')
          // 成功
          this.rightsList = res.data
          // 调用,@parame: a.对应的权限列表， b. 数组
          this.getrights3Keys(role, this.defaultKeys)
          // 显示弹框
          this.getRightDialogVisible = true
        },
        // 定义一个递归方法，用来获取所有权限3的id, 追加到defaultKeys
        getrights3Keys (node, arr) {
          // 判断是不是3级节点,根据有没有children字段即可
          if (!node.children) return arr.push(node.id)
          // 不是3级节点，再遍历的去调用自已即可
          node.children.forEach(item => {
            this.getrights3Keys(item, arr)
          })
        },
        // 监听对话框关闭事件
        rightDialogClosd () {
          this.defaultKeys = []
        },
        // 点击修改分配权限
        async saveRights () {
          // 获取一二所有的权限id
          const keys = [
            ...this.$refs.treeRef.getCheckedKeys(),
            ...this.$refs.treeRef.getHalfCheckedKeys()
          ].join(',')
          // 发送请求
          const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, {
            rids: keys
          })
          if (res.meta.status !== 200) return this.$message.error('分配权限失败')
          // 成功
          this.getRoleList()
          this.getRightDialogVisible = false
        }
      }
    }
</script>

<style scoped lang="less">
  .el-tag{
    margin: 8px;
  }
  .bdBottom{
    border-bottom: 1px solid #ccc;
  }
  .bdTop{
    border-top: 1px solid #eee;
  }
  .itemsCenter{
    display: flex;
    align-items: center;
  }
</style>
