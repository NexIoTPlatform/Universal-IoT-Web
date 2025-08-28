<template>
  <page-header-wrapper>
    <a-card :bordered="false" v-show="show">
      <div class="app-container">
        <!-- 页面头部 -->
        <div class="page-header">
          <a-button type="link" icon="left" @click="back()" class="back-btn">
            返回应用列表
          </a-button>
          <div class="app-title">
            <div class="app-icon">
              <a-icon type="appstore"/>
            </div>
            <div class="app-info">
              <h2 class="app-name">{{ application.appName }}</h2>
              <div class="app-meta">
                <a-tag :color="application.appStatus === 0 ? 'green' : 'red'">
                  {{ application.appStatus === 0 ? '启用' : '禁用' }}
                </a-tag>
                <span class="app-id">AppID: {{ application.appId }}</span>
              </div>
            </div>
          </div>
        </div>

        <!-- 标签页内容 -->
        <a-tabs v-model="activeKey" class="detail-tabs">
          <!-- 应用信息标签页 -->
          <a-tab-pane key="1" tab="应用信息">
            <div class="tab-content">

              <!-- 基本信息卡片 -->
              <a-card title="基本信息" class="info-card" :bordered="false">
                <template slot="extra">
                  <a-button
                    type="link"
                    @click="$refs.createForm.handleUpdate(undefined, application.appUniqueId)"
                    v-hasPermi="['application:application:edit']"
                  >
                    <a-icon type="edit"/>
                    编辑
                  </a-button>
                </template>

                <a-row :gutter="[24, 16]">
                  <a-col :span="12">
                    <div class="info-item">
                      <div class="info-label">应用名称</div>
                      <div class="info-value">{{ application.appName }}</div>
                    </div>
                  </a-col>
                  <!-- <a-col :span="12">
                    <div class="info-item">
                      <div class="info-label">实例名称</div>
                      <div class="info-value">{{ application.instance || '-' }}</div>
                    </div>
                  </a-col> -->
                  <a-col :span="12">
                    <div class="info-item">
                      <div class="info-label">创建时间</div>
                      <div class="info-value">{{ formatDateTime(application.createDate) }}</div>
                    </div>
                  </a-col>
                  <a-col :span="12">
                    <div class="info-item">
                      <div class="info-label">创建账号</div>
                      <div class="info-value">{{ application.unionId }}</div>
                    </div>
                  </a-col>
                  <a-col :span="24">
                    <div class="info-item">
                      <div class="info-label">应用描述</div>
                      <div class="info-value">
                        {{ application.remark || '暂无描述' }}
                      </div>
                    </div>
                  </a-col>
                </a-row>
              </a-card>

              <!-- 开发信息卡片 -->
              <a-card title="开发信息" class="info-card" :bordered="false">
                <template slot="extra">
                  <a-button
                    type="link"
                    @click="showModal"
                    v-hasPermi="['application:application:resetSecret']"
                  >
                    <a-icon type="key"/>
                    重置密钥
                  </a-button>
                </template>

                <a-row :gutter="[24, 16]">
                  <a-col :span="12">
                    <div class="info-item">
                      <div class="info-label">Client ID</div>
                      <div class="info-value code-value">
                        {{ application.appId }}
                        <a-button
                          type="link"
                          size="small"
                          @click="copyToClipboard(application.appId)"
                          class="copy-btn"
                        >
                          <a-icon type="copy"/>
                        </a-button>
                      </div>
                    </div>
                  </a-col>
                  <a-col :span="12">
                    <div class="info-item">
                      <div class="info-label">Client Secret</div>
                      <div class="info-value code-value">
                        <span v-if="showAppSecret">{{ application.appSecret }}</span>
                        <span v-else>********************************</span>
                        <a-button
                          type="link"
                          size="small"
                          @click="appSecretShow"
                          class="copy-btn"
                        >
                          <a-icon :type="showAppSecret ? 'eye-invisible' : 'eye'"/>
                        </a-button>
                        <a-button
                          v-if="showAppSecret"
                          type="link"
                          size="small"
                          @click="copyToClipboard(application.appSecret)"
                          class="copy-btn"
                        >
                          <a-icon type="copy"/>
                        </a-button>
                      </div>
                    </div>
                  </a-col>
                </a-row>
              </a-card>

              <!-- 推送配置卡片 -->
              <a-card title="推送配置" class="info-card" :bordered="false">
                <template slot="extra">
                  <a-button
                    type="link"
                    @click="$refs.createForm.handleDevelopUpdate(undefined, application.appUniqueId, application.appId, unionId)"
                    v-hasPermi="['application:application:edit']"
                  >
                    <a-icon type="edit"/>
                    编辑
                  </a-button>
                </template>

                <!-- 推送方式折叠面板 -->
                <a-collapse v-model="activeCollapseKeys" accordion>

                  <!-- HTTP推送配置 -->
                  <a-collapse-panel key="http" v-if="getConfig('http').support !== undefined">
                    <template slot="header">
                      <span class="protocol-header-title">
                        <a-icon type="global"/> HTTP推送
                      </span>
                      <a-switch
                        :checked="getConfig('http').enable"
                        :disabled="!getConfig('http').support"
                        @change="onHttpEnableChange"
                        class="protocol-switch"
                        size="small"
                      />
                    </template>
                    <div class="protocol-config">
                      <template v-if="!getConfig('http').support">
                        <div class="protocol-dev">该推送方式正在开发中</div>
                      </template>
                      <template v-else-if="getConfig('http').enable">
                        <div class="config-item">
                          <div class="config-label">
                            <a-icon type="link"/>
                            推送地址
                          </div>
                          <div class="config-value">
                            <div class="config-input">
                              <span class="url-text">{{ getHttpConfig().url }}</span>
                              <div class="config-actions">
                                <a-tooltip title="复制地址">
                                  <a-button
                                    type="link"
                                    size="small"
                                    @click="copyToClipboard(getHttpConfig().url)"
                                    class="action-btn"
                                  >
                                    <a-icon type="copy"/>
                                  </a-button>
                                </a-tooltip>
                              </div>
                            </div>
                          </div>
                        </div>

                        <div class="config-item">
                          <div class="config-label">
                            <a-icon type="key"/>
                            HTTP推送密钥
                          </div>
                          <div class="config-value">
                            <div class="config-input">
                              <span class="secret-text">
                                <span v-if="showHttpSecret">{{ getHttpConfig().secret }}</span>
                                <span v-else>********************************</span>
                              </span>
                              <div class="config-actions">
                                <a-tooltip title="显示/隐藏密钥">
                                  <a-button
                                    type="link"
                                    size="small"
                                    @click="toggleHttpSecret"
                                    class="action-btn"
                                  >
                                    <a-icon :type="showHttpSecret ? 'eye-invisible' : 'eye'"/>
                                  </a-button>
                                </a-tooltip>
                                <a-tooltip title="复制密钥" v-if="showHttpSecret">
                                  <a-button
                                    type="link"
                                    size="small"
                                    @click="copyToClipboard(getHttpConfig().secret)"
                                    class="action-btn"
                                  >
                                    <a-icon type="copy"/>
                                  </a-button>
                                </a-tooltip>
                              </div>
                            </div>
                          </div>
                        </div>

                        <div class="config-item">
                          <div class="config-label">
                            <a-icon type="tag"/>
                            HTTP消息头
                          </div>
                          <div class="config-value">
                            <div class="config-input">
                              <span class="header-text">{{ getHttpConfig().header }}</span>
                              <div class="config-actions">
                                <a-tooltip title="复制消息头">
                                  <a-button
                                    type="link"
                                    size="small"
                                    @click="copyToClipboard(getHttpConfig().header)"
                                    class="action-btn"
                                  >
                                    <a-icon type="copy"/>
                                  </a-button>
                                </a-tooltip>
                              </div>
                            </div>
                          </div>
                        </div>

                        <div class="config-tips">
                          <a-icon type="info-circle"/>
                          <span>HTTP推送采用POST方法，数据格式为application/json，密钥将在请求头中携带</span>
                        </div>
                      </template>
                      <template v-else>
                        <div class="protocol-empty">未启用</div>
                      </template>
                    </div>
                  </a-collapse-panel>

                  <!-- RocketMQ推送配置 -->
                  <a-collapse-panel key="rocketmq"
                                    v-if="getConfig('rocketmq').support !== undefined">
                    <template slot="header">
                      <span class="protocol-header-title">
                        <a-icon type="cluster"/> RocketMQ推送
                      </span>
                      <a-switch
                        :checked="getConfig('rocketmq').enable"
                        :disabled="!getConfig('rocketmq').support"
                        @change="onRocketMQEnableChange"
                        class="protocol-switch"
                        size="small"
                      />
                    </template>

                    <div class="protocol-config">
                      <template v-if="!getConfig('rocketmq').support">
                        <div class="protocol-dev">该推送方式正在开发中</div>
                      </template>
                      <template v-else-if="getConfig('rocketmq').enable">
                        <div class="config-item">
                          <div class="config-label">
                            <a-icon type="cluster"/>
                            服务器地址
                          </div>
                          <div class="config-value">
                            <div class="config-input">
                              <span class="url-text">{{ getRocketMQConfig().namesrvAddr }}</span>
                              <div class="config-actions">
                                <a-tooltip title="复制地址">
                                  <a-button
                                    type="link"
                                    size="small"
                                    @click="copyToClipboard(getRocketMQConfig().namesrvAddr)"
                                    class="action-btn"
                                  >
                                    <a-icon type="copy"/>
                                  </a-button>
                                </a-tooltip>
                              </div>
                            </div>
                          </div>
                        </div>

                        <div class="config-item">
                          <div class="config-label">
                            <a-icon type="message"/>
                            Topic
                          </div>
                          <div class="config-value">
                            <div class="config-input">
                              <span class="topic-text">{{ getRocketMQConfig().topic }}</span>
                              <div class="config-actions">
                                <a-tooltip title="复制Topic">
                                  <a-button
                                    type="link"
                                    size="small"
                                    @click="copyToClipboard(getRocketMQConfig().topic)"
                                    class="action-btn"
                                  >
                                    <a-icon type="copy"/>
                                  </a-button>
                                </a-tooltip>
                              </div>
                            </div>
                          </div>
                        </div>

                        <div class="config-item">
                          <div class="config-label">
                            <a-icon type="team"/>
                            消费者组
                          </div>
                          <div class="config-value">
                            <div class="config-input">
                              <span class="group-text">{{ getRocketMQConfig().group }}</span>
                              <div class="config-actions">
                                <a-tooltip title="复制消费者组">
                                  <a-button
                                    type="link"
                                    size="small"
                                    @click="copyToClipboard(getRocketMQConfig().group)"
                                    class="action-btn"
                                  >
                                    <a-icon type="copy"/>
                                  </a-button>
                                </a-tooltip>
                              </div>
                            </div>
                          </div>
                        </div>

                        <div class="config-tips">
                          <a-icon type="info-circle"/>
                          <span>RocketMQ消息队列推送，支持高并发消息处理</span>
                        </div>
                      </template>
                      <template v-else>
                        <div class="protocol-empty">未启用</div>
                      </template>
                    </div>
                  </a-collapse-panel>

                  <!-- MQTT推送配置 -->
                  <a-collapse-panel key="mqtt" v-if="getConfig('mqtt').support !== undefined">
                    <template slot="header">
                      <span class="protocol-header-title">
                        <a-icon type="cloud"/> MQTT推送
                      </span>
                      <a-switch
                        :checked="getConfig('mqtt').enable"
                        :disabled="!getConfig('mqtt').support"
                        @change="onMqttEnableChange"
                        class="protocol-switch"
                        size="small"
                      />
                    </template>
                    <div class="protocol-config">
                      <template v-if="!getConfig('mqtt').support">
                        <div class="protocol-dev">该推送方式正在开发中</div>
                      </template>
                      <template v-else-if="getConfig('mqtt').enable">
                        <div class="config-item">
                          <div class="config-label">
                            <a-icon type="cloud"/>
                            服务器地址
                          </div>
                          <div class="config-value">
                            <div class="config-input">
                              <span class="url-text">{{ getMQTTConfig().url }}</span>
                              <div class="config-actions">
                                <a-tooltip title="复制地址">
                                  <a-button
                                    type="link"
                                    size="small"
                                    @click="copyToClipboard(getMQTTConfig().url)"
                                    class="action-btn"
                                  >
                                    <a-icon type="copy"/>
                                  </a-button>
                                </a-tooltip>
                              </div>
                            </div>
                          </div>
                        </div>

                        <div class="config-item">
                          <div class="config-label">
                            <a-icon type="message"/>
                            Topic
                          </div>
                          <div class="config-value">
                            <div class="config-input">
                              <span class="topic-text">{{ getMQTTConfig().topic }}</span>
                              <div class="config-actions">
                                <a-tooltip title="复制Topic">
                                  <a-button
                                    type="link"
                                    size="small"
                                    @click="copyToClipboard(getMQTTConfig().topic)"
                                    class="action-btn"
                                  >
                                    <a-icon type="copy"/>
                                  </a-button>
                                </a-tooltip>
                              </div>
                            </div>
                          </div>
                        </div>

                        <div class="config-item">
                          <div class="config-label">
                            <a-icon type="user"/>
                            用户名
                          </div>
                          <div class="config-value">
                            <div class="config-input">
                              <span class="username-text">{{ getMQTTConfig().username }}</span>
                              <div class="config-actions">
                                <a-tooltip title="复制用户名">
                                  <a-button
                                    type="link"
                                    size="small"
                                    @click="copyToClipboard(getMQTTConfig().username)"
                                    class="action-btn"
                                  >
                                    <a-icon type="copy"/>
                                  </a-button>
                                </a-tooltip>
                              </div>
                            </div>
                          </div>
                        </div>

                        <div class="config-item">
                          <div class="config-label">
                            <a-icon type="key"/>
                            密码
                          </div>
                          <div class="config-value">
                            <div class="config-input">
                              <span class="secret-text">
                                <span v-if="showMQTTSecret">{{ getMQTTConfig().password }}</span>
                                <span v-else>********************************</span>
                              </span>
                              <div class="config-actions">
                                <a-tooltip title="显示/隐藏密码">
                                  <a-button
                                    type="link"
                                    size="small"
                                    @click="toggleMQTTSecret"
                                    class="action-btn"
                                  >
                                    <a-icon :type="showMQTTSecret ? 'eye-invisible' : 'eye'"/>
                                  </a-button>
                                </a-tooltip>
                                <a-tooltip title="复制密码" v-if="showMQTTSecret">
                                  <a-button
                                    type="link"
                                    size="small"
                                    @click="copyToClipboard(getMQTTConfig().password)"
                                    class="action-btn"
                                  >
                                    <a-icon type="copy"/>
                                  </a-button>
                                </a-tooltip>
                              </div>
                            </div>
                          </div>
                        </div>

                        <div class="config-item">
                          <div class="config-label">
                            <a-icon type="idcard"/>
                            客户端ID
                          </div>
                          <div class="config-value">
                            <div class="config-input">
                              <span class="clientid-text">{{ getMQTTConfig().clientId }}</span>
                              <div class="config-actions">
                                <a-tooltip title="复制客户端ID">
                                  <a-button
                                    type="link"
                                    size="small"
                                    @click="copyToClipboard(getMQTTConfig().clientId)"
                                    class="action-btn"
                                  >
                                    <a-icon type="copy"/>
                                  </a-button>
                                </a-tooltip>
                              </div>
                            </div>
                          </div>
                        </div>

                        <div class="config-tips">
                          <a-icon type="info-circle"/>
                          <span>MQTT物联网消息推送，支持设备双向通信</span>
                        </div>
                      </template>
                      <template v-else>
                        <div class="protocol-empty">未启用</div>
                      </template>
                    </div>
                  </a-collapse-panel>

                  <!-- JDBC推送配置 -->
                  <a-collapse-panel key="jdbc" v-if="getConfig('jdbc').support !== undefined">
                    <template slot="header">
                      <span class="protocol-header-title">
                        <a-icon type="database"/> JDBC推送
                      </span>
                      <a-switch
                        :checked="getConfig('jdbc').enable"
                        :disabled="!getConfig('jdbc').support"
                        @change="onJDBCEnableChange"
                        class="protocol-switch"
                        size="small"
                      />
                    </template>

                    <div class="protocol-config">
                      <template v-if="!getConfig('jdbc').support">
                        <div class="protocol-dev">该推送方式正在开发中</div>
                      </template>
                      <template v-else-if="getConfig('jdbc').enable">
                        <!-- MySQL配置 -->
                        <div v-if="getJDBCConfig().mysql" class="database-config">
                          <div class="db-header">
                            <a-tag color="blue">MySQL</a-tag>
                          </div>
                          <div class="config-item">
                            <div class="config-label">
                              <a-icon type="database"/>
                              连接地址
                            </div>
                            <div class="config-value">
                              <div class="config-input">
                                <span class="url-text">{{ getJDBCConfig().mysql.url }}</span>
                                <div class="config-actions">
                                  <a-tooltip title="复制连接地址">
                                    <a-button
                                      type="link"
                                      size="small"
                                      @click="copyToClipboard(getJDBCConfig().mysql.url)"
                                      class="action-btn"
                                    >
                                      <a-icon type="copy"/>
                                    </a-button>
                                  </a-tooltip>
                                </div>
                              </div>
                            </div>
                          </div>
                          <div class="config-item">
                            <div class="config-label">
                              <a-icon type="user"/>
                              用户名
                            </div>
                            <div class="config-value">
                              <div class="config-input">
                                <span class="username-text">{{
                                    getJDBCConfig().mysql.username
                                  }}</span>
                                <div class="config-actions">
                                  <a-tooltip title="复制用户名">
                                    <a-button
                                      type="link"
                                      size="small"
                                      @click="copyToClipboard(getJDBCConfig().mysql.username)"
                                      class="action-btn"
                                    >
                                      <a-icon type="copy"/>
                                    </a-button>
                                  </a-tooltip>
                                </div>
                              </div>
                            </div>
                          </div>
                        </div>

                        <!-- Oracle配置 -->
                        <div v-if="getJDBCConfig().oracle" class="database-config">
                          <div class="db-header">
                            <a-tag color="red">Oracle</a-tag>
                          </div>
                          <div class="config-item">
                            <div class="config-label">
                              <a-icon type="database"/>
                              连接地址
                            </div>
                            <div class="config-value">
                              <div class="config-input">
                                <span class="url-text">{{ getJDBCConfig().oracle.url }}</span>
                                <div class="config-actions">
                                  <a-tooltip title="复制连接地址">
                                    <a-button
                                      type="link"
                                      size="small"
                                      @click="copyToClipboard(getJDBCConfig().oracle.url)"
                                      class="action-btn"
                                    >
                                      <a-icon type="copy"/>
                                    </a-button>
                                  </a-tooltip>
                                </div>
                              </div>
                            </div>
                          </div>
                          <div class="config-item">
                            <div class="config-label">
                              <a-icon type="user"/>
                              用户名
                            </div>
                            <div class="config-value">
                              <div class="config-input">
                                <span class="username-text">{{
                                    getJDBCConfig().oracle.username
                                  }}</span>
                                <div class="config-actions">
                                  <a-tooltip title="复制用户名">
                                    <a-button
                                      type="link"
                                      size="small"
                                      @click="copyToClipboard(getJDBCConfig().oracle.username)"
                                      class="action-btn"
                                    >
                                      <a-icon type="copy"/>
                                    </a-button>
                                  </a-tooltip>
                                </div>
                              </div>
                            </div>
                          </div>
                        </div>

                        <!-- PostgreSQL配置 -->
                        <div v-if="getJDBCConfig().postgres" class="database-config">
                          <div class="db-header">
                            <a-tag color="green">PostgreSQL</a-tag>
                          </div>
                          <div class="config-item">
                            <div class="config-label">
                              <a-icon type="database"/>
                              连接地址
                            </div>
                            <div class="config-value">
                              <div class="config-input">
                                <span class="url-text">{{ getJDBCConfig().postgres.url }}</span>
                                <div class="config-actions">
                                  <a-tooltip title="复制连接地址">
                                    <a-button
                                      type="link"
                                      size="small"
                                      @click="copyToClipboard(getJDBCConfig().postgres.url)"
                                      class="action-btn"
                                    >
                                      <a-icon type="copy"/>
                                    </a-button>
                                  </a-tooltip>
                                </div>
                              </div>
                            </div>
                          </div>
                          <div class="config-item">
                            <div class="config-label">
                              <a-icon type="user"/>
                              用户名
                            </div>
                            <div class="config-value">
                              <div class="config-input">
                                <span class="username-text">{{
                                    getJDBCConfig().postgres.username
                                  }}</span>
                                <div class="config-actions">
                                  <a-tooltip title="复制用户名">
                                    <a-button
                                      type="link"
                                      size="small"
                                      @click="copyToClipboard(getJDBCConfig().postgres.username)"
                                      class="action-btn"
                                    >
                                      <a-icon type="copy"/>
                                    </a-button>
                                  </a-tooltip>
                                </div>
                              </div>
                            </div>
                          </div>
                        </div>

                        <!-- SQLite配置 -->
                        <div v-if="getJDBCConfig().sqlite" class="database-config">
                          <div class="db-header">
                            <a-tag color="orange">SQLite</a-tag>
                          </div>
                          <div class="config-item">
                            <div class="config-label">
                              <a-icon type="database"/>
                              数据库路径
                            </div>
                            <div class="config-value">
                              <div class="config-input">
                                <span class="url-text">{{ getJDBCConfig().sqlite.url }}</span>
                                <div class="config-actions">
                                  <a-tooltip title="复制数据库路径">
                                    <a-button
                                      type="link"
                                      size="small"
                                      @click="copyToClipboard(getJDBCConfig().sqlite.url)"
                                      class="action-btn"
                                    >
                                      <a-icon type="copy"/>
                                    </a-button>
                                  </a-tooltip>
                                </div>
                              </div>
                            </div>
                          </div>
                        </div>

                        <div class="config-tips">
                          <a-icon type="info-circle"/>
                          <span>JDBC数据库推送，支持多种数据库类型</span>
                        </div>
                      </template>
                      <template v-else>
                        <div class="protocol-empty">未启用</div>
                      </template>
                    </div>
                  </a-collapse-panel>

                </a-collapse>

                <!-- 未配置状态 -->
                <div v-if="!hasAnyConfig()" class="protocol-empty">
                  <div class="empty-content">
                    <a-icon type="disconnect" class="empty-icon"/>
                    <div class="empty-text">未配置推送方式</div>
                    <a-button type="primary" size="small"
                              @click="$refs.createForm.handleDevelopUpdate(undefined, application.appUniqueId, application.appId, unionId)">
                      <a-icon type="plus"/>
                      配置推送
                    </a-button>
                  </div>
                </div>
              </a-card>
            </div>
          </a-tab-pane>

          <!-- 设备管理标签页 -->
          <a-tab-pane key="2" tab="设备管理">
            <device-bind ref="deviceBind" :appUniqueId="appUniqueId"/>
          </a-tab-pane>
        </a-tabs>

        <!-- 重置密钥确认弹窗 -->
        <a-modal
          v-model="modalVisible"
          title="重置密钥确认"
          @ok="handleReset(application)"
          @cancel="modalCancel"
          width="500px"
        >
          <div class="reset-confirm">
            <a-icon type="exclamation-circle"
                    style="color: #faad14; font-size: 22px; margin-right: 16px;"/>
            <div class="reset-content">
              <h4>确认重置应用密钥？</h4>
              <p class="reset-warning">此操作将使当前密钥失效，请输入应用ID确认：</p>
              <div class="confirm-input">
                <span class="confirm-code">{{ application.appId }}</span>
                <a-input
                  v-model="checkInfo"
                  placeholder="请输入上方应用ID"
                  @paste.native.capture.prevent="handlePaste"
                  style="margin-top: 8px"
                />
              </div>
            </div>
          </div>
        </a-modal>

        <!-- 表单组件 -->
        <create-form
          ref="createForm"
          :statusOptions="statusOptions"
          :deleteOptions="[]"
          @ok="getList"
        />
      </div>
    </a-card>
  </page-header-wrapper>
