<template>
  <div class="product-details-container">
    <!-- 页面头部 -->
    <div class="page-header">
      <div class="header-left">
        <a-button
          type="text"
          icon="left"
          @click="back()"
          class="back-btn"
        />
      </div>
    </div>

    <!-- 自定义标签页 -->
    <div class="custom-tabs-container">
      <a-spin :spinning="loading" tip="Loading...">
        <div class="custom-tabs-nav">
          <div
            class="custom-tab-item"
            :class="{ active: activeKey === '1' }"
            @click="activeKey = '1'"
          >
            产品信息
          </div>
          <div
            class="custom-tab-item"
            :class="{ active: activeKey === '2' }"
            @click="activeKey = '2'"
          >
            物模型
          </div>
          <div
            class="custom-tab-item"
            :class="{ active: activeKey === '3' }"
            @click="activeKey = '3'"
          >
            设备管理
          </div>
          <div
            v-if="productDetails.creatorId === $store.state.user.name"
            class="custom-tab-item"
            :class="{ active: activeKey === '4' }"
            @click="activeKey = '4'"
          >
            协议管理
          </div>
          <div
            v-if="['tcp', 'mqtt'].includes(productDetails.thirdPlatform) && productDetails.creatorId === $store.state.user.name && productDetails.deviceNode !== 'GATEWAY_SUB_DEVICE'"
            class="custom-tab-item"
            :class="{ active: activeKey === '5' }"
            @click="activeKey = '5'"
          >
            网络绑定
          </div>
          <div
            v-if="['sniTcp', 'mqtt'].includes(productDetails.thirdPlatform) &&  productDetails.deviceNode !== 'GATEWAY_SUB_DEVICE' && productDetails.creatorId === $store.state.user.name"
            class="custom-tab-item"
            :class="{ active: activeKey === '6' }"
            @click="activeKey = '6'"
          >
            证书管理
          </div>
        </div>

        <div class="custom-tab-content">
          <!-- 产品信息 -->
          <div v-show="activeKey === '1'" class="tab-pane">
            <div class="product-info-content">
              <!-- 产品概览卡片 -->
              <div class="product-overview-card">
                <div class="overview-header">
                  <div class="product-title">
                    <h2>{{ productDetails.name || '产品名称' }}</h2>
                    <div class="product-key">
                      <span class="key-label">ProductKey:</span>
                      <span class="key-value">{{ productDetails.productKey || '-' }}</span>
                      <a-button
                        type="link"
                        size="small"
                        icon="copy"
                        @click="copyToClipboard(productDetails.productKey)"
                        class="copy-btn"
                      />
                    </div>
                  </div>
                  <div class="product-tags">
                    <a-tag color="orange"> {{ productDetails.thirdPlatform || '-' }}</a-tag>
                    <a-tag color="blue">{{ productDetails.transportProtocol }}</a-tag>
                    <a-tag color="green">{{
                        productDetails.storePolicy == 'mysql' ? "MYSQL" : "ClickHouse"
                      }}
                    </a-tag>
                    <a-tag color="purple">{{ getDeviceNodeText(productDetails.deviceNode) }}</a-tag>
                  </div>
                </div>

                <div class="overview-info">
                  <div class="info-item">
                    <span class="label">厂商简称</span>
                    <span class="value">{{ productDetails.companyNo || '-' }}</span>
                  </div>
                  <div class="info-item">
                    <span class="label">所属分类</span>
                    <span class="value">{{ productSortName || '-' }}</span>
                  </div>
                  <div class="info-item"
                       v-if="productDetails.creatorId === $store.state.user.name || $store.state.user.currentAdmin">
                    <span class="label">创建人</span>
                    <span class="value">{{ productDetails.creatorId || '-' }}</span>
                  </div>
                  <div class="info-item full-width" v-if="productDetails.describe">
                    <span class="label">产品描述</span>
                    <span class="value description">{{ productDetails.describe }}</span>
                  </div>
                </div>
              </div>

              <!-- 设备标签卡片 -->
              <div class="info-card" v-if="$store.state.user.currentAdmin">
                <div class="card-header">
                  <div class="card-title">
                    <a-icon type="tags"/>
                    <span>设备标签</span>
                  </div>
                  <a-button
                    type="link"
                    icon="edit"
                    @click="editTags"
                    class="edit-btn"
                  >
                    {{ editTagStatus ? '取消' : '编辑' }}
                  </a-button>
                </div>
                <div class="card-content">
                  <div v-show="!editTagStatus" class="tags-display">
                    <a-tag
                      v-for="(item, index) in productDetails.tagsName"
                      :key="index"
                      class="device-tag"
                    >
                      {{ item }}
                    </a-tag>
                    <span v-if="!productDetails.tagsName || productDetails.tagsName.length === 0"
                          class="no-tags">
                      暂无标签
                    </span>
                  </div>
                  <div v-show="editTagStatus" class="tags-edit">
                    <a-select
                      mode="multiple"
                      placeholder="请选择设备标签"
                      v-model="tagList"
                      allow-clear
                      @change="handleTagChange"
                      class="tag-select"
                    >
                      <a-select-option v-for="item in productTags" :value="item.id" :key="item.id">
                        {{ item.name }}
                      </a-select-option>
                    </a-select>
                    <a-button
                      type="primary"
                      @click="confirmTags"
                      class="save-btn"
                    >
                      保存
                    </a-button>
                  </div>
                </div>
              </div>

              <!-- 网关产品卡片 -->
              <div class="info-card" v-if="productDetails.deviceNode === 'GATEWAY_SUB_DEVICE'">
                <div class="card-header">
                  <div class="card-title">
                    <a-icon type="gateway"/>
                    <span>关联网关</span>
                  </div>
                  <a-button
                    v-if="!selectedGateway"
                    type="primary"
                    icon="plus"
                    @click="openGatewayModal"
                    class="add-btn"
                    v-hasPermi="['system:product:edit']"
                  >
                    添加网关
                  </a-button>
                  <a-button
                    v-else
                    type="default"
                    icon="edit"
                    @click="openGatewayModal"
                    class="edit-btn"
                    v-hasPermi="['system:product:edit']"
                  >
                    更换网关
                  </a-button>
                </div>
                <div class="card-content">
                  <div class="gateway-info">
                    <div v-if="selectedGateway" class="selected-gateway">
                      <a-tag
                        closable
                        @close="removeGateway"
                        class="gateway-tag"
                        color="blue"
                      >
                        {{ selectedGateway.name }}
                      </a-tag>
                    </div>
                    <span v-else class="no-gateways">
                      暂无关联网关产品
                    </span>
                  </div>
                  <SelectGateway ref="selectGateway" @add="addGateway"
                                 :selectedGateway="selectedGateway"/>
                </div>
              </div>

              <!-- 配置信息区域 -->
              <template
                v-if="productDetails.creatorId === $store.state.user.name || $store.state.user.currentAdmin">
                <!-- 使用新的配置编辑器组件 -->
                <ConfigurationEditor
                  v-if="publicConfigurationDicts.length > 0 || configurationDicts.length > 0"
                  :product-id="productId"
                  :public-configuration-array="publicConfigurationArray"
                  :configuration-array="configurationArray"
                  :public-configuration-dicts="publicConfigurationDicts"
                  :configuration-dicts="configurationDicts"
                  @config-updated="getProductDeatils"
                />

                <!-- 存储策略 -->
                <!-- <div class="info-card" v-if="storePolicyCfgArray.length > 0"> -->
                <div class="info-card">
                  <div class="card-header">
                    <div class="card-title">
                      <a-icon type="database"/>
                      <span>存储策略</span>
                    </div>
                    <a-button
                      type="link"
                      icon="edit"
                      @click="editConfig('storageConfiguration')"
                      v-if="productDetails.creatorId === $store.state.user.name || $store.state.user.currentAdmin"
                      v-hasPermi="['system:product:edit']"
                      class="edit-btn"
                    >
                      编辑
                    </a-button>
                  </div>
                  <div class="card-content">
                    <a-table
                      :data-source="storePolicyCfgArray"
                      :columns="storageColumns"
                      :pagination="false"
                      size="middle"
                      class="storage-table"
                    >
                      <span slot="key" slot-scope="text, record">
                        <div class="storage-identifier">
                          <a-icon type="tag"/>
                          <span>{{ record.key }}</span>
                        </div>
                      </span>
                      <span slot="value" slot-scope="text, record">
                        <div class="storage-limit">
                          <span class="limit-value">{{ record.value }}</span>
                          <span class="limit-unit">条</span>
                        </div>
                      </span>
                      <span slot="type" slot-scope="text, record">
                        <a-tag :color="record.type === 'properties' ? 'blue' : 'green'">
                          {{ record.type === 'properties' ? '属性' : '事件' }}
                        </a-tag>
                      </span>
                    </a-table>
                  </div>
                </div>

                <!-- 自定义参数 -->
                <div class="info-card" v-if="otherConfiguration.length > 0">
                  <div class="card-header">
                    <div class="card-title">
                      <a-icon type="code"/>
                      <span>自定义参数</span>
                    </div>
                    <a-button
                      type="link"
                      icon="edit"
                      @click="editConfig('otherConfiguration')"
                      v-if="productDetails.creatorId === $store.state.user.name"
                      v-hasPermi="['system:product:edit']"
                      class="edit-btn"
                    >
                      编辑
                    </a-button>
                  </div>
                  <div class="card-content">
                    <div class="custom-params">
                      <div
                        v-for="item in otherConfiguration"
                        :key="item.id"
                        class="param-item"
                      >
                        <div class="param-header">
                          <span class="param-id">{{ item.id || "缺省" }}</span>
                          <span class="param-name">{{ item.name || "缺省" }}</span>
                        </div>
                        <div class="param-description">{{ item.description || "缺省" }}</div>
                      </div>
                    </div>
                  </div>
                </div>
              </template>
            </div>
          </div>

          <!-- 物模型 -->
          <div v-show="activeKey === '2'" class="tab-pane">
            <!-- 物模型组件 -->
            <div class="metadata-wrapper">
              <metadata
                ref="metadata"
                :product-id="productId"
                :creator-id="productDetails.creatorId"
                :product-key="productDetails.productKey"
                @openMetadataDetail="openMetadataDetail"
                @openImportMetadata="openImportMetadata"/>
            </div>
          </div>

          <!-- 设备管理 -->
          <div v-show="activeKey === '3'" class="tab-pane">
            <!-- <div class="simple-header">
              <h3>设备管理</h3>
              <p>管理该产品下的所有设备实例</p>
            </div> -->
            <product-instance
              v-if="activeKey === '3' && productDetails.productKey"
              ref="instance"
              :product-key="productDetails.productKey"
            />
          </div>

          <!-- 协议管理 -->
          <div v-show="activeKey === '4'" class="tab-pane"
               v-if="productDetails.creatorId === $store.state.user.name">
            <!-- <div class="simple-header">
              <h3>协议管理</h3>
              <p>管理产品的通信协议和数据解析规则</p>
            </div> -->
            <product-protocol
              ref="protocol"
              :product-key="productDetails.productKey"
              :product-name="productDetails.name"
              :user-name="productDetails.creatorId"/>
          </div>

          <!-- 网络组件关联 -->
          <div v-show="activeKey === '5'" class="tab-pane"
               v-if="['tcp', 'mqtt'].includes(productDetails.thirdPlatform)">
            <!-- <div class="simple-header">
              <h3>网络组件关联</h3>
              <p>管理产品与网络组件的绑定关系</p>
            </div> -->
            <network-component-bind
              :product="productDetails"
              @refresh="getProductDeatils"/>
          </div>
          <!-- 证书管理 -->
          <div v-show="activeKey === '6'" class="tab-pane"
               v-if="['sniTcp', 'mqtt'].includes(productDetails.thirdPlatform)">
            <!-- <div class="simple-header">
              <h3>证书绑定</h3>
              <p>管理产品证书的绑定与信息</p>
            </div> -->
            <CertificateBind
              :key="getSslKeyFromConfig(productDetails.configuration) || 'no-cert'"
              :productKey="productDetails.productKey"
              :bindCertKey="getSslKeyFromConfig(productDetails.configuration)"
              @refresh="getProductDeatils"
            />
          </div>
        </div>
      </a-spin>
    </div>

    <!-- 修改产品信息 -->
    <create-form ref="createForm" @ok="getList"/>
    <!-- 修改产品存储策略配置信息 -->
    <storage-configuration-form ref="storageConfigurationForm" @ok="storageConfigOk"/>
    <!-- 修改产品配置信息 -->
    <configuration-form ref="configurationForm" :dicts="dicts" @ok="getProductDeatils"/>
    <!-- 修改产品其他配置信息 -->
    <other-config-form ref="otherConfigurationForm" @ok="getProductDeatils"/>
    <!-- 物模型详细信息 -->
    <metadata-show
      ref="metadataShow"
      :product-id="productId"
      :show="metadataShow"
      :gid="productDetails.configuration"
      :type="metadataShowType"
      @close="closeMetadataDetail"/>
    <!-- 物模型导入 -->
    <import-metadata
      ref="importMetadata"
      :product-id="productId"
      :show="importMetadataShow"
      @close="closeImportMetadata"
      @ok="successUpdateMetadata"/>
  </div>
