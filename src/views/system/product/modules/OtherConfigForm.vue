<template>
  <!-- 增加修改 -->
  <a-modal
    width="900px"
    :title="formTitle"
    :visible="open"
    @cancel="cancel"
    @ok="submitForm"
    :maskClosable="false"
  >
    <a-form-model ref="form" :model="form">
      <a-tooltip>
        <template slot="title">
          添加设备所需要用到的额外参数
        </template>
        <a-button type="primary" @click="addParams" style="margin-bottom: 30px">新增参数</a-button>
      </a-tooltip>

      <template v-for="(item , index) in form.arr">
        <a-row v-if="index === 0" :key="index">
          <a-col :span="6" style="margin-right: 10px">
            <a-tooltip>
              <template slot="title">
                对应后端字段(一般为英文字母组合)
              </template>
              <a-form-model-item
                label="英文标识"
                prop="ext1Id"
                :rules="{required: true , message: '英文标识' + '不能为空', trigger: 'blur'}">
                <a-input v-model="form.ext1Id" placeholder="英文标识"/>
              </a-form-model-item>
            </a-tooltip>
          </a-col>
          <a-col :span="6" style="margin-right: 10px">
            <a-tooltip>
              <template slot="title">
                中文名称，前端提示的名称
              </template>
              <a-form-model-item
                label="中文名称"
                prop="ext1Name"
                :rules="{required: true , message: '中文名称' + '不能为空', trigger: 'blur'}">
                <a-input v-model="form.ext1Name" placeholder="中文名称"/>
              </a-form-model-item>
            </a-tooltip>
          </a-col>
          <a-col :span="7" style="margin-right: 10px">
            <a-tooltip>
              <template slot="title">
                详细描述，如：输入规则、输入位数等等
              </template>
              <a-form-model-item
                label="详细描述"
                prop="ext1Des"
                :rules="{required: true , message: '详细描述' + '不能为空', trigger: 'blur'}">
                <a-input v-model="form.ext1Des" placeholder="详细描述"/>
              </a-form-model-item>
            </a-tooltip>
          </a-col>
          <a-col :span="4" style="margin-top: 44px">
            <a-button type="primary" @click="deleteParams(index)">删除</a-button>
          </a-col>
        </a-row>
        <a-row v-if="index === 1" :key="index">
          <a-col :span="6" style="margin-right: 10px">
            <a-tooltip>
              <template slot="title">
                对应后端字段(一般为英文字母组合)
              </template>
              <a-form-model-item
                label="英文标识"
                prop="ext2Id"
                :rules="{required: true , message: '英文标识' + '不能为空', trigger: 'blur'}">
                <a-input v-model="form.ext2Id" placeholder="英文标识"/>
              </a-form-model-item>
            </a-tooltip>
          </a-col>
          <a-col :span="6" style="margin-right: 10px">
            <a-tooltip>
              <template slot="title">
                中文标识，前端提示的名称
              </template>
              <a-form-model-item
                label="中文标识"
                prop="ext2Name"
                :rules="{required: true , message: '中文标识' + '不能为空', trigger: 'blur'}">
                <a-input v-model="form.ext2Name" placeholder="中文标识"/>
              </a-form-model-item>
            </a-tooltip>
          </a-col>
          <a-col :span="7" style="margin-right: 10px">
            <a-tooltip>
              <template slot="title">
                详细描述，如：输入规则、输入位数等等
              </template>
              <a-form-model-item
                label="详细描述"
                prop="ext2Des"
                :rules="{required: true , message: '详细描述' + '不能为空', trigger: 'blur'}">
                <a-input v-model="form.ext2Des" placeholder="详细描述"/>
              </a-form-model-item>
            </a-tooltip>
          </a-col>
          <a-col :span="4" style="margin-top: 44px">
            <a-button type="primary" @click="deleteParams(index)">删除</a-button>
          </a-col>
        </a-row>
        <a-row v-if="index === 2" :key="index">
          <a-col :span="6" style="margin-right: 10px">
            <a-tooltip>
              <template slot="title">
                对应后端字段(一般为英文字母组合)
              </template>
              <a-form-model-item
                label="英文标识"
                prop="ext3Id"
                :rules="{required: true , message: '英文标识' + '不能为空', trigger: 'blur'}">
                <a-input v-model="form.ext3Id" placeholder="英文标识"/>
              </a-form-model-item>
            </a-tooltip>
          </a-col>
          <a-col :span="6" style="margin-right: 10px">
            <a-tooltip>
              <template slot="title">
                中文标识，前端提示的名称
              </template>
              <a-form-model-item
                label="中文标识"
                prop="ext3Name"
                :rules="{required: true , message: '中文标识' + '不能为空', trigger: 'blur'}">
                <a-input v-model="form.ext3Name" placeholder="中文标识"/>
              </a-form-model-item>
            </a-tooltip>
          </a-col>
          <a-col :span="7" style="margin-right: 10px">
            <a-tooltip>
              <template slot="title">
                详细描述，如：输入规则、输入位数等等
              </template>
              <a-form-model-item
                label="详细描述"
                prop="ext3Des"
                :rules="{required: true , message: '详细描述' + '不能为空', trigger: 'blur'}">
                <a-input v-model="form.ext3Des" placeholder="中文标识"/>
              </a-form-model-item>
            </a-tooltip>
          </a-col>
          <a-col :span="4" style="margin-top: 44px">
            <a-button type="primary" @click="deleteParams(index)">删除</a-button>
          </a-col>
        </a-row>
        <a-row v-if="index === 3" :key="index">
          <a-col :span="6" style="margin-right: 10px">
            <a-tooltip>
              <template slot="title">
                对应后端字段(一般为英文字母组合)
              </template>
              <a-form-model-item
                label="英文标识"
                prop="ext4Id"
                :rules="{required: true , message: '英文标识' + '不能为空', trigger: 'blur'}">
                <a-input v-model="form.ext4Id" placeholder="英文标识"/>
              </a-form-model-item>
            </a-tooltip>
          </a-col>
          <a-col :span="6" style="margin-right: 10px">
            <a-tooltip>
              <template slot="title">
                中文标识，前端提示的名称
              </template>
              <a-form-model-item
                label="中文标识"
                prop="ext4Name"
                :rules="{required: true , message: '中文标识' + '不能为空', trigger: 'blur'}">
                <a-input v-model="form.ext4Name" placeholder="中文标识"/>
              </a-form-model-item>
            </a-tooltip>
          </a-col>
          <a-col :span="7" style="margin-right: 10px">
            <a-tooltip>
              <template slot="title">
                详细描述，如：输入规则、输入位数等等
              </template>
              <a-form-model-item
                label="详细描述"
                prop="ext4Des"
                :rules="{required: true , message: '详细描述' + '不能为空', trigger: 'blur'}">
                <a-input v-model="form.ext4Des" placeholder="中文标识"/>
              </a-form-model-item>
            </a-tooltip>
          </a-col>
          <a-col :span="4" style="margin-top: 44px">
            <a-button type="primary" @click="deleteParams(index)">删除</a-button>
          </a-col>
        </a-row>
      </template>

      <div class="bottom-control">
        <a-space>
          <a-button type="primary" @click="submitForm">
            保存
          </a-button>
          <a-button type="dashed" @click="cancel">
            取消
          </a-button>
        </a-space>
      </div>
    </a-form-model>
  </a-modal>