</template>

<script>
import DeviceBind from '@/views/application/modules/DeviceBind'
import {enable, getApplication, resetSecret} from '@/api/application/application'
import CreateForm from '@/views/application/modules/CreateForm'

export default {
  name: 'ApplicationDetails',
  components: {
    DeviceBind,
    CreateForm
  },
  data() {
    return {
      unionId: null,
      appUniqueId: null,
      show: false,
      application: {},
      loading: false,
      statusOptions: [],
      activeKey: '1',
      showAppSecret: false,
      checkInfo: undefined,
      modalVisible: false,
      httpEnable: false,
      openEnable: false,
      showHttpSecret: false,
      showRocketMQSecret: false,
      showMQTTSecret: false,
      httpConfig: null,
      activeCollapseKeys: ['http'],
      form: {
        appName: null,
        appUniqueId: null,
        appId: null,
        upTopic: null,
        downTopic: null,
        notifyUrl: null,
        appSecret: null,
        validEndDate: null,
        scope: null,
        appStatus: '0',
        instance: null,
        remark: null,
        createDate: null,
        deleted: null
      }
    }
  },
  created() {
    this.appUniqueId = this.$route.params.appUniqueId
    this.getList()
  },
  methods: {
    /** 获取应用详情 */
    getList() {
      this.loading = true
      getApplication(this.appUniqueId).then(response => {
        this.application = response.data
        this.unionId = response.data.unionId
        this.httpEnable = response.data.httpEnable

        // 解析配置JSON
        try {
          this.httpConfig = response.data.cfg ? JSON.parse(response.data.cfg) : null
        } catch (error) {
          console.error('解析配置JSON失败:', error)
          this.httpConfig = null
        }

        // 设置默认展开的面板
        this.setDefaultActiveCollapse()

        this.loading = false
        this.show = true
      }).catch(() => {
        this.$message.error('获取应用信息失败')
        this.back()
      })

      this.getDicts('sys_normal_disable').then(response => {
        this.statusOptions = response.data
      })
    },

    /** 格式化日期时间 */
    formatDateTime(dateStr) {
      if (!dateStr) {
        return '-'
      }
      const date = new Date(dateStr)
      return date.toLocaleString('zh-CN', {
        year: 'numeric',
        month: '2-digit',
        day: '2-digit',
        hour: '2-digit',
        minute: '2-digit',
        second: '2-digit'
      })
    },

    /** 复制到剪贴板 */
    copyToClipboard(text) {
      if (!text) {
        return
      }

      const textarea = document.createElement('textarea')
      textarea.value = text
      document.body.appendChild(textarea)
      textarea.select()
      document.execCommand('copy')
      document.body.removeChild(textarea)

      this.$message.success('复制成功')
    },

    /** 显示/隐藏密钥 */
    appSecretShow() {
      this.showAppSecret = !this.showAppSecret
    },

    /** 显示/隐藏HTTP密钥 */
    toggleHttpSecret() {
      this.showHttpSecret = !this.showHttpSecret
    },

    /** 显示/隐藏RocketMQ密钥 */
    toggleRocketMQSecret() {
      this.showRocketMQSecret = !this.showRocketMQSecret
    },

    /** 显示/隐藏MQTT密码 */
    toggleMQTTSecret() {
      this.showMQTTSecret = !this.showMQTTSecret
    },

    /** 获取HTTP配置 */
    getHttpConfig() {
      if (!this.httpConfig) {
        return null
      }

      // 返回HTTP配置
      return this.httpConfig.http || null
    },

    /** 获取当前激活的推送方式 */
    getActivePushMethod() {
      if (!this.httpConfig) {
        return null
      }

      // 检查哪种推送方式有配置
      if (this.httpConfig.http && (this.httpConfig.http.url || this.httpConfig.http.secret)) {
        return 'HTTP'
      } else if (this.httpConfig.kafka && Object.keys(this.httpConfig.kafka).length > 0) {
        return 'Kafka'
      } else if (this.httpConfig.rocketMQ && Object.keys(this.httpConfig.rocketMQ).length > 0) {
        return 'RocketMQ'
      } else if (this.httpConfig.mqtt && Object.keys(this.httpConfig.mqtt).length > 0) {
        return 'MQTT'
      } else if (this.httpConfig.jdbc && Object.keys(this.httpConfig.jdbc).length > 0) {
        return 'JDBC'
      }

      return null
    },

    /** 检查是否有HTTP配置 */
    hasHttpConfig() {
      return this.httpConfig && this.httpConfig.http && (this.httpConfig.http.url
        || this.httpConfig.http.secret)
    },

    /** 检查是否有RocketMQ配置 */
    hasRocketMQConfig() {
      return this.httpConfig && this.httpConfig.rocketMQ && Object.keys(
        this.httpConfig.rocketMQ).length > 0
    },

    /** 检查是否有MQTT配置 */
    hasMQTTConfig() {
      return this.httpConfig && this.httpConfig.mqtt && Object.keys(this.httpConfig.mqtt).length > 0
    },

    /** 检查是否有JDBC配置 */
    hasJDBCConfig() {
      return this.httpConfig && this.httpConfig.jdbc && Object.keys(this.httpConfig.jdbc).length > 0
    },

    /** 检查是否有任何配置 */
    hasAnyConfig() {
      return this.hasHttpConfig() || this.hasRocketMQConfig() || this.hasMQTTConfig()
        || this.hasJDBCConfig()
    },

    /** 获取RocketMQ配置 */
    getRocketMQConfig() {
      if (!this.httpConfig || !this.httpConfig.rocketMQ) {
        return {}
      }
      return this.httpConfig.rocketMQ
    },

    /** 获取MQTT配置 */
    getMQTTConfig() {
      if (!this.httpConfig || !this.httpConfig.mqtt) {
        return {}
      }
      return this.httpConfig.mqtt
    },

    /** 获取JDBC配置 */
    getJDBCConfig() {
      if (!this.httpConfig || !this.httpConfig.jdbc) {
        return {}
      }
      return this.httpConfig.jdbc
    },

    /** 获取配置 */
    getConfig(type) {
      if (!this.httpConfig || !this.httpConfig[type]) {
        // 返回默认配置，根据推送方式类型设置默认support状态
        const defaultSupport = {
          'http': true,      // HTTP推送已支持
          'rocketmq': false, // RocketMQ开发中
          'kafka': false,    // Kafka开发中
          'mqtt': false,     // MQTT开发中
          'jdbc': false      // JDBC开发中
        }
        return {
          support: defaultSupport[type] || false,
          enable: false
        }
      }

      const config = this.httpConfig[type]

      // 如果配置中没有support字段，根据推送方式类型设置默认值
      if (config.support === undefined) {
        const defaultSupport = {
          'http': true,      // HTTP推送已支持
          'rocketmq': false, // RocketMQ开发中
          'kafka': false,    // Kafka开发中
          'mqtt': false,     // MQTT开发中
          'jdbc': false      // JDBC开发中
        }
        config.support = defaultSupport[type] || false
      }

      // 如果配置中没有enable字段，根据是否有配置内容来设置
      if (config.enable === undefined) {
        if (type === 'http') {
          config.enable = !!(config.url || config.secret)
        } else if (type === 'rocketmq') {
          config.enable = !!(config.namesrvAddr || config.topic)
        } else if (type === 'kafka') {
          config.enable = !!(config.bootstrapServers || config.topic)
        } else if (type === 'mqtt') {
          config.enable = !!(config.url || config.topic)
        } else if (type === 'jdbc') {
          config.enable = !!(config.url || config.username)
        } else {
          config.enable = false
        }
      }

      return config
    },
    onHttpEnableChange(checked) {
      // 这里可加API调用或本地更新cfg逻辑
      this.httpConfig.http.enable = checked
      this.$message.success(checked ? 'HTTP推送已启用' : 'HTTP推送已关闭')
    },
    onMqttEnableChange(checked) {
      this.httpConfig.mqtt.enable = checked
      this.$message.success(checked ? 'MQTT推送已启用' : 'MQTT推送已关闭')
    },
    onRocketMQEnableChange(checked) {
      this.httpConfig.rocketMQ.enable = checked
      this.$message.success(checked ? 'RocketMQ推送已启用' : 'RocketMQ推送已关闭')
    },
    onJDBCEnableChange(checked) {
      this.httpConfig.jdbc.enable = checked
      this.$message.success(checked ? 'JDBC推送已启用' : 'JDBC推送已关闭')
    },

    /** 返回上一级 */
    back() {
      this.$router.push({name: 'Application'})
    },

    /** 显示重置密钥弹窗 */
    showModal() {
      this.modalVisible = true
      this.checkInfo = undefined
    },

    /** 取消重置密钥 */
    modalCancel() {
      this.modalVisible = false
      this.checkInfo = undefined
    },

    /** 禁止粘贴 */
    handlePaste() {
      this.$message.warning('为确保操作安全，请手动输入应用ID')
    },

    /** 重置密钥 */
    handleReset(row) {
      if (this.checkInfo !== this.application.appId) {
        this.$message.error('输入的应用ID不正确')
        return
      }

      resetSecret({appUniqueId: row.appUniqueId}).then(() => {
        this.getList()
        this.$message.success('密钥重置成功')
        this.modalCancel()
      }).catch(() => {
        this.$message.error('密钥重置失败')
      })
    },

    /** HTTP协议开关 */
    onHttpChange(checked) {
      this.form.appUniqueId = this.appUniqueId
      this.form.httpEnable = checked
      this.form.mqttEnable = this.mqttEnable

      enable(this.form).then(() => {
        this.$message.success('HTTP协议状态修改成功')
      }).catch(() => {
        this.$message.error('修改失败')
        this.httpEnable = !checked // 回滚状态
      })
    },

    /** 下载配置文件 */
    downloadConfig(filename, text) {
      const element = document.createElement('a')
      element.setAttribute('href', 'data:text/plain;charset=utf-8,' + encodeURIComponent(text))
      element.setAttribute('download', filename)
      element.style.display = 'none'

      document.body.appendChild(element)
      element.click()
      document.body.removeChild(element)

      this.$message.success('配置文件下载成功')
    },

    /** 获取HTTP状态 */
    getHttpStatus() {
      const activeMethod = this.getActivePushMethod()
      if (this.httpEnable && activeMethod === 'HTTP') {
        return '已启用'
      } else if (this.httpEnable && activeMethod && activeMethod !== 'HTTP') {
        return '开发中'
      } else if (this.httpEnable && !activeMethod) {
        return '已启用但未配置'
      } else {
        return '未启用'
      }
    },

    /** 测试HTTP连接 */
    testHttpUrl() {
      const httpConfig = this.getHttpConfig()
      if (!httpConfig || !httpConfig.url) {
        this.$message.warning('请先配置HTTP推送地址')
        return
      }

      // 这里可以调用API测试HTTP连接
      this.$message.info('HTTP连接测试功能开发中')
    },

    /** 设置默认展开的面板 */
    setDefaultActiveCollapse() {
      if (this.hasHttpConfig()) {
        this.activeCollapseKeys = ['http']
      } else if (this.hasRocketMQConfig()) {
        this.activeCollapseKeys = ['rocketmq']
      } else if (this.hasMQTTConfig()) {
        this.activeCollapseKeys = ['mqtt']
      } else if (this.hasJDBCConfig()) {
        this.activeCollapseKeys = ['jdbc']
      } else {
        this.activeCollapseKeys = []
      }
    }
  }
}
</script>

