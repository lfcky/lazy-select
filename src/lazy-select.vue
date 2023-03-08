<!--懒加载下拉框，初始只加载一定数量的选项，滚动条下拉到最底部时追加选项-->
<template>
  <el-select v-model="value" :multiple="multiple" filterable clearable :placeholder="placeholder"
             :filter-method="onFilter" ref="select" @change="onChange" @clear="onFilter()" @visible-change="onVisibleChange">
    <el-option
      v-for="item of showData"
      :key="item[props['value']]"
      :label="item[props['label']]"
      :value="item[props['value']]">
    </el-option>
  </el-select>
</template>

<script>
export default {
  model: {
    prop: 'value',
    event: 'change'
  },
  props: {
    multiple: {
      type: Boolean,
      default: false
    },
    placeholder: '',
    value: {
      required: true
    },
    options: [],
    props: {
      default: {
        value: 'value',
        label: 'label'
      }
    },
    pageSize: {
      type: Number,
      default: 10
    }
  },
  watch: {
    options () {
      // this.reset()
    },
    value () {
      this.supplementOption()
    }
  },
  data () {
    return {
      scrollEle: null,
      showData: [],
      pageIndex: 1,
      searchContent: '',
      pageLock: false // 翻页锁定，数据加载完成前不许再翻页
    }
  },
  mounted () {
    // 滚动到底后触发翻页
    this.scrollEle = this.$refs.select.$el.querySelector('.el-select-dropdown .el-select-dropdown__wrap')
    this.scrollEle.addEventListener('scroll', () => {
      if (this.checkBottom()) {
        this.loadMore()
      }
    })
  },
  methods: {
    // 下拉框出现/隐藏
    onVisibleChange (show) {
      if (show) {
        this.reset()
      } else {
        this.searchContent = ''
      }
    },
    // 选中值改变
    onChange () {
      this.$emit('change', this.value)
    },
    // 搜索触发
    onFilter (val) {
      this.searchContent = val || ''
      this.reset()
    },
    // 重置，搜索内容改变或列表更改时重置
    reset () {
      this.pageIndex = 0
      this.loadMore()
    },
    // 检查滚动条是否触底
    checkBottom () {
      return this.scrollEle.scrollHeight - this.scrollEle.scrollTop === this.scrollEle.clientHeight
    },
    // 翻页
    loadMore () {
      this.pageIndex++
      this.getData()
    },
    // 加载数据
    getData () {
      let c = 0
      const total = this.pageSize * this.pageIndex
      this.showData = this.options.filter((item) => {
        item._created = false
        // 超出显示数量则不再加载
        if (c >= total) return false
        if (~String(item[this.props['label']]).indexOf(this.searchContent)) {
          // 符合条件则标记已创建
          item._created = true
          c++
          return true
        }
        return false
      })
      // 若已选择的选项未被创建则增补
      this.supplementOption()
    },
    // 根据key搜索option
    getOption (key) {
      return this.options ? this.options.find((item) => item[this.props['value']] === key) : null
    },
    // 补充未被创建的option
    supplementOption () {
      const list = this.multiple ? this.value : [this.value]
      for (const key of list) {
        let option = this.getOption(key)
        if (option && !option._created) {
          option._created = true
          this.showData.push(option)
        }
      }
    }
  }
}
</script>

<style scoped>

</style>