</template>

<script>
import {getProduct, updateOtherConfig} from '@/api/system/dev/product'

export default {
  name: 'OtherConfigForm',
  props: {},
  components: {},
  data() {
    return {
      // 标题
      formTitle: '修改产品其他配置信息',
      // 显示控制
      open: false,
      // 表单参数
      form: {
        ext1Id: undefined,
        ext1Name: undefined,
        ext1Des: undefined,
        ext2Id: undefined,
        ext2Name: undefined,
        ext2Des: undefined,
        ext3Id: undefined,
        ext3Name: undefined,
        ext3Des: undefined,
        ext4Id: undefined,
        ext4Name: undefined,
        ext4Des: undefined,
        productId: undefined,
        arr: []
      }
    }
  },
  filters: {},
  created() {
  },
  computed: {},
  watch: {},
  mounted() {
  },
  methods: {
    // 取消按钮
    cancel() {
      this.open = false
      this.reset()
    },
    // 表单重置
    reset() {
      this.form = {
        ext1Id: undefined,
        ext1Name: undefined,
        ext1Des: undefined,
        ext2Id: undefined,
        ext2Name: undefined,
        ext2Des: undefined,
        ext3Id: undefined,
        ext3Name: undefined,
        ext3Des: undefined,
        ext4Id: undefined,
        ext4Name: undefined,
        ext4Des: undefined,
        productId: undefined,
        arr: []
      }
    },
    /** 修改按钮操作 */
    handleUpdateAndAdd(id) {
      this.reset()
      this.$nextTick(() => {
        try {
          this.$refs.form.resetFields()
        } catch (e) {

        }
      })
      getProduct(id).then(res => {
        console.log('res = ', res)
        if (res.data.thirdConfiguration !== undefined) {
          const str = JSON.parse(res.data.thirdConfiguration)
          if (str.customField !== undefined) {
            this.form.arr = str.customField
            for (let i = 0; i < str.customField.length; i++) {
              this.form[`ext` + (i + 1) + `Id`] = str.customField[i].id
              this.form[`ext` + (i + 1) + `Name`] = str.customField[i].name
              this.form[`ext` + (i + 1) + `Des`] = str.customField[i].description
            }
          }
        }
      })
      this.form.productId = id
      this.open = true
    },
    /** 提交按钮 */
    submitForm: function () {
      var that = this
      this.$refs.form.validate(valid => {
        if (valid) {
          updateOtherConfig(this.form).then((res) => {
            console.log('res = ', res)
            that.$message.success(
              '修改成功',
              3
            )
            this.open = false
            this.$emit('ok')
          })
        } else {
          return false
        }
      })
    },
    // 删除
    deleteParams(index) {
      if (index === 0) {
        this.form.ext1Id = this.form.ext2Id
        this.form.ext1Name = this.form.ext2Name
        this.form.ext1Des = this.form.ext2Des
        this.form.ext2Id = this.form.ext3Id
        this.form.ext2Name = this.form.ext3Name
        this.form.ext2Des = this.form.ext3Des
        this.form.ext3Id = this.form.ext4Id
        this.form.ext3Name = this.form.ext4Name
        this.form.ext3Des = this.form.ext4Des
      } else if (index === 1) {
        this.form.ext2Id = this.form.ext3Id
        this.form.ext2Name = this.form.ext3Name
        this.form.ext2Des = this.form.ext3Des
        this.form.ext3Id = this.form.ext4Id
        this.form.ext3Name = this.form.ext4Name
        this.form.ext3Des = this.form.ext4Des
      } else if (index === 2) {
        this.form.ext3Id = this.form.ext4Id
        this.form.ext3Name = this.form.ext4Name
        this.form.ext3Des = this.form.ext4Des
      }
      this.form.ext4Id = undefined
      this.form.ext4Name = undefined
      this.form.ext4Des = undefined
      this.form.arr.splice(index, 1)
    },
    // 增加参数
    addParams() {
      if (this.form.arr.length >= 4) {
        this.$message.error(`最多只能设置 4 个额外参数!`)
        return
      }
      this.form.arr.push({id: undefined, name: undefined, description: undefined})
      console.log(this.form)
    }
  }
}
</script>