<style scoped lang="less">
.app-container {
  // max-width: 960px;
  // margin: 32px auto 0 auto;
  // padding: 0 16px 32px 16px;
  background: #fff;
  border-radius: 10px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.03);
}

.page-header {
  margin-bottom: 12px;
  padding: 0;
  border-bottom: none;

  .back-btn {
    margin-bottom: 8px;
    padding: 0;
    color: #1890ff;
    font-size: 13px;

    &:hover {
      color: #40a9ff;
    }
  }

  .app-title {
    display: flex;
    align-items: center;
    gap: 12px;

    .app-icon {
      width: 48px;
      height: 48px;
      border-radius: 8px;
      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
      display: flex;
      align-items: center;
      justify-content: center;

      .anticon {
        font-size: 24px;
        color: #fff;
      }
    }

    .app-info {
      display: flex;
      align-items: center;
      gap: 12px;

      .app-name {
        font-size: 18px;
        font-weight: 600;
        color: #222;
        margin: 0;
      }

      .app-meta {
        display: flex;
        align-items: center;
        gap: 8px;

        .app-id {
          color: #b0b3b9;
          font-size: 12px;
          background: #f5f6fa;
          padding: 2px 6px;
          border-radius: 4px;
          border: none;
        }
      }
    }
  }
}

