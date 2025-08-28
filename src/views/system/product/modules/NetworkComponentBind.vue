<template>
  <div class="network-component-bind">
    <!-- 未绑定状态 -->
    <div v-if="!product.networkUnionId" class="unbind-state">
      <div class="state-header">
        <a-alert
          message="网络组件关联"
          description="默认使用内置MQTT，已有设备时，请谨慎操作！！！"
          type="warning"
          show-icon
          style="margin-bottom: 20px"
        />
      </div>

      <!-- 组件列表 -->
      <div class="component-list">
        <div class="list-header">
          <h3>{{ getPlatformTitle() }}组件列表</h3>
          <a-button
            type="primary"
            icon="reload"
            size="small"
            @click="loadComponentList"
            :loading="loading"
          >
            刷新列表
          </a-button>
        </div>

        <a-spin :spinning="loading">
          <div v-if="componentList.length === 0" class="empty-state">
            <a-empty
              description="暂无可用的网络组件"
              :image="emptyImage"
            >
              <a-button type="primary" @click="goToCreate">
                创建网络组件
              </a-button>
            </a-empty>
          </div>

          <div v-else class="component-grid">
            <a-card
              v-for="item in componentList"
              :key="item.id"
              hoverable
              class="component-card"
              :class="{ 'selected': selectedComponent && selectedComponent.id === item.id }"
              @click="selectComponent(item)"
            >
              <div class="card-header">
                <div class="component-type">
                  <a-icon
                    :type="getComponentIcon(item.type)"
                    :style="{ color: getComponentColor(item.type) }"
                  />
                  <span class="type-label">{{ getComponentTypeLabel(item.type) }}</span>
                </div>
                <a-badge
                  :status="item.running ? 'success' : 'default'"
                  :class="{ 'breath-badge': item.running }"
                />
              </div>

              <div class="card-body">
                <div class="component-name">{{ item.name }}</div>
                <div class="component-info">
                  <div class="info-item">
                    <a-icon type="key"/>
                    <span>标识：{{ item.unionId }}</span>
                  </div>
                  <div class="info-item">
                    <a-icon type="disconnect"/>
                    <span>状态：{{ item.running ? '运行中' : '已停止' }}</span>
                  </div>
                  <div class="info-item" v-if="getComponentHost(item)">
                    <a-icon type="global"/>
                    <span>地址：{{ getComponentHost(item) }}</span>
                  </div>
                </div>
              </div>

              <div class="card-actions">
                <a-button
                  type="primary"
                  size="small"
                  @click.stop="bindComponent(item)"
                  :loading="binding"
                >
                  绑定此组件
                </a-button>
              </div>
            </a-card>
          </div>
        </a-spin>
      </div>
    </div>

    <!-- 已绑定状态 -->
    <div v-else class="bind-state">
      <div class="state-header">
        <a-alert
          message="已绑定网络组件"
          description="当前产品已绑定网络组件，可以查看组件信息或解绑"
          type="success"
          show-icon
          style="margin-bottom: 20px"
        />
      </div>

      <div class="bound-component">
        <a-spin :spinning="loading">
          <div v-if="!boundComponent" class="loading-bound">
            <a-empty description="正在加载绑定组件信息..."/>
          </div>

          <a-card v-else class="bound-card">
            <div class="card-header">
              <div class="component-type">
                <a-icon
                  :type="getComponentIcon(boundComponent.type)"
                  :style="{ color: getComponentColor(boundComponent.type) }"
                />
                <span class="type-label">{{ getComponentTypeLabel(boundComponent.type) }}</span>
              </div>
              <a-badge
                :status="boundComponent.running ? 'success' : 'default'"
                :class="{ 'breath-badge': boundComponent.running }"
              />
            </div>

            <div class="card-body">
              <div class="component-name">{{ boundComponent.name }}</div>
              <div class="component-info">
                <div class="info-item">
                  <a-icon type="key"/>
                  <span>标识：{{ boundComponent.unionId }}</span>
                </div>
                <div class="info-item">
                  <a-icon type="disconnect"/>
                  <span>状态：{{ boundComponent.running ? '运行中' : '已停止' }}</span>
                </div>
                <div class="info-item" v-if="getComponentHost(boundComponent)">
                  <a-icon type="global"/>
                  <span>地址：{{ getComponentHost(boundComponent) }}</span>
                </div>
                <!-- <div class="info-item">
                  <a-icon type="clock-circle" />
                  <span>绑定时间：{{ formatTime(boundComponent.createTime) }}</span>
                </div> -->
              </div>

              <!-- 通信主题格式 -->
              <div class="topic-section">
                <div class="topic-header">
                  <a-icon type="message" style="color: #1890ff; margin-right: 6px;"/>
                  <span class="topic-title">通信信息</span>
                  <a-tooltip title="${productKey} 和 ${deviceId} 分别替换为产品Key和设备ID">
                    <a-icon type="question-circle"
                            style="margin-left: 8px; color: #999; cursor: pointer;"/>
                  </a-tooltip>
                </div>

                <!-- MQTT主题 -->
                <div v-if="product.thirdPlatform === 'mqtt'" class="topic-content">
                  <div class="topic-category">
                    <a-icon type="appstore" style="color: #1890ff; margin-right: 4px;"/>
                    <span class="category-title">物模型主题</span>
                  </div>
                  <div class="topic-list">
                    <div class="topic-item">
                      <span class="topic-label">属性上报：</span>
                      <span class="topic-value">$thing/up/property/${productKey}/${deviceId}</span>
                    </div>
                    <div class="topic-item">
                      <span class="topic-label">事件上报：</span>
                      <span class="topic-value">$thing/up/event/${productKey}/${deviceId}</span>
                    </div>
                    <div class="topic-item">
                      <span class="topic-label">指令发布：</span>
                      <span class="topic-value">$thing/down/${productKey}/${deviceId}</span>
                    </div>
                  </div>

                  <div class="topic-category">
                    <a-icon type="swap" style="color: #fa8c16; margin-right: 4px;"/>
                    <span class="category-title">透传主题</span>
                  </div>
                  <div class="topic-list">
                    <div class="topic-item">
                      <span class="topic-label">数据上报：</span>
                      <span class="topic-value">$thing/up/${productKey}/${deviceId}</span>
                    </div>
                    <div class="topic-item">
                      <span class="topic-label">指令发布：</span>
                      <span class="topic-value">$thing/down/${productKey}/${deviceId}</span>
                    </div>
                  </div>
                  <div class="topic-category">
                    <a-icon type="setting" style="color: #52c41a; margin-right: 4px;"/>
                    <span class="category-title">系统级别主题（未实现）</span>
                  </div>
                  <div class="topic-list">
                    <div class="topic-item">
                      <span class="topic-label">OTA上报：</span>
                      <span class="topic-value">$ota/report/${productKey}/${deviceId}</span>
                    </div>
                    <div class="topic-item">
                      <span class="topic-label">OTA下发：</span>
                      <span class="topic-value">$ota/update/${productKey}/${deviceId}</span>
                    </div>
                  </div>

                </div>
                <!-- TCP主题 -->
                <div v-if="product.thirdPlatform === 'tcp'" class="topic-content">
                  <!-- <div class="topic-category">
                    <a-icon type="code" style="color: #fa8c16; margin-right: 4px;" />
                    <span class="category-title">数据格式</span>
                  </div> -->
                  <div class="topic-list">
                    <div class="topic-item">
                      <span class="topic-label">数据类型：</span>
                      <span class="topic-value">{{
                          getConfigValue(boundComponent, 'decodeType')
                        }}</span>
                    </div>
                    <div class="topic-item">
                      <span class="topic-label">解析类型：</span>
                      <span class="topic-value">{{
                          getConfigValue(boundComponent, 'parserType')
                        }}</span>
                    </div>
                    <div class="topic-item">
                      <span class="topic-label">是否预解码出deviceId：</span>
                      <span class="topic-value">{{
                          getConfigValue(boundComponent, 'alwaysPreDecode')
                        }}</span>
                    </div>
                  </div>
                  <!--
                                    <div class="topic-category">
                                      <a-icon type="user" style="color: #1890ff; margin-right: 4px;" />
                                      <span class="category-title">设备标识</span>
                                    </div>
                                    <div class="topic-list">
                                      <div class="topic-item">
                                        <span class="topic-label">连接标识：</span>
                                        <span class="topic-value">${productKey}/${deviceId}</span>
                                      </div>
                                    </div> -->
                </div>
              </div>
            </div>
            <div class="card-actions">
              <a-space>
                <a-button
                  type="default"
                  size="small"
                  @click="viewComponentDetail"
                >
                  查看详情
                </a-button>
                <a-button
                  v-if="product.thirdPlatform === 'mqtt'"
                  type="danger"
                  size="small"
                  @click="unbindComponent"
                  :loading="unbinding"
                >
                  解绑组件
                </a-button>
              </a-space>
            </div>
          </a-card>
        </a-spin>
      </div>
    </div>
  </div>
