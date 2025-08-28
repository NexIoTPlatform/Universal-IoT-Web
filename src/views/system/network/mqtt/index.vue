<template>
  <page-header-wrapper>
    <a-card :bordered="false">
      <!-- 条件搜索 -->
      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row :gutter="24">
            <a-col :lg="6" :md="8" :sm="12" :xs="24">
              <a-form-item label="MQTT类型" prop="type">
                <a-select placeholder="请选择MQTT类型" style="width: 100%" v-model="queryParam.type"
                          allow-clear>
                  <a-select-option value="MQTT_CLIENT">
                    <span class="type-cell">
                      <a-icon type="cloud" style="color: #fa8c16; margin-right: 8px;"/>
                      <span>MQTT客户端</span>
                    </span>
                  </a-select-option>
                  <a-select-option value="MQTT_SERVER">
                    <span class="type-cell">
                      <a-icon type="cloud-server" style="color: #eb2f96; margin-right: 8px;"/>
                      <span>MQTT服务端</span>
                    </span>
                  </a-select-option>
                </a-select>
              </a-form-item>
            </a-col>
            <a-col :lg="6" :md="8" :sm="12" :xs="24">
              <a-form-item label="组件名称" prop="name">
                <a-input v-model.trim="queryParam.name" placeholder="请输入MQTT组件名称"
                         allow-clear>
                  <a-icon slot="prefix" type="tag"/>
                </a-input>
              </a-form-item>
            </a-col>
            <!-- <a-col :lg="6" :md="8" :sm="12" :xs="24">
              <a-form-item label="产品Key" prop="productKey">
                <a-input v-model.trim="queryParam.productKey" placeholder="请输入产品Key" allow-clear>
                  <a-icon slot="prefix" type="appstore" />
                </a-input>
              </a-form-item>
            </a-col> -->
            <a-col :lg="6" :md="8" :sm="12" :xs="24">
              <a-form-item label="运行状态" prop="running">
                <a-select placeholder="请选择运行状态" style="width: 100%"
                          v-model="queryParam.running" allow-clear>
                  <a-select-option :value="true">
                    <a-icon type="check-circle" style="color: #52c41a; margin-right: 8px;"/>
                    运行中
                  </a-select-option>
                  <a-select-option :value="false">
                    <a-icon type="stop" style="color: #ff4d4f; margin-right: 8px;"/>
                    已停止
                  </a-select-option>
                </a-select>
              </a-form-item>
            </a-col>
            <!-- <a-col :lg="6" :md="8" :sm="12" :xs="24">
              <a-form-item label="唯一标识" prop="unionId">
                <a-input v-model.trim="queryParam.unionId" placeholder="请输入唯一标识" allow-clear>
                  <a-icon slot="prefix" type="key" />
                </a-input>
              </a-form-item>
            </a-col> -->
            <a-col :lg="6" :md="8" :sm="12" :xs="24">
              <span class="table-page-search-submitButtons">
                <a-button type="primary" @click="handleQuery" icon="search">查询</a-button>
                <a-button style="margin-left: 8px" @click="resetQuery" icon="reload">重置</a-button>
              </span>
            </a-col>
          </a-row>
        </a-form>
      </div>
      <!-- 操作 -->
      <!-- 操作 -->
      <div class="table-operations">
        <a-space>
          <a-button type="primary" @click="$refs.createForm.handleAdd()"
                    v-hasPermi="['network:mqtt:add']" icon="plus">
            新增
          </a-button>
          <a-button
            type="danger"
            :disabled="multiple"
            @click="handleDelete"
            v-hasPermi="['network:mqtt:remove']"
            ghost
            icon="delete">
            删除
          </a-button>
          <a-button @click="handleExport" v-hasPermi="['network:mqtt:export']" icon="export">
            导出
          </a-button>
        </a-space>
        <a-button
          type="dashed"
          shape="circle"
          :loading="loading"
          icon="reload"
          @click="getList"
          class="refresh-btn"
        />
      </div>
      <!-- 增加修改 -->
      <create-form
        ref="createForm"
        :networkTypeOptions="mqttTypeOptions"
        :productOptions="productOptions"
        @ok="getList"
      />

      <!-- 空状态 -->
      <a-empty v-if="!loading && list.length === 0" description="暂无MQTT网络组件数据"/>

      <!-- 卡片网格 -->
      <a-row gutter="16">
        <a-col :span="6" v-for="item in list" :key="item.id">
          <a-card hoverable class="network-card">
            <div class="card-header">
              <span style="display:flex;align-items:center;">
                <a-badge :status="item.running ? 'success' : 'default'"
                         :class="{ 'breath-badge': item.running }"
                         style="margin-right:12px;font-size:18px;line-height:1;"/>
              </span>
              <a @click="handleView(item)" class="card-title">
                {{ getDisplayName(item) }}
                <a-icon type="cloud" style="color:#faad14;margin-left:4px;"/>
              </a>
            </div>
            <div class="card-body">
              <div class="card-row">
                <a-icon type="global" style="margin-right:4px;"/>
                Broker地址：
                <a-tooltip :title="getConfigValue(item, 'host')">{{
                    getConfigValue(item, 'host')
                  }}
                </a-tooltip>
              </div>
              <div class="card-row">
                <a-icon type="key" style="margin-right:4px;"/>
                Broker标识：
                <a-tooltip :title="item.unionId">{{ item.unionId }}</a-tooltip>
              </div>
              <div class="card-row">
                <a-icon type="team" style="margin-right:4px;"/>
                连接设备数：<span>{{ item.deviceCount || 0 }}</span></div>
            </div>
            <div class="card-actions">
              <a-button
                type="link"
                size="small"
                @click="handleStart(item)"
                v-hasPermi="['network:mqtt:start']"
                :disabled="!item.state || item.running">
                启动
              </a-button>
              <a-button
                type="link"
                size="small"
                @click="handleStop(item)"
                v-hasPermi="['network:mqtt:stop']"
                :disabled="!item.state || !item.running">
                停止
              </a-button>
              <a-tooltip title="编辑">
                <a-button type="link" size="small" @click="$refs.createForm.handleUpdate(item)"
                          v-hasPermi="['network:mqtt:edit']">
                  <a-icon type="edit"/>
                </a-button>
              </a-tooltip>
              <a-tooltip title="删除">
                <a-button type="link" size="small" @click="handleDelete(item)"
                          v-hasPermi="['network:mqtt:remove']" class="delete-btn">
                  <a-icon type="delete"/>
                </a-button>
              </a-tooltip>
            </div>
          </a-card>
        </a-col>
      </a-row>

      <a-pagination
        v-if="total > 0"
        class="ant-table-pagination"
        show-size-changer
        show-quick-jumper
        :current="queryParam.pageNum"
        :total="total"
        :page-size="queryParam.pageSize || 8"
        :pageSizeOptions="['4','8','12','16']"
        :showTotal="total => `共 ${total} 条`"
        @showSizeChange="onShowSizeChange"
        @change="changeSize"
      />
    </a-card>
  </page-header-wrapper>
