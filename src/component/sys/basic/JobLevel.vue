<template>
    <div>
        <div>
      <el-input size="small" class="input_type" placeholder="添加职称..."
        prefix-icon="el-icon-plus" @keydown.enter.native="addJobLevel" v-model="job.name">
      </el-input>
    <el-select v-model="job.titleLevel" placeholder="职称等级" size="small"
                       style="margin-left: 5px;margin-right: 5px">
                <el-option
                        v-for="item in titleLevels"
                        :key="item"
                        :label="item"
                        :value="item">
                </el-option>
            </el-select>
      <el-button type="primary" icon="el-icon-plus" size="small" @click="addJobLevel">
        添加
      </el-button>
    </div>
    <div>
        <el-table :data="joblevels" stripe border type="small" style="width: 70%" @selection-change="handleSelectionChange">
        <el-table-column type="selection" width="56"> </el-table-column>
        <el-table-column prop="id" label="编号" width="56"> </el-table-column>
        <el-table-column prop="name" label="职称" width="180"> </el-table-column>
        <el-table-column prop="titleLevel" label="级别" width="180"> </el-table-column>
        <el-table-column prop="createDate" label="创建时间" width="200"> </el-table-column>
        <el-table-column fixed="right" label="操作">
          <template slot-scope="scope">
            <el-button size="mini" @click="showEditDialog(scope.$index, scope.row)">编辑</el-button>
            <el-button size="mini" type="danger" @click="handleDelete(scope.$index, scope.row)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
            <el-button type="danger" size="small" style="margin-top: 8px" @click="deleteMany"
                 :disabled="multipleSelection.length === 0">
        批量删除
      </el-button>
    </div>
        <el-dialog title="修改职称" :visible.sync="dialogVisible" width="30%">
         <div>
        <el-tag>职称</el-tag>
        <el-input class="update_input" size="small" v-model="updateJob.name"></el-input>
      </div>
      <br>
      <div>
        <el-tag>等级</el-tag>

           <el-select v-model="updateJob.titleLevel" placeholder="职称等级" size="small"
                       style="margin-left: 5px;margin-right: 5px">
                <el-option
                        v-for="item in titleLevels"
                        :key="item"
                        :label="item"
                        :value="item">
                </el-option>
            </el-select>
      </div>
       <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false" size="small" >取 消</el-button>
        <el-button type="primary" @click="doUpdate" size="small" >确 定</el-button>
      </span>
      </el-dialog>
    </div>
</template>
<script>
export default {
    name:'JobLevel',
    data(){
        return{
            job:{
                name:'',  
                 titleLevel: '',
            },
            updateJob:{
                name:'',
                 titleLevel: '',
            },
            joblevels:[],
            titleLevels:[
               '正高级',
                    '副高级',
                    '中级',
                    '初级',
                    '员级',
            ],
             // 对话框显示与否的标志位
      dialogVisible: false,
      // 批量删除的数据记录
      multipleSelection: []
        }
    },
    methods:{
        async initJobLevels () {
      const data = await this.getRequest('/system/basic/job/')
      if (data) {
        this.joblevels = data.obj
      }
    },
       // 添加新记录的事件处理
    async addJobLevel() {
      if (this.job.name && this.job.titleLevel) {
        const resp = await this.postRequest('/system/basic/job/', this.job)
        if (resp) {
          this.initJobLevels()
          this.job.name = ''  
          this.job.titleLevel=''
        }
       } 
      else {
        this.$message.error('字段不能为空')
      }
    },   
        // 显示修改对话框
    showEditDialog (index, data) {
      // this.updatePos = data // 浅拷贝会改变表格的记录
      Object.assign(this.updateJob, data)  // 使用深拷贝
      this.dialogVisible = true
    },
     // 弹框确认修改的事件处理
    async doUpdate() {
      const resp = await this.putRequest('/system/basic/job/', this.updateJob)
      if (resp) {
        this.initJobLevels()
        this.updateJob.name = ''
         this.updateJob.titleLevel = ''
        this.dialogVisible = false
      }
    },
    // 表格记录的删除按钮的事件处理
    handleDelete (index, data) {
      this.$confirm('此操作将永久删除' + data.name + '职称, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.deleteRequest('/system/basic/job/' + data.id).then(resp => {
          this.initJobLevels()
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消操作'
        });
      });
    },
    // 记录多选的处理
    handleSelectionChange(val) {
      console.log(val)
      this.multipleSelection = val
    },
    // 批量删除
    deleteMany() {
      this.$confirm('此操作将永久删除' + this.multipleSelection.length + '条记录, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        // 生成删除记录 id的查询字符串
        let ids = "?"
        this.multipleSelection.forEach(item => {
          ids += "ids=" + item.id + '&'
        })
        this.deleteRequest('/system/basic/job/' + ids).then(resp => {
          this.initJobLevels()
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消操作'
        });
      });
    }
    },
    mounted(){
        this.initJobLevels()
    }
}
</script>
<style scoped>
.input_type {
    width: 300px;
    margin-right: 8px;
    margin-bottom: 16px;
  }
  .update_input {
    width: 200px;
    margin-left: 8px;
  }
</style>
