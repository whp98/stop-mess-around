<!--
 * Author       : OBKoro1
 * Date         : 2021-05-25 15:18:00
 * LastEditors  : OBKoro1
 * LastEditTime : 2022-03-13 17:49:40
 * FilePath     : /stop-mess-around/src/options/App/settingPage/CreateItem.vue
 * Description  : 新增摸鱼网站
 * koroFileheader插件
 * Copyright (c) 2021 by OBKoro1, All Rights Reserved.
-->
<template>
  <el-dialog
    :close-on-click-modal="false"
    :close-on-press-escape="false"
    :visible.sync="dialogVisible"
    :show-close="false"
    :before-close="resetFields"
    width="550px"
  >
    <div
      slot="title"
      class="dialog_title"
    >
      {{ '新增摸鱼网站' }}
    </div>
    <div class="form-class">
      <el-form
        ref="ruleForm"
        :model="ruleForm"
        :rules="rules"
        label-width="150px"
      >
        <el-form-item
          prop="labelName"
          required
        >
          <span slot="label">{{ '摸鱼网站名' }}: </span>
          <el-input
            v-model="ruleForm.labelName"
            class="input-class"
            :placeholder="'添加摸鱼网站名'"
          />
        </el-form-item>
        <el-form-item
          prop="site"
          required
        >
          <span slot="label">{{ '摸鱼网址' }}: </span>
          <el-input
            v-model="ruleForm.site"
            class="input-class"
            autosize
            type="textarea"
            :placeholder="'添加摸鱼网址(唯一)'"
          />
        </el-form-item>
        <el-form-item prop="tip">
          <span slot="label">
            <el-tooltip
              :content="'匹配到摸鱼网站时显示的励志语录'"
              placement="top"
            >
              <span>{{ '励志语录' }}</span>
            </el-tooltip>
          </span>
          <el-input
            v-model="ruleForm.tip"
            class="input-class"
            :placeholder="'为空则为随机励志语录'"
          />
        </el-form-item>
        <el-form-item prop="time">
          <span slot="label">
            <el-tooltip
              :content="'检测到摸鱼网址后，停留几秒关闭网页'"
              placement="top"
            >
              <span>{{ '停留几秒' }}</span>
            </el-tooltip>
          </span>
          <el-input
            v-model.number="ruleForm.time"
            :placeholder="'为空则为全局设置的停留时间'"
            autocomplete="off"
          />
        </el-form-item>
        <el-form-item prop="checkoutStudy">
          <span slot="label">
            <el-tooltip
              :content="'关闭检测后多少分钟后重新启用检测'"
              placement="top"
            >
              <span>{{ '自动开启' }}</span>
            </el-tooltip>
          </span>
          <el-input
            v-model.number="ruleForm.checkoutStudy"
            :placeholder="'为空则为全局设置的自动开启时间'"
            autocomplete="off"
          />
        </el-form-item>
        <el-form-item
          prop="matchRule"
          required
        >
          <span slot="label">
            <el-tooltip
              :content="'网址匹配规则: 开头全等 => 前面的必须一模一样，包含=> 包含即可，严格相等=>一模一样'"
              placement="top"
            >
              <span>{{ '匹配规则' }}</span>
            </el-tooltip>
          </span>
          <el-radio
            v-model="ruleForm.matchRule"
            label="start"
          >
            {{ '开头全等' }}
          </el-radio>
          <el-radio
            v-model="ruleForm.matchRule"
            label="includes"
          >
            {{ '包含' }}
          </el-radio>
          <el-radio
            v-model="ruleForm.matchRule"
            label="strict"
          >
            {{ '严格相等' }}
          </el-radio>
        </el-form-item>
        <el-form-item
          prop="jump"
          required
        >
          <span slot="label">
            <el-tooltip
              :content="'网址匹配后，不关闭页面，跳转到其他页面'"
              placement="top"
            >
              <span>{{ '跳转' }}</span>
            </el-tooltip>
          </span>
          <el-radio
            v-model="ruleForm.jump"
            :label="true"
          >
            {{ '开启' }}
          </el-radio>
          <el-radio
            v-model="ruleForm.jump"
            :label="false"
          >
            {{ '关闭' }}
          </el-radio>
        </el-form-item>
        <el-form-item
          v-if="ruleForm.jump"
          prop="jumpUrl"
        >
          <span slot="label">
            <el-tooltip
              :content="'为空则使用全局跳转链接'"
              placement="top"
            >
              <span>{{ '关闭页面跳转链接' }}</span>
            </el-tooltip>
          </span>
          <el-input
            v-model="ruleForm.jumpUrl"
            class="input-class"
            :placeholder="'为空则使用全局跳转链接'"
          />
        </el-form-item>
      </el-form>
    </div>
    <div
      slot="footer"
      class="dialog-footer"
    >
      <el-button @click="close">
        取 消
      </el-button>
      <el-button
        type="primary"
        @click="confirmFn"
      >
        添 加
      </el-button>
    </div>
  </el-dialog>