</template>
<style scoped lang="less">
.product-details-container {
  background: #ffffff;
  padding: 0;
}

/* 页面头部 */
.page-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px 20px;
  background: #ffffff;
  border-bottom: 1px solid #e8eaed;

  .header-left {
    display: flex;
    align-items: center;
    gap: 16px;

    .back-btn {
      width: 40px;
      height: 40px;
      border-radius: 8px;
      background: #f8fafc;
      border: 1px solid #e2e8f0;
      display: flex;
      align-items: center;
      justify-content: center;
      transition: all 0.2s ease;

      &:hover {
        background: #e2e8f0;
        border-color: #1966ff;
        color: #1966ff;
        transform: scale(1.05);
      }
    }
  }
}

/* 自定义标签页样式 */
.custom-tabs-container {
  background: #fff;
  border-radius: 6px;
  border: 1px solid #e8e8e8;
  overflow: hidden;
}

.custom-tabs-nav {
  display: flex;
  background: #fafafa;
  border-bottom: 1px solid #e8e8e8;
  overflow-x: auto;
  scrollbar-width: none;
  -ms-overflow-style: none;
}

.custom-tabs-nav::-webkit-scrollbar {
  display: none;
}

.custom-tab-item {
  display: flex;
  align-items: center;
  padding: 12px 16px;
  cursor: pointer;
  transition: all 0.2s ease;
  color: #666;
  font-size: 14px;
  font-weight: 500;
  white-space: nowrap;
  min-width: fit-content;
  border-right: 1px solid #e8e8e8;
  background: #fafafa;
}

