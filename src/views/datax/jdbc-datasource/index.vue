<template>
  <div class="app-container">
    <div class="filter-container">
      <el-input
        v-model="listQuery.datasourceName"
        clearable
        placeholder="数据源名称"
        style="width: 200px;"
        class="filter-item"
        @keyup.enter.native="handleFilter"
      />
      <el-button v-waves class="filter-item" type="primary" icon="el-icon-search" @click="fetchData">
        搜索
      </el-button>
      <el-button class="filter-item" style="margin-left: 10px;" type="primary" icon="el-icon-edit" @click="handleCreate">
        添加
      </el-button>
      <!-- <el-checkbox v-model="showReviewer" class="filter-item" style="margin-left:15px;" @change="tableKey=tableKey+1">
        reviewer
      </el-checkbox> -->
    </div>
    <el-table
      v-loading="listLoading"
      :data="list"
      element-loading-text="Loading"
      border
      fit
      highlight-current-row
    >
      <!-- <el-table-column align="center" label="序号" width="95">
        <template slot-scope="scope">{{ scope.$index }}</template>
      </el-table-column> -->
      <el-table-column label="数据源ID" width="95" align="center">
        <template slot-scope="scope">{{ scope.row.id }}</template>
      </el-table-column>
      <el-table-column label="数据源" width="200" align="center">
        <template slot-scope="scope">{{ scope.row.datasource }}</template>
      </el-table-column>
      <el-table-column label="数据源名称" width="150" align="center">
        <template slot-scope="scope">{{ scope.row.datasourceName }}</template>
      </el-table-column>
      <el-table-column label="数据源分组" width="200" align="center">
        <template slot-scope="scope">{{ scope.row.datasourceGroup }}
        </template>
      </el-table-column>
      <!--<el-table-column label="用户名" width="150" align="center">
        <template slot-scope="scope">{{ scope.row.jdbcUsername ? scope.row.jdbcUsername:'-' }}</template>
      </el-table-column>-->
      <el-table-column label="jdbc连接串" align="center" :show-overflow-tooltip="true">
        <template slot-scope="scope">{{ scope.row.jdbcUrl ? scope.row.jdbcUrl:'-' }}</template>
      </el-table-column>
      <!-- <el-table-column label="jdbc驱动类" width="200" align="center" :show-overflow-tooltip="true">
        <template slot-scope="scope">{{ scope.row.jdbcDriverClass ? scope.row.jdbcDriverClass:'-' }}</template>
      </el-table-column>-->
      <el-table-column label="ZK地址" width="200" align="center" :show-overflow-tooltip="true">
        <template slot-scope="scope">{{ scope.row.zkAdress ? scope.row.zkAdress:'-' }}</template>
      </el-table-column>
      <el-table-column label="数据库名" width="200" align="center" :show-overflow-tooltip="true">-->
        <template slot-scope="scope">{{ scope.row.databaseName ? scope.row.databaseName:'-' }}</template>-->
      </el-table-column>
      <el-table-column label="备注" width="150" align="center">
        <template slot-scope="scope">{{ scope.row.comments }}</template>
      </el-table-column>
      <el-table-column label="操作" align="center" width="230" fixed="right" class-name="small-padding fixed-width">
        <template slot-scope="{row}">
          <el-button type="primary" size="mini" @click="handleUpdate(row)">
            编辑
          </el-button>
          <el-button v-if="row.status!='deleted'" size="mini" type="danger" @click="handleDelete(row)">
            删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <pagination
      v-show="total>0"
      :total="total"
      :page.sync="listQuery.current"
      :limit.sync="listQuery.size"
      @pagination="fetchData"
    />

    <el-dialog :title="textMap[dialogStatus]" :visible.sync="dialogFormVisible" :close-on-press-escape="false" :close-on-click-modal="false" width="800px">
      <el-form ref="dataForm" :rules="rules" :model="temp" label-position="left" label-width="100px">
        <el-form-item label="数据源" prop="datasource">
          <el-select
            v-model="temp.datasource"
            placeholder="数据源"
            style="width: 200px"
            @change="selectDataSource(temp.datasource)"
          >
            <el-option v-for="item in dataSources" :key="item.value" :label="item.label" :value="item.value" />
          </el-select>
        </el-form-item>
        <el-form-item label="数据源名称" prop="datasourceName">
          <el-input v-model="temp.datasourceName" placeholder="数据源名称" style="width: 40%" />
        </el-form-item>
        <el-form-item label="数据源分组" prop="datasourceGroup">
          <el-input v-model="temp.datasourceGroup" placeholder="数据源分组" style="width: 40%" />
        </el-form-item>
        <el-form-item v-if="jdbc" label="用户名">
          <el-input v-model="temp.jdbcUsername" placeholder="用户名" style="width: 40%" />
        </el-form-item>
        <el-form-item v-if="visible" v-show="jdbc" label="密码">
          <el-input v-model="temp.jdbcPassword" type="password" placeholder="密码" style="width: 40%">
            <i slot="suffix" title="显示密码" style="cursor:pointer" class="el-icon-view" @click="changePass('show')" />
          </el-input>
        </el-form-item>
        <el-form-item v-show="jdbc" v-else label="密码">
          <el-input v-model="temp.jdbcPassword" type="text" placeholder="密码" style="width: 40%">
            <i slot="suffix" title="隐藏密码" style="cursor:pointer" class="el-icon-check" @click="changePass('hide')" />
          </el-input>
        </el-form-item>
        <el-form-item v-if="jdbc" label="jdbc url" prop="jdbcUrl">
          <el-input
            v-model="temp.jdbcUrl"
            :autosize="{ minRows: 3, maxRows: 6}"
            type="textarea"
            placeholder="jdbc url"
            style="width: 60%"
          />
        </el-form-item>
        <el-form-item v-if="mongodb" label="地址" prop="jdbcUrl">
          <el-input
            v-model="temp.jdbcUrl"
            :autosize="{ minRows: 3, maxRows: 6}"
            type="textarea"
            placeholder="127.0.0.1:27017"
            style="width: 60%"
          />
        </el-form-item>
        <el-form-item v-if="jdbc" label="jdbc驱动类" prop="jdbcDriverClass">
          <el-input v-model="temp.jdbcDriverClass" placeholder="jdbc驱动类" style="width: 60%" />
        </el-form-item>
        <el-form-item v-if="hbase" label="ZK地址" prop="zkAdress">
          <el-input v-model="temp.zkAdress" placeholder="127.0.0.1:2181" style="width: 60%" />
        </el-form-item>
        <el-form-item v-if="mongodb" label="数据库名称" prop="databaseName">
          <el-input v-model="temp.databaseName" placeholder="数据库名称" style="width: 60%" />
        </el-form-item>
        <el-form-item label="注释">
          <el-input
            v-model="temp.comments"
            :autosize="{ minRows: 2, maxRows: 4}"
            type="textarea"
            placeholder="Please input"
            style="width: 60%"
          />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">
          取消
        </el-button>
        <el-button type="primary" @click="dialogStatus==='create'?createData():updateData()">
          确认
        </el-button>
        <el-button type="primary" @click="testDataSource()">
          测试连接
        </el-button>
      </div>
    </el-dialog>
    <el-dialog :visible.sync="dialogPluginVisible" title="Reading statistics">
      <el-table :data="pluginData" border fit highlight-current-row style="width: 100%">
        <el-table-column prop="key" label="Channel" />
        <el-table-column prop="pv" label="Pv" />
      </el-table>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="dialogPvVisible = false">Confirm</el-button>
      </span>
    </el-dialog>
    <el-dialog :title="'更新任务数据源'" :visible.sync="dialogUpdateJobVisible" :close-on-press-escape="false" :close-on-click-modal="false" width="800px">
      <div>
        当前使用数据源：{{ temp.datasourceName }}
      </div>
      <div style="margin-top: 20px">
        <el-radio-group v-model="batchUpdateJobDatasourceType">
          <el-radio :label="0">替换reader</el-radio>
          <el-radio :label="1">替换writer</el-radio>
        </el-radio-group>
      </div>
      <el-divider />
      <div style="margin-top: 20px">
        <el-radio-group v-model="batchUpdateJobDatasourceOptionType">
          <el-radio :label="0">按项目选择</el-radio>
          <el-radio :label="1">按任务选择</el-radio>
        </el-radio-group>
      </div>
      <el-transfer
        filterable
        filter-placeholder="搜索"
        :titles="['列表', '目标']"
        v-model="batchUpdateJobDatasourceList"
        :data="batchUpdateJobDatasourceOptionList"
        style="margin-top: 20px">
      </el-transfer>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogUpdateJobVisible = false">
          取消
        </el-button>
        <el-button type="primary" @click="handleBatchUpdateJobDatasource">
          确认
        </el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import * as datasourceApi from '@/api/datax-jdbcDatasource'