</template>

<script>

export default {
  name: 'CreateItem',
  props: {
    showCreateItem: {
      require: true,
      type: Boolean,
      default: false,
    },
  },
  inject: ['tableDataSpliceUpdate', 'getSetting', 'getTableData', 'settingUpdate'],
  data() {
    return {
      Setting: {},
      ruleForm: {
        labelName: '', // 摸鱼网站名字
        site: '', // 摸鱼网站地址
        matchRule: 'start', // 匹配规则 start/strict/includes
        time: undefined, // 是否立即关闭摸鱼网站
        checkoutStudy: undefined, // 定时自动开启
        jump: false, // 检测到摸鱼网址后 跳转到哪个页面
        jumpUrl: '', // 跳转页面
        open: true, // 是否检测该摸鱼网站
        tip: '', // 内卷提示
        closeTime: 0, // 啥时候关闭
      },
      rules: {
        labelName: [
          { required: true, message: '请输入摸鱼网站名', trigger: 'blur' },
        ],
        site: [
          { required: true, message: '请输入摸鱼网址', trigger: 'blur' },
          {
            min: 8, message: '至少8个字符', trigger: 'blur',
          },
        ],
        tip: [
          { message: '请输入励志语录', trigger: 'blur' },
        ],
        jumpUrl: [
          { required: true, message: '请输入跳转网址', trigger: 'blur' },
          {
            min: 8, message: '至少8个字符', trigger: 'blur',
          },
        ],
      },
    }
  },
  computed: {
    dialogVisible: {
      get() {
        return this.showCreateItem
      },
      set() {
        this.close()
      },
    },
  },
  watch: {
    // 初始化数据
    dialogVisible(val) {
      if (val) {
        this.initRuleForm()
      }
    },
  },
  mounted() {
    setTimeout(() => {
      this.openCreate()
    }, 500)
  },
  methods: {
    // 将全局设置放入新增中
    initRuleForm() {
      this.Setting = this.getSetting()
      Object.keys(this.Setting).forEach((key) => {
        // 不赋值 使用全局 全局变更 跟着变更
        if (key === 'tip') return
        if (key === 'time') return
        if (key === 'checkoutStudy') return
        // 使用设置
        if (this.ruleForm[key] !== undefined) {
          this.ruleForm[key] = this.Setting[key]
        }
      })
    },
    // popup添加摸鱼网址
    openCreate() {
      this.Setting = this.getSetting()
      if (this.Setting.addSite !== '') {
        this.$emit('close', 'showCreateItem', true)
        this.ruleForm.site = this.Setting.addSite
        // 重置摸鱼网站
        this.Setting.addSite = ''
        this.settingUpdate(this.Setting)
      }
    },
    // 摸鱼网站是否添加过
    hasAdd(arr) {
      return arr.find((item) => item.site === this.ruleForm.site)
    },
    confirmFn() {
      this.$refs.ruleForm.validate((valid) => {
        if (valid) {
          const arr = this.getTableData()
          if (this.hasAdd(arr) === undefined) {
            // 添加到第一个
            const obj = JSON.parse(JSON.stringify(this.ruleForm))
            // 添加到末尾
            this.tableDataSpliceUpdate(arr.length, 0, obj)
            // 判断
            this.close()
          } else {
            this.$message.error('摸鱼网址重复, 请重新输入')
          }
        } else {
          return false
        }
      })
    },
    close() {
      this.ruleForm.site = ''
      this.$refs.ruleForm.resetFields()
      this.$emit('close', 'showCreateItem', false)
    },
    resetFields(done) {
      this.$refs.ruleForm.resetFields()
      done()
    },
  },
}
</script>

<style scoped>
.form-class {
  width: 90%;
}
</style>>