.custom-tab-item:last-child {
  border-right: none;
}

.custom-tab-item:hover {
  background: #e6f7ff;
  color: #1890ff;
}

.custom-tab-item.active {
  background: #ffffff;
  color: #1890ff;
  border-bottom: 2px solid #1890ff;
  margin-bottom: -1px;
}

.custom-tab-content {
  min-height: 500px;
  background: #fff;
}

.tab-pane {
  padding: 10px;
}

/* 产品信息内容区域 */
.product-info-content {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

/* 产品概览卡片 */
.product-overview-card {
  background: #ffffff;
  border: 1px solid #e8eaed;
  border-radius: 12px;
  padding: 24px;
  margin-bottom: 20px;
  /* 去掉阴影效果，保持扁平样式 */
  /* box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08); */

  .overview-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 20px;

    .product-title {
      h2 {
        margin: 0 0 8px 0;
        font-size: 20px;
        font-weight: 600;
        color: #1a202c;
      }

      .product-key {
        display: flex;
        align-items: center;
        gap: 8px;
        margin-top: 4px;

        .key-label {
          font-size: 14px;
          color: #64748b;
        }

        .key-value {
          font-family: 'Monaco', 'Menlo', monospace;
          font-size: 16px;
          font-weight: 600;
          color: #1a202c;
          word-break: break-all;
          flex: 1;
        }

        .copy-btn {
          background: #f8fafc;
          border: 1px solid #e2e8f0;
          color: #64748b;
          width: 32px;
          height: 32px;
          border-radius: 8px;
          display: flex;
          align-items: center;
          justify-content: center;
          transition: all 0.2s ease;

          &:hover {
            background: #1966ff;
            border-color: #1966ff;
            color: white;
            transform: scale(1.05);
          }
        }
      }

      .product-tags {
        display: flex;
        flex-wrap: wrap;
        gap: 8px;

        .ant-tag {
          border-radius: 6px;
          font-weight: 500;
          font-size: 12px;
          padding: 2px 8px;
          border: none;
          box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
        }
      }
    }
  }

  .overview-info {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    gap: 16px;

    .info-item {
      display: flex;
      align-items: center;
      gap: 12px;
      padding: 12px 16px;
      background: #f8fafc;
      border: 1px solid #e2e8f0;
      border-radius: 8px;
      transition: all 0.2s ease;

      &:hover {
        background: #f1f5f9;
        border-color: #cbd5e1;
        transform: translateY(-1px);
        box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
      }

      &.full-width {
        grid-column: 1 / -1;
      }

      .label {
        font-size: 12px;
        color: #64748b;
        font-weight: 500;
        line-height: 1.2;
        min-width: 60px;
      }

      .value {
        font-size: 14px;
        color: #1a202c;
        font-weight: 500;
        line-height: 1.3;
        flex: 1;

        &.description {
          line-height: 1.5;
          color: #4a5568;
        }
      }
    }
  }
}

