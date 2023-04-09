<template>
  <div id='app'>
    <!--添加搜索和重置功能-->
    <!-- add search and reset functionality -->
    <div style='margin-bottom: 14px;'>
      <a-input placeholder='Enter a task or keyword' class='my_ipt' v-model='searchValue' />
      <a-button type='primary' @click='searchList'>Research</a-button>
      <a-button style='margin-left: 10px;' @click='searchReset'>Reset</a-button>
    </div>
    <!-- 双向数据绑定  v-model-->
    <!-- Two-way data binding v-model-->
    <div>
      <a-input placeholder='Enter the task you will complete' class='my_ipt' v-model='inputValue' />
      <a-button type='primary' @click='addList'>Add</a-button>
    </div>
    <!-- 双向数据绑定视图显示改变-->
    <!-- Two-way data binding view shows changes -->
    <div style='margin-left: 10px;'>{{ inputValue }}</div>

    <a-list bordered :dataSource='infoList' class='dt_list'>
      <a-list-item slot='renderItem' slot-scope='item'>
        <a-checkbox :checked='item.done' @change='cbStateChanged(item.id, $event)'>{{ item.info }}</a-checkbox>
        <!-- 删除链接 -->
        <!--delete link-->
        <a slot='actions' @click='delList(item.id)'>Delete</a>
      </a-list-item>

      <!-- footer区域 -->
      <!--footer part-->
      <div slot='footer' class='footer'>
        <!-- 未完成的任务个数 -->
        <!--The number of undone things-->
        <span>Number of remaining events:{{ unDoneLength }}</span>
        <!-- 操作按钮 -->
        <!--button-->
        <a-button-group>
          <a-button :type="viewKey === 'all' ? 'primary' : 'default'" @click="changeList('all')">All things</a-button>
          <a-button :type="viewKey === 'undone' ? 'primary' : 'default'" @click="changeList('undone')">Undone</a-button>
          <a-button :type="viewKey === 'done' ? 'primary' : 'default'" @click="changeList('done')">Done</a-button>
        </a-button-group>
        <!-- 把已经完成的任务清空 -->
        <!--Delete all done things-->
        <a @click='clean'>The cleanup is complete.</a>
      </div>
    </a-list>
  </div>
</template>

<script>

// Vue 的使用diff 算法，直接更新需要改变的地方，不需要全部改动
// Vue uses the diff algorithm to directly update what needs to be changed, without changing everything
import axios from 'axios'

export default {
  name: 'app',
  data () {
    return {
      list: [],
      inputValue: '',
      searchValue: '',
      nextId: 5,
      viewKey: 'all'
    }
  },
  created () {
    this.getList()
  },
  methods: {
    searchReset () {
      this.searchValue = ''
      this.getList()
    },
    searchList () {
      this.list = this.list.filter(item => item.info.includes(this.searchValue))
    },
    addList () {
      if (this.inputValue.trim().length <= 0) {
        return this.$message.warning('The text box input cannot be empty.')
      }
      const obj = {
        id: this.nextId,
        info: this.inputValue.trim(),
        done: false
      }
      this.list.push(obj)
      this.nextId++
      this.inputValue = ''
    },
    delList (id) {
      const index = this.list.findIndex(x => x.id === id)
      if (index !== -1) {
        this.list.splice(index, 1)
      }
    },
    cbStateChanged (id, e) {
      const param = {
        id: id,
        status: e.target.checked
      }
      const index = this.list.findIndex(x => x.id === param.id)
      if (index !== -1) {
        this.list[index].done = param.status
      }
    },
    clean () {
      this.list = this.list.filter(x => x.done === false)
    },
    changeList (key) {
      this.viewKey = key
    },
    async getList (context) {
      const { data: res } = await axios.get('/list.json')
      this.list = res
    }
  },
  computed: {
    unDoneLength () {
      const temp = this.list.filter(x => x.done === false)
      return temp.length
    },
    infoList () {
      if (this.viewKey === 'all') {
        return this.list
      }
      if (this.viewKey === 'undone') {
        return this.list.filter(x => x.done === false)
      }
      if (this.viewKey === 'done') {
        return this.list.filter(x => x.done === true)
      }
      return []
    }
  }
}
</script>

<style scoped>
#app {
  padding: 30px;
}

.my_ipt {
  width: 900px;
  margin-right: 30px;
}

.dt_list {
  width: auto;
  height: auto;
  margin-top: 30px;
}

.footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
</style>
