<template>
  <div class="wrapper">
    <div class="search clearfix">
      <el-button class="fr" type="primary" @click="handleAddRole">添加角色</el-button>
    </div>
    <div class="table">
      <el-table
        :data="list"
        row-key="id"
        border
        fit
        highlight-current-row
        style="width: 100%">
        <el-table-column align="center" label="序号" width="80" type="index">
        </el-table-column>
        <el-table-column align="center" label="角色名称" width="200">
          <template slot-scope="scope">
            <span>{{scope.row.roleName}}</span>
          </template>
        </el-table-column>
        <el-table-column align="center" label="角色描述" width="200">
          <template slot-scope="scope">
            <span>{{scope.row.describes}}</span>
          </template>
        </el-table-column>
        <el-table-column align="center" label="创建人" width="120">
          <template slot-scope="scope">
            <span>{{scope.row.createUser}}</span>
          </template>
        </el-table-column>
        <el-table-column align="center" label="状态" width="80">
          <template slot-scope="scope">
            <span>{{scope.row.enabled === '1' ? '启用' : '禁用'}}</span>
          </template>
        </el-table-column>
        <el-table-column align="center" label="创建时间" :formatter="getCreateTime">
        </el-table-column>
        <el-table-column align="center" label="修改时间" :formatter="getUpdateTime">
        </el-table-column>
        <el-table-column align="center" label="操作">
          <template slot-scope="scope">
            <div>
              <el-button
                size="mini"
                type="primary"
                @click="handleCheck(scope.row.roleId)"
              >编辑</el-button>
              <el-button
                size="mini"
                type="warning"
                @click="handleAllot(scope.row)"
              >分配权限</el-button>
              <el-button
                size="mini"
                type="danger"
                @click="handleDeleteRole(scope.row.roleId)"
              >删除</el-button>
            </div>
          </template>
        </el-table-column>
      </el-table>
    </div>
    <el-dialog
      title="新增"
      :visible.sync="dialogVisible"
      :close-on-click-modal="false"
      width="420px">
      <div>
        <el-row>
        <span style="color: red">* </span>
        <span>角色名称：</span>
        <el-input
          style="width: 200px"
          v-model="roleName"
          placeholder="请输入角色名称">
        </el-input>
        </el-row>
        <!-- <el-row style="margin-top: 20px">
        <span>角色排序：</span>
        <el-input
          style="width: 200px"
          v-model.number="sort"
          placeholder="请输入排序">
        </el-input>
        </el-row> -->
        <el-row style="margin-top: 20px">
        <span style="color: red">* </span>
        <span>角色描述：</span>
         <el-input
          style="width: 200px"
          placeholder="角色描述"
          v-model="describes">
        </el-input>
        </el-row>
        <el-row style="margin-top: 20px">
        <span style="color: red">* </span>
        <span>是否禁用：</span>
        <el-select
          style="width: 200px"
          v-model="enabled"
          placeholder="请选择">
          <el-option
            v-for="item in enableOptions"
            :key="item.value"
            :label="item.label"
            :value="item.value">
          </el-option>
        </el-select>
        </el-row>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="onRoleSubmit">确 定</el-button>
      </span>
    </el-dialog>
    <el-dialog
      title="分配权限"
      :visible.sync="dialogRoleVisible"
      width="300px">
      <div>
        <el-tree
          :data="treeData"
          ref="tree"
          show-checkbox
          node-key="resourceId"
          :default-checked-keys="defaultTreeData"
          @check-change="checkChange"
          :props="defaultProps">
        </el-tree>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="onAllotubmit">确 定</el-button>
      </span>
    </el-dialog>
    <el-dialog
      title="编辑"
      :visible.sync="updateWrapper"
      :close-on-click-modal="false"
      width="420px">
      <el-form label-width="100px" :model="adminForm" ref="adminForm" :rules="rules">
        <!-- <el-form-item label="排序：" prop="roleSort">
          <el-input
            style="width: 240px"
            placeholder="请输入排序"
            v-model.number="adminForm.roleSort"></el-input>
        </el-form-item> -->
        <el-form-item label="角色名称：" prop="roleName">
          <el-input
            style="width: 240px"
            placeholder="角色名称"
            v-model="adminForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述：" prop="describes">
          <el-input
            style="width: 240px"
            placeholder="角色描述"
            v-model="adminForm.describes"></el-input>
        </el-form-item>
        <el-form-item label="是否禁用：" prop="enabled">
          <el-select
            style="width: 240px"
            v-model="adminForm.enabled"
            placeholder="请选择">
            <el-option
              v-for="item in enableOptions"
              :key="item.value"
              :label="item.label"
              :value="item.value">
            </el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="onSubmitUpdate">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import moment from 'moment'