</template>

<script>
import {listNetwork} from '@/api/system/network'
import {updateProductNetworkUnionId} from '@/api/system/dev/product'
import {parseTime} from '@/utils/ruoyi'

export default {
  name: 'NetworkComponentBind',
  props: {
    product: {
      type: Object,
      required: true
    }
  },
  data() {
    return {
      loading: false,
      binding: false,
      unbinding: false,
      componentList: [],
      selectedComponent: null,
      boundComponent: null,
      emptyImage: 'data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iNjQiIGhlaWdodD0iNDEiIHZpZXdCb3g9IjAgMCA2NCA0MSIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj4KPHBhdGggZD0iTTMyIDQwQzQ0LjA5MDkgNDAgNTQgMzAuMDkwOSA1NCAxOEM1NCA1LjkwOTEgNDQuMDkwOSAtNCAzMiAtNEMxOS45MDkxIC00IDEwIDUuOTA5MSAxMCAxOEMxMCAzMC4wOTA5IDE5LjkwOTEgNDAgMzIgNDBaIiBmaWxsPSIjRjVGNUY1Ii8+CjxwYXRoIGQ9Ik0zMiAzNkM0Mi4wNTg5IDM2IDUwIDI4LjA1ODkgNTAgMThDNTAgNy45NDExIDQyLjA1ODkgMCAzMiAwQzIxLjk0MTEgMCAxNCA3Ljk0MTEgMTQgMThDMTQgMjguMDU4OSAyMS45NDExIDM2IDMyIDM2WiIgZmlsbD0iI0YwRjBGMCIvPgo8cGF0aCBkPSJNMzIgMjhDMzYuNDE4MyAyOCA0MCAyNC40MTgzIDQwIDIwQzQwIDE1LjU4MTcgMzYuNDE4MyAxMiAzMiAxMkMyNy41ODE3IDEyIDI0IDE1LjU4MTcgMjQgMjBDMjQgMjQuNDE4MyAyNy41ODE3IDI4IDMyIDI4WiIgZmlsbD0iI0YwRjBGMCIvPgo8L3N2Zz4K'
    }
  },
  watch: {
    product: {
      handler(newProduct) {
        if (newProduct.networkUnionId) {
          // 如果已绑定，加载已绑定组件信息
          this.loadBoundComponent()
        } else {
          // 如果未绑定，清空已绑定组件并加载组件列表
          this.boundComponent = null
          this.loadComponentList()
        }
      },
      immediate: true
    }
  },
  created() {
    // 如果已绑定，只加载已绑定组件信息
    if (this.product.networkUnionId) {
      this.loadBoundComponent()
    } else {
      // 如果未绑定，加载组件列表
      this.loadComponentList()
    }
  },
  methods: {
    parseTime,
    /** 获取平台标题 */
    getPlatformTitle() {
      if (this.product.thirdPlatform === 'tcp') {
        return 'TCP'
      }
      if (this.product.thirdPlatform === 'mqtt') {
        return 'MQTT'
      }
      return '网络'
    },
    /** 获取组件图标 */
    getComponentIcon(type) {
      if (type.includes('TCP_CLIENT')) {
        return 'user'
      }
      if (type.includes('TCP_SERVER')) {
        return 'cloud-server'
      }
      if (type.includes('MQTT_CLIENT')) {
        return 'cloud'
      }
      if (type.includes('MQTT_SERVER')) {
        return 'cloud-server'
      }
      return 'api'
    },
    /** 获取组件颜色 */
    getComponentColor(type) {
      if (type.includes('TCP_CLIENT')) {
        return '#1890ff'
      }
      if (type.includes('TCP_SERVER')) {
        return '#52c41a'
      }
      if (type.includes('MQTT_CLIENT')) {
        return '#fa8c16'
      }
      if (type.includes('MQTT_SERVER')) {
        return '#eb2f96'
      }
      return '#666'
    },
    /** 获取组件类型标签 */
    getComponentTypeLabel(type) {
      if (type === 'TCP_CLIENT') {
        return 'TCP客户端'
      }
      if (type === 'TCP_SERVER') {
        return 'TCP服务端'
      }
      if (type === 'MQTT_CLIENT') {
        return 'MQTT客户端'
      }
      if (type === 'MQTT_SERVER') {
        return 'MQTT服务端'
      }
      return type
    },
    /** 通用配置值获取方法 */
    getConfigValue(component, key, defaultValue = null) {
      try {
        const config = JSON.parse(component.configuration)
        return config[key] || defaultValue
      } catch (error) {
        return null
      }
    },
    /** 获取组件主机地址 */
    getComponentHost(component) {
      const host = this.getConfigValue(component, 'host')
      const port = this.getConfigValue(component, 'port')

      if (host) {
        if (port) {
          return `${host}:${port}`
        }
        return host
      }
      return null
    },
    /** 格式化时间 */
    formatTime(time) {
      if (!time) {
        return '-'
      }
      return this.parseTime(time)
    },
    /** 加载组件列表 */
    async loadComponentList() {
      this.loading = true
      try {
        const types = this.getComponentTypes()
        const response = await listNetwork({
          pageNum: 1,
          pageSize: 100,
          type: types
        })
        this.componentList = response.rows || []
      } catch (error) {
        this.$message.error('加载组件列表失败：' + (error.message || error))
      } finally {
        this.loading = false
      }
    },
    /** 获取组件类型 */
    getComponentTypes() {
      if (this.product.thirdPlatform === 'tcp') {
        return ['TCP_CLIENT', 'TCP_SERVER']
      }
      if (this.product.thirdPlatform === 'mqtt') {
        return ['MQTT_CLIENT', 'MQTT_SERVER']
      }
      return []
    },
    /** 选择组件 */
    selectComponent(component) {
      this.selectedComponent = component
    },
    /** 绑定组件 */
    async bindComponent(component) {
      this.binding = true
      try {
        await updateProductNetworkUnionId({
          id: this.product.id,
          networkUnionId: component.unionId
        })
        this.$message.success('绑定成功')
        this.$emit('refresh')
      } catch (error) {
        this.$message.error('绑定失败：' + (error.message || error))
      } finally {
        this.binding = false
      }
    },
    /** 解绑组件 */
    async unbindComponent() {
      this.$confirm({
        title: '确认解绑',
        content: '确定要解绑当前网络组件吗？解绑后产品将无法使用网络通信功能。',
        onOk: async () => {
          this.unbinding = true
          try {
            await updateProductNetworkUnionId({
              id: this.product.id,
              networkUnionId: ''
            })
            this.$message.success('解绑成功')
            this.$emit('refresh')
          } catch (error) {
            this.$message.error('解绑失败：' + (error.message || error))
          } finally {
            this.unbinding = false
          }
        }
      })
    },
    /** 查看组件详情 */
    viewComponentDetail() {
      if (!this.boundComponent) {
        return
      }

      const route = this.product.thirdPlatform === 'tcp'
        ? `/system/network/tcp/detail/${this.boundComponent.id}`
        : `/system/network/mqtt/detail/${this.boundComponent.id}`

      this.$router.push(route)
    },
    /** 跳转到创建页面 */
    goToCreate() {
      const route = this.product.thirdPlatform === 'tcp'
        ? '/system/network/tcp'
        : '/system/network/mqtt'

      this.$router.push(route)
    },
    /** 加载已绑定组件信息 */
    async loadBoundComponent() {
      if (!this.product.networkUnionId) {
        this.boundComponent = null
        return
      }

      this.loading = true
      try {
        // 直接查询已绑定的组件信息
        const response = await listNetwork({
          pageNum: 1,
          pageSize: 100,
          unionId: this.product.networkUnionId
        })

        if (response.rows && response.rows.length > 0) {
          this.boundComponent = response.rows[0]
        } else {
          // 如果没找到，尝试从组件列表中查找
          if (this.componentList.length > 0) {
            this.boundComponent = this.componentList.find(
              item => item.unionId === this.product.networkUnionId
            )
          }
        }
      } catch (error) {
        console.error('加载已绑定组件失败：', error)
        this.$message.error('加载已绑定组件信息失败')
      } finally {
        this.loading = false
      }
    }
  }
}
</script>