/* 信息卡片 */
.info-card {
  background: #ffffff;
  border: 1px solid #e8eaed;
  border-radius: 12px;
  overflow: hidden;
  transition: all 0.2s ease;
  margin-bottom: 16px;

  &:hover {
    box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
    transform: translateY(-2px);
  }

  .card-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 16px 20px;
    background: #f8fafc;
    border-bottom: 1px solid #e8eaed;

    .card-title {
      display: flex;
      align-items: center;
      gap: 8px;
      font-size: 16px;
      font-weight: 600;
      color: #1a202c;

      .anticon {
        color: #1966ff;
        font-size: 16px;
      }
    }

    .edit-btn, .add-btn {
      border-radius: 6px;
      font-weight: 500;
      height: 32px;
      padding: 0 12px;
    }
  }

  .card-content {
    padding: 20px;
  }
}


/* 设备标签 */
.tags-display {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;

  .device-tag {
    border-radius: 6px;
    font-weight: 500;
    background: #f0f9ff;
    border-color: #bae6fd;
    color: #0369a1;
  }

  .no-tags {
    color: #94a3b8;
    font-style: italic;
    padding: 8px 0;
  }
}

.tags-edit {
  display: flex;
  gap: 12px;
  align-items: center;

  .tag-select {
    flex: 1;
    max-width: 400px;
  }

  .save-btn {
    border-radius: 6px;
    height: 32px;
    padding: 0 16px;
  }
}

/* 网关标签 */
.gateway-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;

  .gateway-tag {
    border-radius: 6px;
    font-weight: 500;
    background: #fef3c7;
    border-color: #fbbf24;
    color: #92400e;
  }

  .no-gateways {
    color: #94a3b8;
    font-style: italic;
    padding: 8px 0;
  }
}

/* 配置列表 */
.config-list {
  display: flex;
  flex-direction: column;
  gap: 12px;

  .config-item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 12px 16px;
    background: #f8fafc;
    border: 1px solid #e2e8f0;
    border-radius: 8px;
    transition: all 0.2s ease;

    &:hover {
      background: #f1f5f9;
      border-color: #cbd5e1;
      transform: translateX(4px);
    }

    .config-label {
      font-size: 14px;
      color: #64748b;
      font-weight: 500;
    }

    .config-value {
      font-size: 14px;
      color: #1a202c;
      font-weight: 500;
      max-width: 200px;
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: nowrap;
    }
  }
}

/* 存储策略表格 */
.storage-table {
  border-radius: 14px;
  overflow: hidden;

  :deep(.ant-table) {
    border-radius: 14px;
    background: transparent;
    font-size: 14px;
  }

  :deep(.ant-table-thead > tr > th) {
    background: #f8fafc;
    border-bottom: 1.5px solid #e2e8f0;
    color: #374151;
    font-weight: 600;
    font-size: 14px;
    padding: 8px 10px;
    height: 40px;
  }

  :deep(.ant-table-tbody > tr > td) {
    padding: 8px 10px;
    border-bottom: 1px solid #f1f5f9;
    font-size: 14px;
    background: #fff;
    line-height: 1.5;
    height: 40px;
  }

  :deep(.ant-table-tbody > tr) {
    transition: all 0.2s ease;

    &:hover > td {
      background: #f4f8ff;
    }
  }
}

.storage-identifier {
  display: flex;
  align-items: center;
  gap: 8px;
  font-weight: 500;
  color: #1966ff;

  .anticon {
    font-size: 14px;
    color: #64748b;
  }
}

.storage-limit {
  display: flex;
  align-items: center;
  gap: 4px;

  .limit-value {
    font-size: 16px;
    font-weight: 600;
    color: #0c4a6e;
  }

  .limit-unit {
    font-size: 12px;
    color: #64748b;
  }
}

/* 自定义参数 */
.custom-params {
  display: flex;
  flex-direction: column;
  gap: 12px;

  .param-item {
    padding: 16px;
    background: #f8fafc;
    border: 1px solid #e2e8f0;
    border-radius: 8px;
    transition: all 0.2s ease;

    &:hover {
      background: #f1f5f9;
      border-color: #cbd5e1;
      transform: translateX(4px);
    }

    .param-header {
      display: flex;
      align-items: center;
      gap: 12px;
      margin-bottom: 8px;

      .param-id {
        font-family: 'Monaco', 'Menlo', monospace;
        font-size: 12px;
        color: #1966ff;
        font-weight: 600;
        background: #eff6ff;
        padding: 4px 8px;
        border-radius: 4px;
        border: 1px solid #dbeafe;
      }

      .param-name {
        font-size: 14px;
        color: #1a202c;
        font-weight: 500;
      }
    }

    .param-description {
      font-size: 13px;
      color: #64748b;
      line-height: 1.5;
      padding-left: 4px;
    }
  }
}