</template>

<script>
import {
  delNetwork,
  delNetworkBatch,
  listNetwork,
  startNetwork,
  stopNetwork
} from '@/api/system/network'
import {listProduct} from '@/api/system/dev/product'
import CreateForm from '../modules/CreateForm'
import {parseTime} from '@/utils/ruoyi'

export default {
  name: 'MqttNetwork',
  components: {
    CreateForm
  },
  data() {
    return {
      // 遮罩层
      loading: false,
      // 选中数组
      ids: [],
      // 非单个禁用
      single: true,
      // 非多个禁用
      multiple: true,
      // 显示搜索条件
      advanced: false,
      // 总条数
      total: 0,
      // MQTT网络组件表格数据
      list: [],
      // 弹出层标题
      title: '',
      // 是否显示弹出层
      open: false,
      // MQTT类型选项
      mqttTypeOptions: [
        {dictValue: 'MQTT_CLIENT', dictLabel: 'MQTT客户端'},
        {dictValue: 'MQTT_SERVER', dictLabel: 'MQTT服务端'}
      ],
      // 产品选项
      productOptions: [],
      // 查询参数
      queryParam: {
        pageNum: 1,
        pageSize: 12,
        type: undefined,
        name: undefined,
        productKey: undefined,
        running: undefined,
        unionId: undefined
      },
      // 表格列配置
      columns: [
        {
          title: '组件名称',
          dataIndex: 'name',
          scopedSlots: {customRender: 'name'},
          ellipsis: true,
          width: '28%'
        },
        {
          title: '产品Key',
          dataIndex: 'productKey',
          ellipsis: true,
          width: '32%'
        },
        {
          title: '唯一标识',
          dataIndex: 'unionId',
          ellipsis: true,
          width: '28%'
        },
        {
          title: '操作',
          dataIndex: 'operation',
          scopedSlots: {customRender: 'operation'},
          width: '12%',
          align: 'center',
          fixed: false
        }
      ]
    }
  },
  created() {
    this.getList()
    this.getProductOptions()
  },
  methods: {
    parseTime,
    /** 查询MQTT网络组件列表 */
    getList() {
      this.loading = true
      // 只查询MQTT类型的网络组件
      const params = {...this.queryParam}
      if (!params.type) {
        params.type = ['MQTT_CLIENT', 'MQTT_SERVER']
      }

      listNetwork(params).then(response => {
        this.list = response.rows
        this.total = response.total
        this.loading = false
      }).catch(() => {
        this.loading = false
      })
    },
    /** 获取产品选项 */
    getProductOptions() {
      listProduct({pageSize: 1000}).then(response => {
        this.productOptions = response.rows.map(item => ({
          value: item.productKey,
          label: item.name
        }))
      })
    },
    /** 搜索按钮操作 */
    handleQuery() {
      this.queryParam.pageNum = 1
      this.getList()
    },
    /** 重置按钮操作 */
    resetQuery() {
      this.queryParam = {
        pageNum: 1,
        pageSize: 10,
        type: undefined,
        name: undefined,
        productKey: undefined,
        running: undefined,
        unionId: undefined
      }
      this.handleQuery()
    },
    onShowSizeChange(current, pageSize) {
      this.queryParam.pageSize = pageSize
      this.getList()
    },
    changeSize(current, pageSize) {
      this.queryParam.pageNum = current
      this.queryParam.pageSize = pageSize
      this.getList()
    },
    /** 多选框选中数据 */
    onSelectChange(selectedRowKeys, selectedRows) {
      this.ids = selectedRowKeys
      this.single = selectedRowKeys.length !== 1
      this.multiple = !selectedRowKeys.length
    },
    /** 启动MQTT网络组件 */
    handleStart(row) {
      this.$confirm({
        title: '确认操作',
        content: `确定要启动MQTT网络组件"${row.name}"吗？`,
        onOk: () => {
          startNetwork(row.id).then(response => {
            this.$message.success('启动成功')
            this.getList()
          }).catch(error => {
            this.$message.error(error.msg || '启动失败')
          })
        }
      })
    },
    /** 停止MQTT网络组件 */
    handleStop(row) {
      this.$confirm({
        title: '确认操作',
        content: `确定要停止MQTT网络组件"${row.name}"吗？`,
        onOk: () => {
          stopNetwork(row.id).then(response => {
            this.$message.success('停止成功')
            this.getList()
          }).catch(error => {
            this.$message.error(error.msg || '停止失败')
          })
        }
      })
    },
    /** 删除按钮操作 */
    handleDelete(row) {
      const ids = row.id || this.ids
      const names = row.name || this.list.filter(item => this.ids.includes(item.id)).map(
        item => item.name).join(',')
      this.$confirm({
        title: '确认删除',
        content: `确定要删除MQTT网络组件"${names}"吗？`,
        onOk: () => {
          const api = Array.isArray(ids) ? delNetworkBatch : delNetwork
          const params = Array.isArray(ids) ? ids.join(',') : ids
          api(params).then(response => {
            this.$message.success('删除成功')
            this.getList()
          })
        }
      })
    },
    /** 查看详情 */
    handleView(row) {
      this.$router.push(`/system/network/mqtt/detail/${row.id}`)
    },
    /** 导出按钮操作 */
    handleExport() {
      this.$message.info('导出功能待实现')
    },
    typeShortLabel(item) {
      if (item.type === 'MQTT_CLIENT') {
        return 'MQTT C'
      }
      if (item.type === 'MQTT_SERVER') {
        return 'MQTT S'
      }
      return item.type
    },
    typeFullLabel(item) {
      if (item.type === 'MQTT_CLIENT') {
        return 'MQTT客户端'
      }
      if (item.type === 'MQTT_SERVER') {
        return 'MQTT服务端'
      }
      return item.type
    },
    typeTagStyle(item) {
      if (item.type === 'MQTT_CLIENT') {
        return {
          background: '#fff7e6',
          color: '#fa8c16',
          borderRadius: '12px',
          fontSize: '11px',
          border: 'none',
          padding: '0 10px',
          display: 'inline-flex',
          alignItems: 'center',
          gap: '4px'
        }
      }
      if (item.type === 'MQTT_SERVER') {
        return {
          background: '#fff0f6',
          color: '#eb2f96',
          borderRadius: '12px',
          fontSize: '11px',
          border: 'none',
          padding: '0 10px',
          display: 'inline-flex',
          alignItems: 'center',
          gap: '4px'
        }
      }
      return {
        background: '#fafafa',
        color: '#999',
        borderRadius: '12px',
        fontSize: '11px',
        border: 'none',
        padding: '0 10px',
        display: 'inline-flex',
        alignItems: 'center',
        gap: '4px'
      }
    },
    getDisplayName(item) {
      // 如果产品名称不为空，显示产品名称，否则显示组件名称
      const productName = this.productName(item)
      return productName || item.name
    },
    productName(item) {
      const found = this.productOptions.find(opt => opt.value === item.productKey)
      return found ? found.label : ''
    },
    getConfigValue(item, key, defaultValue = '未配置') {
      try {
        const config = JSON.parse(item.configuration)
        return config[key] || defaultValue
      } catch (error) {
        return '配置错误'
      }
    },
    getMqttHost(item) {
      return this.getConfigValue(item, 'host')
    },
    getMqttPort(item) {
      return this.getConfigValue(item, 'port')
    }
  }
}
</script>