.detail-tabs {
  :deep(.ant-tabs-bar) {
    margin-bottom: 12px;
    border-bottom: 1px solid #f0f0f0;

    .ant-tabs-tab {
      font-size: 14px;
      padding: 8px 16px;
    }
  }
}

.tab-content {
  .info-card {
    margin-bottom: 12px;
    border-radius: 8px;
    box-shadow: 0 1px 4px rgba(0, 0, 0, 0.03);
    border: none;
    background: #fafbfc;

    :deep(.ant-card-head) {
      background: transparent;
      padding: 12px 16px;
    }

    :deep(.ant-card-body) {
      padding: 16px;
    }
  }
}

.info-item {
  .info-label {
    font-size: 13px;
    color: #b0b3b9;
    margin-bottom: 4px;
    font-weight: 500;
  }

  .info-value {
    font-size: 13px;
    color: #222;
    word-break: break-all;
    line-height: 1.5;

    &.code-value, &.url-value {
      font-family: 'Monaco', 'Menlo', monospace;
      background: #f5f6fa;
      padding: 8px 12px;
      border-radius: 6px;
      border: 1px solid #f0f0f0;
      display: flex;
      align-items: center;
      justify-content: space-between;
      font-size: 12px;
    }
  }
}

.copy-btn {
  padding: 4px 6px;
  color: #1890ff;
  border-radius: 4px;
  transition: all 0.2s;

  &:hover {
    color: #40a9ff;
    background: #f0f8ff;
  }
}