import waves from '@/directive/waves' // waves directive
import { parseTime } from '@/utils'
import Pagination from '@/components/Pagination'
import * as job from '@/api/datax-job-info'
import * as jobProjectApi from '@/api/datax-job-project'

export default {
  name: 'JdbcDatasource',
  components: { Pagination },
  directives: { waves },
  filters: {
    statusFilter(status) {
      const statusMap = {
        published: 'success',
        draft: 'gray',
        deleted: 'danger'
      }
      return statusMap[status]
    }
  },
  data() {
    return {
      list: null,
      listLoading: true,
      total: 0,
      listQuery: {
        current: 1,
        size: 10
      },
      pluginTypeOptions: ['reader', 'writer'],
      dialogPluginVisible: false,
      pluginData: [],
      dialogFormVisible: false,
      dialogUpdateJobVisible: false,
      dialogStatus: '',
      textMap: {
        update: 'Edit',
        create: 'Create'
      },
      rules: {
        datasourceName: [{ required: true, message: 'this is required', trigger: 'blur' }],
        jdbcUsername: [{ required: true, message: 'this is required', trigger: 'blur' }],
        jdbcPassword: [{ required: true, message: 'this is required', trigger: 'blur' }],
        jdbcUrl: [{ required: true, message: 'this is required', trigger: 'blur' }],
        jdbcDriverClass: [{ required: true, message: 'this is required', trigger: 'blur' }],
        datasource: [{ required: true, message: 'this is required', trigger: 'change' }],
        zkAdress: [{ required: true, message: 'this is required', trigger: 'blur' }],
        databaseName: [{ required: true, message: 'this is required', trigger: 'blur' }]
      },
      temp: {
        id: undefined,
        datasourceName: '',
        datasourceGroup: 'Default',
        jdbcUsername: '',
        jdbcPassword: '',
        jdbcUrl: '',
        jdbcDriverClass: '',
        comments: '',
        datasource: '',
        zkAdress: '',
        databaseName: ''
      },
      visible: true,
      dataSources: [
        { value: 'mysql', label: 'mysql' },
        { value: 'oracle', label: 'oracle' },
        { value: 'postgresql', label: 'postgresql' },
        { value: 'sqlserver', label: 'sqlserver' },
        { value: 'hive', label: 'hive' },
        { value: 'hbase', label: 'hbase' },
        { value: 'mongodb', label: 'mongodb' },
        { value: 'clickhouse', label: 'clickhouse' }
      ],
      jdbc: true,
      hbase: false,
      mongodb: false,
      batchUpdateJobDatasourceType: 0, // 0-reader 1-writer
      batchUpdateJobDatasourceOptionType: '', // 选项类别 0-项目 1-任务
      batchUpdateJobDatasourceOptionList: [],
      batchUpdateJobDatasourceList: [],
      jobIdList: [],
      jobProjectList: []
    }
  },
  watch: {
    batchUpdateJobDatasourceOptionType: function(label) {
      this.batchUpdateJobDatasourceOptionList = label === 0 ? this.jobProjectList : this.jobIdList
    }
  },
  created() {
    this.fetchData()
  },
  methods: {
    selectDataSource(datasource) {
      if (datasource === 'mysql') {
        this.temp.jdbcUrl = 'jdbc:mysql://{host}:{port}/{database}'
        this.temp.jdbcDriverClass = 'com.mysql.jdbc.Driver'
      } else if (datasource === 'oracle') {
        this.temp.jdbcUrl = 'jdbc:oracle:thin:@//{host}:{port}/{database}'
        this.temp.jdbcDriverClass = 'oracle.jdbc.OracleDriver'
      } else if (datasource === 'postgresql') {
        this.temp.jdbcUrl = 'jdbc:postgresql://{host}:{port}/{database}'
        this.temp.jdbcDriverClass = 'org.postgresql.Driver'
      } else if (datasource === 'sqlserver') {
        this.temp.jdbcUrl = 'jdbc:sqlserver://{host}:{port};DatabaseName={database}'
        this.temp.jdbcDriverClass = 'com.microsoft.sqlserver.jdbc.SQLServerDriver'
      } else if (datasource === 'clickhouse') {
        this.temp.jdbcUrl = 'jdbc:clickhouse://{host}:{port}/{database}'
        this.temp.jdbcDriverClass = 'ru.yandex.clickhouse.ClickHouseDriver'
      } else if (datasource === 'hive') {
        this.temp.jdbcUrl = 'jdbc:hive2://{host}:{port}/{database}'
        this.temp.jdbcDriverClass = 'org.apache.hive.jdbc.HiveDriver'
        this.hbase = this.mongodb = false
        this.jdbc = true
      }
      this.getShowStrategy(datasource)
    },
    fetchData() {
      this.listLoading = true
      datasourceApi.list(this.listQuery).then(response => {
        const { records } = response
        const { total } = response
        this.total = total
        this.list = records
        this.listLoading = false
      })
    },
    resetTemp() {
      this.temp = {
        id: undefined,
        datasourceName: '',
        datasourceGroup: 'Default',
        jdbcUsername: '',
        jdbcPassword: '',
        jdbcUrl: '',
        jdbcDriverClass: '',
        comments: ''
      }
    },
    handleCreate() {
      this.resetTemp()
      this.dialogStatus = 'create'
      this.dialogFormVisible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].clearValidate()
      })
      this.getJobIdList()
      console.log(this.jobIdList)
    },
    createData() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          datasourceApi.created(this.temp).then(() => {
            this.fetchData()
            this.dialogFormVisible = false
            this.$notify({
              title: 'Success',
              message: 'Created Successfully',
              type: 'success',
              duration: 2000
            })
          })
        }
      })
    },
    testDataSource() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          datasourceApi.test(this.temp).then(response => {
            if (response.data === false) {
              this.$notify({
                title: 'Fail',
                message: response.data.msg,
                type: 'fail',
                duration: 2000
              })
            } else {
              this.$notify({
                title: 'Success',
                message: 'Tested Successfully',
                type: 'success',
                duration: 2000
              })
            }
          })
        }
      })
    },
    handleUpdate(row) {
      this.getShowStrategy(row.datasource)
      this.temp = Object.assign({}, row) // copy obj
      this.dialogStatus = 'update'
      this.dialogFormVisible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].clearValidate()
      })
    },
    updateData() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          const tempData = Object.assign({}, this.temp)
          datasourceApi.updated(tempData).then(() => {
            this.fetchData()
            this.dialogFormVisible = false
            this.$notify({
              title: 'Success',
              message: 'Update Successfully',
              type: 'success',
              duration: 2000
            })
            if (this.jdbc) {
              this.$confirm('更新成功，是否同步更新任务数据源信息', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'success'
              }).then(() => {
                this.getJobIdList()
                this.getJobProjectList()
                this.dialogUpdateJobVisible = true
              })
            }
          })
        }
      })
    },
    getShowStrategy(datasource) {
      if (datasource === 'hbase') {
        this.jdbc = this.mongodb = false
        this.hbase = true
      } else if (datasource === 'mongodb') {
        this.jdbc = this.hbase = false
        this.mongodb = true
        this.temp.jdbcUrl = 'mongodb://[username:password@]host1[:port1][,...hostN[:portN]]][/[database][?options]]'
      } else {
        this.hbase = this.mongodb = false
        this.jdbc = true
      }
    },
    handleDelete(row) {
      this.$confirm('确定删除吗？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        datasourceApi.deleted({ idList: row.id }).then(response => {
          this.fetchData()
          this.$notify({
            title: 'Success',
            message: 'Delete Successfully',
            type: 'success',
            duration: 2000
          })
        })
      })
      // const index = this.list.indexOf(row)
    },
    handleFetchPv(id) {
      datasourceApi.fetched(id).then(response => {
        this.pluginData = response
        this.dialogPvVisible = true
      })
    },
    formatJson(filterVal, jsonData) {
      return jsonData.map(v => filterVal.map(j => {
        if (j === 'timestamp') {
          return parseTime(v[j])
        } else {
          return v[j]
        }
      }))
    },
    changePass(value) {
      this.visible = !(value === 'show')
    },
    getJobIdList() {
      job.getJobIdList().then(response => {
        const { content } = response
        this.jobIdList = []
        content.forEach(item => {
          this.jobIdList.push({
            key: item.id,
            label: item.jobDesc
          })
        })
      })
    },
    getJobProjectList() {
      jobProjectApi.getJobProjectList().then(response => {
        this.jobProjectList = []
        response.forEach(item => {
          this.jobProjectList.push({
            key: item.id,
            label: item.name
          })
        })
      })
    },
    handleBatchUpdateJobDatasource() {
      let message = ''
      if (this.temp.id === undefined) {
        message = '数据源为空'
      }
      if (this.batchUpdateJobDatasourceType !== 0 && this.batchUpdateJobDatasourceType !== 1) {
        message = '数据源content类型错误'
      }
      if (this.batchUpdateJobDatasourceOptionType !== 0 && this.batchUpdateJobDatasourceType !== 1) {
        message = '更新列表类型错误'
      }
      if (this.batchUpdateJobDatasourceList === undefined || this.batchUpdateJobDatasourceList.length === 0) {
        message = '更新列表错误或列表为空'
      }
      if (message !== '') {
        this.$alert(message, '出错了', {
          confirmButtonText: '确定'
        })
      } else {
        job.batchUpdateJobDatasource({
          datasourceId: this.temp.id,
          batchUpdateJobDatasourceType: this.batchUpdateJobDatasourceType,
          batchUpdateJobOptionType: this.batchUpdateJobDatasourceOptionType,
          batchUpdateJobList: this.batchUpdateJobDatasourceList
        }).then(() => {
          this.dialogUpdateJobVisible = false
          this.resetBatchUpdateJobDatasource()
          this.$notify({
            title: 'Success',
            message: 'Update Successfully',
            type: 'success',
            duration: 2000
          })
        })
      }
    },
    resetBatchUpdateJobDatasource() {
      this.batchUpdateJobDatasourceType = 0
      this.batchUpdateJobDatasourceOptionType = ''
      this.batchUpdateJobDatasourceList = []
    }
  }
}
</script>