/* 物模型标签页样式 */
.metadata-tab-content {
  display: flex;
  flex-direction: column;
  gap: 0;
}

.metadata-wrapper {
  background: #ffffff;
  // border: 1px solid #e8eaed;
  border-top: none;
  border-radius: 0 0 12px 12px;
  overflow: hidden;

  :deep(.metadata-container) {
    background: transparent;
    min-height: auto;
    padding: 0;

    .metadata-tabs {
      :deep(.ant-tabs-bar) {
        background: #f8fafc;
        margin: 0;
        border-radius: 0;
        border: none;
        // border-bottom: 1px solid #e8eaed;
      }
    }

    .tab-content {
      border: none;
      border-radius: 0;
      box-shadow: none;
    }
  }
}

/* 响应式设计 */
@media (max-width: 768px) {
  .product-details-container {
    padding: 0;
  }

  .page-header {
    padding: 16px 16px;
    flex-direction: column;
    gap: 16px;
    align-items: flex-start;
  }

  .product-overview-card {
    padding: 20px;

    .overview-header {
      flex-direction: column;
      align-items: flex-start;
      gap: 16px;

      .product-title h2 {
        font-size: 18px;
      }

      .product-key .key-value {
        font-size: 14px;
      }
    }

    .overview-info {
      grid-template-columns: 1fr;
      gap: 12px;

      .info-item {
        padding: 12px;
      }
    }
  }

  .info-card {
    margin-bottom: 12px;

    .card-header {
      padding: 12px 16px;
    }

    .card-content {
      padding: 16px;
    }
  }

  .config-list, .storage-list {
    gap: 8px;

    .config-item, .storage-item {
      padding: 10px 12px;
    }
  }

  .custom-params {
    gap: 8px;

    .param-item {
      padding: 12px;
    }
  }
}


@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateX(-20px);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

.product-overview-card,
.info-card {
  animation: slideIn 0.3s ease-out;
}

.info-card:nth-child(2) {
  animation-delay: 0.1s;
}

.info-card:nth-child(3) {
  animation-delay: 0.2s;
}

.info-card:nth-child(4) {
  animation-delay: 0.3s;
}

.info-card:nth-child(5) {
  animation-delay: 0.4s;
}

.info-card:nth-child(6) {
  animation-delay: 0.5s;
}

/* 响应式设计优化 */
@media (max-width: 768px) {
  .custom-tabs-nav {
    flex-wrap: nowrap;
    overflow-x: auto;
    padding: 0 4px;
  }

  .custom-tab-item {
    padding: 8px 12px;
    font-size: 12px;
    min-width: 80px;
    margin: 0 1px;
  }

  .custom-tab-item:first-child {
    margin-left: 4px;
  }

  .custom-tab-item:last-child {
    margin-right: 4px;
  }

  .tab-pane {
    padding: 16px;
  }

  .custom-tabs-container {
    margin: 0 16px;
  }

  .page-header {
    padding: 16px 16px;
  }
}

@media (max-width: 480px) {
  .custom-tab-item {
    padding: 10px 12px;
    font-size: 12px;
    min-width: 80px;
  }

  .tab-pane {
    padding: 12px;
  }
}
</style>
<script>

import {getProduct, modifyProductGateway, updateProduct} from '@/api/system/dev/product'
import {getInfo} from '@/api/system/dev/sort'
import CreateForm from './CreateForm'
import metadata from './metadata'
import configurationForm from './configurationForm'
import storageConfigurationForm from './storageConfigurationForm1'
import OtherConfigForm from './OtherConfigForm'
import productInstance from './productInstance'
import metadataShow from './metadataShow'
import importMetadata from './importMetadata'
import productProtocol from './productProtocol'
import SelectGateway from './SelectGateway'
import NetworkComponentBind from './NetworkComponentBind'
import {bindCertificate, getCertificateList, unbindCertificate} from '@/api/system/certificate'
import CertificateBind from './CertificateBind.vue'
import ConfigurationEditor from './ConfigurationEditor.vue'