<style lang="less" scoped>
.table-page-search-wrapper {
  .table-page-search-submitButtons {
    display: flex;
    align-items: center;
    height: 32px;
  }
}

.table-operations {
  margin-bottom: 16px;
  display: flex;
  justify-content: space-between;
  align-items: center;

  .refresh-btn {
    margin-left: auto;
  }
}

.type-cell {
  display: flex;
  align-items: center;
}

.network-card {
  margin-bottom: 16px;
  border-radius: 8px;
  box-shadow: 0 2px 8px #f0f1f2;
  transition: box-shadow 0.2s;
}

.network-card:hover {
  box-shadow: 0 4px 16px #e6f7ff;
}

.network-card.card-enabled {
  border-left: 4px solid #52c41a;
  background: linear-gradient(135deg, #f6ffed 0%, #ffffff 100%);
}

.network-card.card-disabled {
  border-left: 4px solid #ff4d4f;
  background: linear-gradient(135deg, #fff2f0 0%, #ffffff 100%);
  opacity: 0.8;
}

.card-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  font-size: 16px;
  font-weight: bold;
  margin-bottom: 8px;
}

.status-indicator {
  display: flex;
  align-items: center;
  gap: 4px;
}

.card-title {
  margin-left: 8px;
  cursor: pointer;
  color: #1890ff;
  transition: color 0.2s;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  display: block;
  flex: 1;
}

.card-title:hover {
  color: #40a9ff;
}

.card-body {
  margin: 12px 0 0 0;
}

.card-row {
  font-size: 13px;
  color: #666;
  margin-bottom: 4px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.card-actions {
  display: flex;
  gap: 4px;
  margin-top: 12px;
  padding-top: 12px;
  border-top: 1px solid #f0f0f0;
  flex-wrap: nowrap;
  justify-content: flex-start;
  overflow: hidden;
}

.ant-btn-link {
  padding: 2px 6px;
  font-size: 14px;
  height: auto;
  border-radius: 4px;
  transition: all 0.2s;
}

.ant-btn-link.running {
  color: #52c41a;
}

.ant-btn-link.delete-btn {
  color: #ff4d4f;
}

.ant-btn-link:hover {
  background: #f5f5f5;
  transform: scale(1.05);
}

// 响应式设计
@media (max-width: 768px) {
  .network-card {
    margin-bottom: 12px;
  }

  .card-actions {
    gap: 6px;

    .ant-btn-link {
      padding: 6px 10px;
    }
  }
}

.breath-badge {
  animation: breath-scale 1.2s infinite ease-in-out;
}

@keyframes breath-scale {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.5);
  }
  100% {
    transform: scale(1);
  }
}
</style>