.protocol-item {
  margin-bottom: 12px;
  padding: 12px;
  background: #f8f9fb;
  border-radius: 8px;
  border: 1px solid #f0f0f0;

  .protocol-header {
    margin-bottom: 10px;
  }

  .protocol-info {
    gap: 8px;
  }

  .protocol-tag {
    font-size: 12px;
    padding: 2px 10px;
    border-radius: 12px;
  }

  .protocol-desc {
    color: #b0b3b9;
    font-size: 12px;
  }
}

.protocol-config {
  .config-item {
    margin-bottom: 10px;
  }

  .config-label {
    font-size: 13px;
    color: #4e5969;
    margin-bottom: 4px;
    font-weight: 500;
  }

  .config-value .config-input {
    background: #fff;
    border: 1px solid #f0f0f0;
    border-radius: 6px;
    padding: 8px 12px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    font-size: 12px;
  }

  .config-actions {
    gap: 4px;
    margin-left: 8px;
  }
}

.protocol-summary {
  margin-top: 12px;
  padding: 12px;
  background: #f8f9fb;
  border-radius: 8px;
  border: 1px solid #f0f0f0;

  .summary-title {
    font-size: 14px;
    font-weight: 600;
    color: #222;
    margin-bottom: 10px;
  }

  .summary-content {
    gap: 12px;
  }

  .status-item {
    padding: 8px 12px;
    border-radius: 6px;
    font-size: 12px;
  }
}