export default {
  name: 'ProductDetails',
  components: {
    productInstance,
    CreateForm,
    configurationForm,
    metadataShow,
    importMetadata,
    metadata,
    productProtocol,
    OtherConfigForm,
    storageConfigurationForm,
    SelectGateway,
    NetworkComponentBind,
    CertificateBind,
    ConfigurationEditor
  },
  props: [],
  data() {
    return {
      otherConfiguration: [],
      // 导入物模型输入框是否显示
      importMetadataShow: false,
      // 物模型详情是否显示
      metadataShow: false,
      // 物模型显示类型 type === 1 IoT物模型；type === 2 的物模型
      metadataShowType: 1,
      // 物模型
      metadata: undefined,
      // 字典数据
      dicts: [],
      configurationDicts: [],
      publicConfigurationDicts: [],
      // 加载开关
      loading: false,
      // 配置信息数组
      configurationArray: [],
      // 公共配置信息数组
      publicConfigurationArray: [],
      // 存储策略显示数组
      storePolicyCfgArray: [],
      // 产品分组id
      productSortName: undefined,
      // 产品id
      productId: undefined,
      // 产品详情
      productDetails: {
        productId: undefined,
        productKey: undefined,
        productSecret: undefined,
        thirdPlatform: undefined,
        thirdConfiguration: undefined,
        companyNo: undefined,
        classifiedId: undefined,
        configuration: undefined,
        networkWay: undefined,
        deviceNode: undefined,
        projectName: undefined,
        projectId: undefined,
        classifiedName: undefined,
        messageProtocol: undefined,
        orgId: undefined,
        name: undefined,
        creatorId: undefined,
        describe: undefined,
        storePolicy: undefined,
        storePolicyConfiguration: undefined,
        transportProtocol: undefined,
        photoUrl: undefined,
        protocolName: undefined,
        metadata: undefined,
        state: undefined
      },
      activeKey: '1',
      // 设备标签字典
      productTags: [],
      // 标签选择
      tagList: [],
      // 是否编辑标签
      editTagStatus: false,
      // 网关标签
      selectedGateway: null,
      showCertModal: false,
      certList: [],
      certLoading: false,
      certPagination: {current: 1, pageSize: 10, total: 0},
      certColumns: [
        {title: '证书名称', dataIndex: 'name', align: 'center'},
        {title: '证书标识', dataIndex: 'sslKey', align: 'center'},
        {
          title: '到期时间',
          dataIndex: 'expireTime',
          scopedSlots: {customRender: 'expireTime'},
          align: 'center'
        }
      ],
      selectedCertId: null,
      bindCert: null,
      // 存储策略表格列定义
      storageColumns: [
        {
          title: '存储标识',
          dataIndex: 'key',
          scopedSlots: {customRender: 'key'},
          align: 'left'
        },
        {
          title: '存储限制',
          dataIndex: 'value',
          scopedSlots: {customRender: 'value'},
          align: 'left'
        },
        {
          title: '类型',
          dataIndex: 'type',
          scopedSlots: {customRender: 'type'},
          align: 'left'
        }
      ]
    }
  },
  created() {
    this.productId = this.$route.params.id
    console.log('this.productId', this.productId)
    // console.log('this.productId', this.productId)
    // 获取标签字典
    this.getDictMap(['product_tags']).then(res => {
      res.data['product_tags'].forEach((item) => {
        this.productTags.push({
          id: item.dictValue,
          name: item.dictLabel
        })
      })
    })
    this.getProductDeatils()
  },
  methods: {
    // 设备标签选择
    handleTagChange(value) {
      this.productDetails.tags = value.join()
    },
    // 编辑设备标签
    editTags() {
      this.editTagStatus = !this.editTagStatus
    },
    // 保存标签修改
    confirmTags() {
      const data = {
        id: this.productDetails.id,
        tags: this.productDetails.tags
      }
      updateProduct(data).then(res => {
        this.$message.success('修改成功', 3)
        this.editTagStatus = false
        this.getProductDeatils()
      })
    },
    /** 查询设备产品列表(无作用) */
    getList() {
    },
    color16() { // 十六进制颜色随机
      var colorList = ['#428BCA', '#2db7f5', '108ee9', '#87d068']
      var randomItem = colorList[Math.random() * colorList.length | 0]
      return randomItem
    },
    getProductTags(e) {
      const actions = []
      Object.keys(this.productTags).some((key) => {
        const tag = e ? e.split(',') : []
        tag.forEach((item) => {
          if (this.productTags[key].id === '' + item) {
            actions.push(this.productTags[key].name)
            return true
          }
        })
      })
      return actions
    },
    // 根据设备id获取设备详情及设备配置信息
    getProductDeatils() {
      console.log(this.$store.state)
      this.loading = true
      this.configurationDicts = []
      this.publicConfigurationDicts = []
      this.dicts = []
      this.configurationArray = []
      this.publicConfigurationArray = []
      this.storePolicyCfgArray = []
      getProduct(this.productId).then(response => {
        // console.log('response = ', JSON.parse(response.data.metadata))

        // 新的API结构：直接返回 gwName 和 gwProductKey
        if (response.data.gwProductKey && response.data.gwName) {
          this.selectedGateway = {
            name: response.data.gwName,
            productKey: response.data.gwProductKey
          }
        } else {
          this.selectedGateway = null
        }

        response.data.tagsName = this.getProductTags(response.data.tags)
        this.productDetails = response.data
        if (this.productDetails.tags) {
          this.tagList = this.productDetails.tags.split(',')
        }
        console.log('this.productDetails = ', this.productDetails)
        if (this.productDetails.thirdConfiguration) {
          var thirdCfg = JSON.parse(this.productDetails.thirdConfiguration)
          if (thirdCfg) {
            this.otherConfiguration = thirdCfg.customField
            console.log('this.otherConfiguration = ', this.otherConfiguration)
          }
        }
        let configArray = []
        let configData = {}
        if (this.productDetails.configuration !== undefined && this.productDetails.configuration
          !== null) {
          // 配置信息key数组
          configArray = Object.keys(JSON.parse(this.productDetails.configuration))
          // 配置信息key对象
          configData = JSON.parse(this.productDetails.configuration)
        }
        if (this.productDetails.storePolicyConfiguration) {
          const storeJson = JSON.parse(response.data.storePolicyConfiguration)
          const properties = storeJson.properties
          const eventsList = storeJson.event
          if (properties) {
            for (let i = 0; i < properties.length; i++) {
              this.storePolicyCfgArray.push({
                key: properties[i].id,
                value: properties[i].maxStorage,
                type: 'properties'
              })
            }
          }
          if (eventsList) {
            for (let j = 0; j < eventsList.length; j++) {
              this.storePolicyCfgArray.push({
                key: eventsList[j].id,
                value: eventsList[j].maxStorage,
                type: 'events'
              })
            }
          }
        }
        getInfo({id: this.productDetails.classifiedId}).then(response => {
          if (response.data !== undefined && response.data !== null) {
            this.productSortName = response.data.classifiedName
          }
        })

        // 获取公共配置和私有配置
        var dictArray = []
        dictArray.push('product_configuration_' + this.productDetails.thirdPlatform)
        dictArray.push('product_configuration_public')

        this.getDictMap(dictArray).then(res => {
          // 私有配置
          if (res.data['product_configuration_' + this.productDetails.thirdPlatform] !== undefined
            && res.data['product_configuration_' + this.productDetails.thirdPlatform] !== null) {
            this.configurationDicts = res.data['product_configuration_'
            + this.productDetails.thirdPlatform]
            if (this.configurationDicts.length > 0) {
              if (configArray !== undefined) {
                for (let j = 0; j < this.configurationDicts.length; j++) {
                  let config = {}
                  try {
                    if (this.configurationDicts[j].remark !== undefined ||
                      this.configurationDicts[j].remark !== null ||
                      this.configurationDicts[j].remark !== '') {
                      config = JSON.parse(this.configurationDicts[j].remark)
                    }
                  } catch (e) {
                    config = {}
                  }
                  for (let i = 0; i < configArray.length; i++) {
                    if (configArray[i] === this.configurationDicts[j].dictValue) {
                      this.configurationArray.push({
                        key: this.configurationDicts[j].dictLabel,
                        value: configData[configArray[i]],
                        remark: config.remark,
                        type: config.type
                      })
                      break
                    }
                  }
                  if (this.configurationArray[j] === undefined || this.configurationArray[j]
                    === null) {
                    this.configurationArray.push({
                      key: this.configurationDicts[j].dictLabel,
                      value: undefined,
                      remark: config.remark,
                      type: config.type
                    })
                  }
                }
              } else {
                for (let j = 0; j < this.configurationDicts.length; j++) {
                  let config = {}
                  try {
                    if (this.configurationDicts[j].remark !== undefined ||
                      this.configurationDicts[j].remark !== null ||
                      this.configurationDicts[j].remark !== '') {
                      config = JSON.parse(this.configurationDicts[j].remark)
                    }
                  } catch (e) {
                    config = {}
                  }
                  this.configurationArray.push({
                    key: this.configurationDicts[j].dictLabel,
                    value: undefined,
                    remark: config.remark,
                    type: config.type
                  })
                }
              }
            }
            console.log('this.configurationArray', this.configurationArray)
          }

          // 公共配置
          this.publicConfigurationDicts = res.data['product_configuration_public']
          if (this.publicConfigurationDicts.length > 0) {
            if (configArray) {
              for (let j = 0; j < this.publicConfigurationDicts.length; j++) {
                for (let i = 0; i < configArray.length; i++) {
                  if (configArray[i] === this.publicConfigurationDicts[j].dictValue) {
                    this.publicConfigurationArray.push({
                      key: this.publicConfigurationDicts[j].dictLabel,
                      value: configData[configArray[i]],
                      remark: this.publicConfigurationDicts[j].remark
                    })
                    break
                  }
                }
                if (this.publicConfigurationArray[j] === undefined
                  || this.publicConfigurationArray[j] === null) {
                  this.publicConfigurationArray.push({
                    key: this.publicConfigurationDicts[j].dictLabel,
                    value: undefined,
                    remark: this.publicConfigurationDicts[j].remark
                  })
                }
              }
            } else {
              for (let j = 0; j < this.publicConfigurationDicts.length; j++) {
                this.publicConfigurationArray.push({
                  key: this.publicConfigurationDicts[j].dictLabel,
                  value: undefined,
                  remark: this.publicConfigurationDicts[j].remark
                })
              }
            }
          }
        })
      }).finally(() => {
        this.loading = false
      })
    },
    // 返回上一级
    back() {
      this.$router.back()
    },
    // 修改配置
    editConfig(type) {
      if (type === 'publicConfiguration') {
        this.dicts = this.publicConfigurationDicts
        this.$refs.configurationForm.handleUpdateById(this.productId, 'publicConfiguration')
      } else if (type === 'privateConfiguration') {
        console.log('this.configurationDicts +++++ ', this.configurationDicts)
        this.dicts = this.configurationDicts
        this.$refs.configurationForm.handleUpdateById(this.productId, 'privateConfiguration')
      } else if (type === 'otherConfiguration') {
        this.dicts = this.configurationDicts
        this.$refs.otherConfigurationForm.handleUpdateAndAdd(this.productId)
      } else if (type === 'storageConfiguration') {
        this.$refs.storageConfigurationForm.handleUpdateById(this.productId)
      }
    },
    // 打开物模型详情
    openMetadataDetail() {
      if (this.metadataShow) {
        this.metadataShowType = 1
        this.metadataShow = false
      } else {
        this.metadataShowType = 1
        this.metadataShow = true
      }
    },
    // 关闭物模型详情
    closeMetadataDetail() {
      this.metadataShow = false
    },
    // 打开导入物模型
    openImportMetadata() {
      if (this.importMetadataShow) {
        this.importMetadataShow = false
      } else {
        this.importMetadataShow = true
      }
    },
    // 关闭导入物模型
    closeImportMetadata() {
      this.importMetadataShow = false
    },
    // 成功更新物模型
    successUpdateMetadata() {
      this.importMetadataShow = false
      this.$refs.metadata.refresMetadata()
    },

    // 存储策略配置成功
    storageConfigOk() {
      console.log('存储策略配置成功')
    },

    // 网关相关
    openGatewayModal() {
      this.$refs.selectGateway.showModal()
    },
    addGateway(val) {
      this.selectedGateway = val
      this.modifyGateway()
    },
    removeGateway() {
      this.$confirm({
        title: '提示',
        content: '确认删除网关关联?',
        onOk: () => {
          this.selectedGateway = null
          this.modifyGateway()
        }
      })
    },
    modifyGateway() {
      const gwProductKey = this.selectedGateway ? this.selectedGateway.productKey : ''
      modifyProductGateway(this.productId, gwProductKey)
      .then(() => {
        this.$message.success('操作成功!')
      })
    },
    // 搜索|过滤产品
    filterOption(value, option) {
      return (
        option.componentOptions.children[0].children[0].text.toLowerCase().indexOf(
          value.toLowerCase()) >= 0
      )
    },
    // 复制到剪贴板
    copyToClipboard(text) {
      if (navigator.clipboard) {
        navigator.clipboard.writeText(text).then(() => {
          this.$message.success('已复制到剪贴板')
        }).catch(() => {
          this.fallbackCopyTextToClipboard(text)
        })
      } else {
        this.fallbackCopyTextToClipboard(text)
      }
    },
    // 兜底复制方法
    fallbackCopyTextToClipboard(text) {
      const textArea = document.createElement('textarea')
      textArea.value = text
      document.body.appendChild(textArea)
      textArea.focus()
      textArea.select()
      try {
        document.execCommand('copy')
        this.$message.success('已复制到剪贴板')
      } catch (err) {
        this.$message.error('复制失败')
      }
      document.body.removeChild(textArea)
    },
    // 获取平台颜色
    getPlatformColor(platform) {
      const colorMap = {
        'universal': '#1890ff',
        'aliyun': '#fa8c16',
        'tencent': '#722ed1',
        'huawei': '#eb2f96',
        'baidu': '#13c2c2'
      }
      return colorMap[platform] || '#1890ff'
    },
    // 获取设备节点文本
    getDeviceNodeText(deviceNode) {
      const nodeMap = {
        'DEVICE': '直连设备',
        'GATEWAY': '网关设备',
        'GATEWAY_SUB_DEVICE': '网关子设备'
      }
      return nodeMap[deviceNode] || deviceNode || '-'
    },
    loadBindCert() {
      // 假设产品详情有bindCertKey字段
      if (this.productDetails.bindCertKey) {
        getCertificateList({sslKey: this.productDetails.bindCertKey}).then(res => {
          if (res.rows && res.rows.length) {
            this.bindCert = res.rows[0]
          } else {
            this.bindCert = null
          }
        })
      } else {
        this.bindCert = null
      }
    },
    loadCertList(page = 1, size = 10) {
      this.certLoading = true
      getCertificateList({pageNum: page, pageSize: size}).then(res => {
        this.certList = res.rows || []
        this.certPagination.total = res.total || 0
        this.certPagination.current = page
        this.certPagination.pageSize = size
      }).finally(() => {
        this.certLoading = false
      })
    },
    onCertSelect(selectedRowKeys) {
      this.selectedCertId = selectedRowKeys[0]
    },
    handleBind() {
      const cert = this.certList.find(c => c.id === this.selectedCertId)
      if (!cert) {
        return
      }
      bindCertificate(this.productDetails.productKey, cert.sslKey).then(res => {
        this.$message.success('绑定成功')
        this.showCertModal = false
        this.loadBindCert()
      })
    },
    handleUnbind() {
      unbindCertificate(this.productDetails.productKey).then(res => {
        this.$message.success('解绑成功')
        this.loadBindCert()
      })
    },
    parseTime(val) {
      if (!val) {
        return ''
      }
      const d = new Date(val)
      if (isNaN(d.getTime())) {
        return val
      }
      const pad = n => n < 10 ? '0' + n : n
      return `${d.getFullYear()}-${pad(d.getMonth() + 1)}-${pad(d.getDate())} ${pad(
        d.getHours())}:${pad(d.getMinutes())}:${pad(d.getSeconds())}`
    },
    getSslKeyFromConfig(config) {
      if (!config) {
        return ''
      }
      try {
        const obj = typeof config === 'string' ? JSON.parse(config) : config
        return obj.sslKey || ''
      } catch (e) {
        return ''
      }
    }
  },
  watch: {
    activeKey(val) {
      if (val === '5') {
        this.loadBindCert()
        this.loadCertList()
      }
    }
  }
}
</script>

<style lang="less" scoped>
.action-buttons {
  display: flex;
  justify-content: flex-end;
  margin-bottom: 16px;

  .ant-space {
    .ant-btn {
      height: 28px;
      padding: 0 12px;
      font-size: 12px;
      border-radius: 4px;

      &.ant-btn-primary {
        background: #1890ff;
        border-color: #1890ff;

        &:hover {
          background: #40a9ff;
          border-color: #40a9ff;
        }
      }

      &.ant-btn-default {
        border-color: #d9d9d9;
        color: #666;

        &:hover {
          border-color: #40a9ff;
          color: #40a9ff;
        }
      }
    }
  }
}

.bind-cert-header {
  display: flex;
  align-items: center;
  font-size: 15px;
  margin-bottom: 12px;
}

.gateway-info {
  margin-bottom: 16px;

  .selected-gateway {
    margin-bottom: 8px;

    .gateway-tag {
      font-size: 14px;
      padding: 4px 8px;
      border-radius: 4px;
    }
  }

  .no-gateways {
    color: #999;
    font-style: italic;
  }
}
</style>