export default {
  name: 'Role',
  data () {
    return {
      list: [],
      dialogVisible: false,
      dialogRoleVisible: false,
      updateWrapper: false,
      roleName: '',
      enabled: '1',
      describes: '',
      roleId: '',
      adminForm: {
        roleId: '',
        roleName: '',
        describes: '',
        enabled: ''
      },
      rules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' }
        ],
        describes: [
          { required: true, message: '请输入角色描述', trigger: 'blur' }
        ],
        enabled: [
          { required: true, message: '请选择状态', trigger: 'blur' }
        ]
      },
      enableOptions: [
        {
          value: '1',
          label: '启用'
        },
        {
          value: '0',
          label: '禁用'
        }
      ],
      defaultProps: {
        children: 'children',
        label: 'name'
      },
      treeData: [],
      defaultTreeData: [],
      total: '',
      currentPage: 1,
      rolesIndexArr: [],
      rolesNameArr: [],
      currentSort: '',
      currentName: '',
      resourceList: [],
      disabledArr: [],
      treeParentIds: []
    }
  },
  created () {
    this.getSysRoleList()
    this.getSourceList()
  },
  watch: {
    dialogRoleVisible (newValue) {
      if (newValue === false) {
        this.defaultTreeDate = []
        this.$refs.tree.setCheckedNodes(this.defaultTreeDate)
      }
    }
  },
  methods: {
    getSysRoleList () {
      this.$api['role.list']({
        enabled: '',
        pageSize: 1000,
        pageNumber: this.currentPage
      }).then(res => {
        this.list = res.list
        if (this.list.length > 0) {
          this.rolesNameArr = this.list.map(name => name.roleName)
        }
      })
    },
    getSourceList () {
      this.$api['platformMenu.list']().then(res => {
        // this.treeData = res.resourceTree
        this.treeParentIds = []
        let initList = []
        let list = []
        let data = []
        initList = this.getResourceList(res.resourceTree)
        initList.forEach((item, index) => {
          list[index] = {
            resourceId: item.id,
            resourceParentId: item.parentId,
            name: item.name
          }
        })
        data = list.filter(item => item.resourceParentId === null)
        data.forEach((item, index) => {
          if (item.name !== '运营监控') {
            this.treeData[index] = {
              resourceId: item.resourceId,
              name: item.name,
              disabled: false,
              children: []
            }
          } else {
            this.treeData[index] = {
              resourceId: item.resourceId,
              name: item.name,
              disabled: true,
              children: []
            }
          }
        })
        this.treeData.forEach((tree, index) => {
          tree.children = list.filter(i => i.resourceParentId === tree.resourceId)
          tree.children.forEach(item => {
            item.disabled = false
          })
        })
        this.treeData.forEach((tree, index) => {
          tree.children.forEach(child => {
            child.children = list.filter(i => i.resourceParentId === child.resourceId)
            child.children.forEach(item => {
              item.disabled = false
            })
          })
        })
        this.getParentNode(this.treeData)
      })
    },
    // 递归遍历children
    getResourceList (resource) {
      resource.forEach(item => {
        this.resourceList.push({
          id: item.id,
          parentId: item.parentId,
          name: item.title
        })
        if (item.children.length > 0) {
          this.getResourceList(item.children)
        }
      })
      return this.resourceList
    },
    // 获取所有父节点
    getParentNode (params) {
      params.forEach(item => {
        if (item.children && item.children.length > 0) {
          this.treeParentIds.push(item.resourceId)
          this.getParentNode(item.children)
        }
      })
      // console.log(this.treeParentIds)
    },
    onRoleSubmit () {
      if (this.roleName.length > 0 && this.describes !== '') {
        if (!this.rolesNameArr.some(item => item === this.roleName)) {
          this.$api['role.add']({
            roleName: this.roleName,
            enabled: this.enabled,
            describes: this.describes
          }).then(res => {
            this.getSysRoleList()
            this.adminForm.roleName = this.roleName
            this.$message.success('添加成功!')
            this.dialogVisible = false
          }).catch(() => {
            this.$message.error('输入的内容不合法')
          })
        } else {
          this.$message.error('角色名称已存在')
        }
      } else {
        this.$message.error('请检查信息是否填写完整')
      }
    },
    handleDeleteRole (id) {
      this.$confirm('此操作将永久删除该角色和角色绑定的权限, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$api['role.delete']({
          id: id
        }).then(res => {
          this.getSysRoleList({
            pageSize: 1000,
            pageNumber: this.currentPage
          })
          this.$message.success('删除成功！')
        }).catch((err) => {
          this.$message.error(`${err.message}绑定了角色，不能删除`)
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    },
    handleAllot (row) {
      let arr = JSON.parse(JSON.stringify(row.resources))
      this.dialogRoleVisible = true
      this.roleId = row.roleId
      // arr = arr.filter(item => item.resourceId !== '1' &&
      //   item.resourceId !== '2' &&
      //   item.resourceId !== '11' &&
      //   item.resourceId !== '14' &&
      //   item.resourceId !== '20' &&
      //   item.resourceId !== '23'
      // )
      console.log('-------', this.treeParentIds)
      arr = arr.sort((prev, next) => Number(prev.resourceId) - Number(next.resourceId))
      this.treeParentIds.forEach(id => {
        if (arr.some(item => item.resourceId === id)) {
          arr = arr.filter(deleteItem => deleteItem.resourceId !== id)
        }
      })
      console.log(arr)
      if (arr && arr.length > 0) {
        // 初始化 进入树形结构时的选中id
        let currentIndex = ''
        arr = arr.map(res => res.resourceId)
        this.defaultTreeData = Array.from(new Set(arr))
        this.defaultTreeData = this.defaultTreeData.sort((prev, next) => prev - next)
        this.treeData.forEach((item) => {
          if (item.name === '运营监控') {
            // 拿到需要判断的树形结构的所有子节点
            const currentItemIds = item.children.map(id => id.resourceId)
            currentItemIds.forEach(current => {
              // 选出属性结构的选中节点
              if (this.defaultTreeData.some(date => date === current)) {
                currentIndex = current
              }
            })
            // 判断将没选中的属性节点设置成不能点击
            item.children.forEach(list => {
              if (currentIndex && list.resourceId !== currentIndex) {
                list.disabled = true
              } else if (!currentIndex) {
                list.disabled = false
              }
            })
          }
        })
      } else {
        this.defaultTreeData = []
      }
    },
    onAllotubmit () {
      let allresourceIds = []
      allresourceIds = this.$refs.tree.getHalfCheckedNodes().map(item => item.resourceId)
      allresourceIds = [...allresourceIds, ...this.$refs.tree.getCheckedKeys()]
      const resourceIds = allresourceIds
      this.$api['resource.updateRole']({
        resourceIds,
        roleId: this.roleId
      }).then(res => {
        this.dialogRoleVisible = false
        this.$message.success('分配成功！')
        this.defaultTreeData = []
        this.getSysRoleList()
      })
    },
    handleCheck (id) {
      this.updateWrapper = true
      this.adminForm.roleId = id
      this.$api['role.detail']({
        id
      }).then(res => {
        this.adminForm.roleName = res.roleName
        this.adminForm.describes = res.describes
        this.adminForm.enabled = res.enabled
        this.currentName = res.roleName
      })
      this.$nextTick(() => {
        this.$refs['adminForm'].resetFields()
      })
    },
    onSubmitUpdate () {
      this.$refs['adminForm'].validate((valid) => {
        if (valid) {
          if (this.currentName === this.adminForm.roleName ||
          !this.rolesNameArr.some(name => name === this.adminForm.roleName)) {
            this.$api['role.update'](this.adminForm).then(res => {
              this.getSysRoleList({
                pageSize: 1000,
                pageNumber: this.currentPage
              })
              this.$message.success('修改成功！')
              this.updateWrapper = false
            })
          } else {
            this.$message.error('角色名已存在')
          }
        } else {
          return false
        }
      })
    },
    handleAddRole () {
      this.roleName = ''
      this.describes = ''
      this.dialogVisible = true
    },
    getCreateTime (row) {
      return moment(row.createTime).format('YYYY-MM-DD HH:mm:ss')
    },
    getUpdateTime (row) {
      return moment(row.updateTime).format('YYYY-MM-DD HH:mm:ss')
    },
    checkChange (arr, obj, ccc) {
      let checkArr = arr.resourceId
      if ((checkArr === '24' && this.$refs.tree.getCheckedKeys().some(item => item === checkArr)) ||
        (checkArr === '25' && this.$refs.tree.getCheckedKeys().some(item => item === checkArr)) ||
        (checkArr === '26' && this.$refs.tree.getCheckedKeys().some(item => item === checkArr))) {
        this.treeData.forEach(item => {
          if (item.name === '运营监控') {
            item.children.forEach(child => {
              if (child.resourceId !== checkArr) {
                child.disabled = true
              } else {
                child.disabled = false
              }
            })
          }
        })
      } else if ((checkArr === '24' && this.$refs.tree.getCheckedKeys().some(item => item !== checkArr)) ||
        (checkArr === '25' && this.$refs.tree.getCheckedKeys().some(item => item !== checkArr)) ||
        (checkArr === '26' && this.$refs.tree.getCheckedKeys().some(item => item !== checkArr)) ||
        this.$refs.tree.getCheckedKeys().length === 0) {
        this.treeData.forEach(item => {
          if (item.name === '运营监控') {
            item.children.forEach(child => {
              child.disabled = false
            })
          }
        })
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.wrapper {
  width: 100%;
  padding: 20px 20px;
  box-sizing: border-box;
  .search {
    margin-bottom: 20px;
  }
}
</style>