.reset-confirm {
  .reset-content {
    h4 {
      font-size: 15px;
      font-weight: 600;
      margin-bottom: 8px;
    }

    .reset-warning {
      font-size: 13px;
      margin-bottom: 8px;
    }

    .confirm-code {
      padding: 6px 10px;
      border-radius: 6px;
      font-size: 13px;
    }
  }
}

.protocol-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding-bottom: 4px;

  .protocol-label {
    display: flex;
    align-items: center;
    gap: 8px;

    .ant-tag {
      font-size: 13px;
    }

    .ant-switch {
      margin-left: 8px;
    }
  }

  .protocol-desc {
    color: #b0b3b9;
    font-size: 13px;
    flex: 1;
    text-align: right;
    margin-left: 16px;
  }
}

// 折叠面板样式
:deep(.ant-collapse) {
  border: none;
  background: transparent;

  .ant-collapse-item {
    margin-bottom: 8px;
    border: 1px solid #f0f0f0;
    border-radius: 8px;
    background: #fff;
    overflow: hidden;

    .ant-collapse-header {
      background: #f8f9fb;
      padding: 12px 16px;
      border-bottom: 1px solid #f0f0f0;
      display: flex;
      align-items: center;
      gap: 8px;

      .ant-collapse-arrow {
        color: #1890ff;
        font-size: 12px;
        margin-right: 8px;
        flex-shrink: 0;
      }

      .ant-collapse-header-text {
        display: flex;
        align-items: center;
        flex: 1;
        min-width: 0;
      }
    }

    .ant-collapse-content {
      border: none;

      .ant-collapse-content-box {
        padding: 16px;
        background: #fff;
      }
    }
  }
}