<style lang="less" scoped>
.network-component-bind {
  .state-header {
    margin-bottom: 20px;
  }

  .component-list {
    .list-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 16px;

      h3 {
        margin: 0;
        font-size: 16px;
        font-weight: 600;
        color: #1a202c;
      }
    }

    .empty-state {
      padding: 40px 0;
      text-align: center;
    }

    .component-grid {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
      gap: 16px;

      .component-card {
        border-radius: 8px;
        transition: all 0.2s ease;
        cursor: pointer;

        &:hover {
          transform: translateY(-2px);
          box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
        }

        &.selected {
          border-color: #1890ff;
          box-shadow: 0 0 0 2px rgba(24, 144, 255, 0.2);
        }

        .card-header {
          display: flex;
          justify-content: space-between;
          align-items: center;
          margin-bottom: 12px;

          .component-type {
            display: flex;
            align-items: center;
            gap: 8px;

            .type-label {
              font-size: 12px;
              font-weight: 500;
              color: #666;
            }
          }
        }

        .card-body {
          .component-name {
            font-size: 16px;
            font-weight: 600;
            color: #1a202c;
            margin-bottom: 8px;
          }

          .component-info {
            .info-item {
              display: flex;
              align-items: center;
              gap: 6px;
              font-size: 13px;
              color: #666;
              margin-bottom: 4px;

              .anticon {
                font-size: 12px;
              }
            }
          }
        }

        .card-actions {
          margin-top: 12px;
          padding-top: 12px;
          border-top: 1px solid #f0f0f0;
        }
      }
    }
  }

  .bound-component {
    .loading-bound {
      padding: 40px 0;
      text-align: center;
    }

    .bound-card {
      border-radius: 8px;
      border: 2px solid #52c41a;
      background: linear-gradient(135deg, #f6ffed 0%, #ffffff 100%);

      .card-header {
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin-bottom: 12px;

        .component-type {
          display: flex;
          align-items: center;
          gap: 8px;

          .type-label {
            font-size: 14px;
            font-weight: 600;
            color: #52c41a;
          }
        }
      }

      .card-body {
        .component-name {
          font-size: 18px;
          font-weight: 600;
          color: #1a202c;
          margin-bottom: 12px;
        }

        .component-info {
          .info-item {
            display: flex;
            align-items: center;
            gap: 8px;
            font-size: 14px;
            color: #666;
            margin-bottom: 6px;

            .anticon {
              font-size: 14px;
            }
          }
        }

        /* 主题格式样式 */

        .topic-section {
          margin-top: 16px;
          padding-top: 16px;
          border-top: 1px solid #f0f0f0;

          .topic-header {
            display: flex;
            align-items: center;
            margin-bottom: 12px;

            .topic-title {
              font-size: 14px;
              font-weight: 600;
              color: #333;
            }
          }

          .topic-content {
            .topic-category {
              display: flex;
              align-items: center;
              margin-bottom: 8px;
              margin-top: 12px;

              .category-title {
                font-size: 13px;
                font-weight: 500;
                color: #333;
              }
            }

            .topic-list {
              .topic-item {
                display: flex;
                align-items: flex-start;
                margin-bottom: 6px;
                font-size: 12px;

                .topic-label {
                  color: #666;
                  min-width: 80px;
                  margin-right: 8px;
                  flex-shrink: 0;
                }

                .topic-value {
                  color: #1890ff;
                  font-family: 'Monaco', 'Menlo', 'Ubuntu Mono', monospace;
                  background: #f8f9fa;
                  padding: 2px 6px;
                  border-radius: 3px;
                  border: 1px solid #e8e8e8;
                  word-break: break-all;
                  flex: 1;
                  font-size: 11px;
                  line-height: 1.4;
                }
              }
            }
          }
        }
      }

      .card-actions {
        margin-top: 16px;
        padding-top: 16px;
        border-top: 1px solid #f0f0f0;
      }
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
    transform: scale(1.2);
  }
  100% {
    transform: scale(1);
  }
}

// 响应式设计
@media (max-width: 768px) {
  .network-component-bind {
    .component-grid {
      grid-template-columns: 1fr;
      gap: 12px;
    }

    .component-card {
      .card-body {
        .component-name {
          font-size: 14px;
        }

        .component-info {
          .info-item {
            font-size: 12px;
          }
        }
      }
    }
  }
}
</style> 