:deep(.ant-collapse-item) {
  border-radius: 12px;
  margin-bottom: 16px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.04);
  border: none;
  overflow: hidden;
}

:deep(.ant-collapse-header) {
  background: #fff;
  border-radius: 12px 12px 0 0;
  min-height: 56px;
  display: flex;
  align-items: center;
  gap: 16px;
  font-size: 16px;
  font-weight: 600;
  color: #222;
  box-shadow: 0 1px 0 #f0f0f0;
  transition: background 0.2s;
}

:deep(.ant-collapse-header):hover {
  background: #f7faff;
}

.collapse-title {
  flex: 1;
  min-width: 0;
  font-size: 16px;
  font-weight: 600;
  color: #222;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.protocol-tag {
  display: flex;
  align-items: center;
  font-size: 16px;
  color: #1890ff;
  background: #f0f8ff;
  border-radius: 8px;
  padding: 2px 12px;
  margin-right: 8px;
  font-weight: 500;
}

.protocol-tag .anticon {
  font-size: 18px;
  margin-right: 6px;
}

.protocol-switch {
  margin-left: 12px;
  flex-shrink: 0;
}

.database-config {
  margin-bottom: 16px;
  padding: 12px;
  background: #f8f9fb;
  border-radius: 6px;
  border: 1px solid #f0f0f0;

  .db-header {
    margin-bottom: 8px;

    .ant-tag {
      font-size: 11px;
      padding: 1px 6px;
      border-radius: 8px;
    }
  }
}

.config-tips {
  margin-top: 12px;
  padding: 8px 12px;
  background: #f0f8ff;
  border-radius: 6px;
  border-left: 3px solid #1890ff;
  font-size: 12px;
  color: #1890ff;

  .anticon {
    margin-right: 6px;
  }
}

.protocol-empty {
  padding: 32px 16px;
  text-align: center;

  .empty-content {
    .empty-icon {
      font-size: 48px;
      color: #d9d9d9;
      margin-bottom: 16px;
    }

    .empty-text {
      font-size: 14px;
      color: #b0b3b9;
      margin-bottom: 16px;
    }
  }
}

.protocol-dev {
  padding: 32px 16px;
  text-align: center;
  color: #b0b3b9;
  font-size: 15px;
}

@media (max-width: 768px) {
  .app-container {
    padding: 0 4px;
  }

  .page-header .app-title {
    flex-direction: column;
    align-items: flex-start;
    gap: 8px;
  }

  .tab-content .info-card {
    padding: 0;
  }
}
</style>
