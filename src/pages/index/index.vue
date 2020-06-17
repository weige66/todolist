<template>
  <!-- 顶部输入框 -->
  <view class="container">
    <view class="header">
      <text class="plus">添加:</text>
      <input
        class="new-todo"
        :value="input"
        placeholder="这里输入内容..."
        auto-focus
        @input="inputChangeHandle"
        @confirm="addTodoHandle"
      />
    </view>
    <block v-if="todos.length">
      <view class="todos">
        <!--当列表项被标记为完成时，应该得到“完成”类 -->
        <view
          class="item"
          :class="item.completed ? ' completed' : '' "
          v-for="(item, index) in todos"
          :key="item.index"
          @tap="toggleTodoHandle"
          :data-index="index"
        >
          <!-- 完成:成功，待做:循环 -->
          <icon class="checkbox" :type="item.completed ? 'success' : 'circle'" />
          <text class="name" v-if="item.completed ? 'success' : 'circle'">{{ item.name }}</text>
          <icon class="remove" type="clear" size="16" @tap="removeTodoHandle" :data-index="index" />
        </view>
      </view>
      <!-- 切换按钮 -->
      <view class="footer">
        <text class="btn" @tap="toggleAllHandle">全部切换</text>
        <text v-if="leftCount" :class="leftCount === 1 ? 'item' : 'items'">代办：{{ leftCount }}项</text>
        <text class="btn" v-if="todos.length > leftCount" @tap="clearCompletedHandle">删除已完成</text>
      </view>
    </block>
    <block v-else>
      <view class="empty">
        <text class="title">恭喜!</text>
        <text class="content">没有更多的工作了.</text>
      </view>
    </block>
  </view>
</template>

<script>
export default {
  data() {
    return {
      input: '',
      todos: [],
      leftCount: 0,
      allCompleted: false
    }
  },
  onLoad() {
    this.load()
  },
  methods: {
    inputChangeHandle(e) {
      this.input = e.detail.value
    },
    save() {
      uni.setStorageSync('todo_list', this.todos)
    },
    load() {
      var todos = uni.getStorageSync('todo_list')
      if (todos) {
        var leftCount = todos.filter(function(item) {
          return !item.completed
        }).length
        this.todos = todos
        this.leftCount = leftCount
      }
    },
    addTodoHandle(e) {
      if (!this.input || !this.input.trim()) return
      var todos = this.todos
      todos.push({ name: this.input, completed: false })
      this.input = ''
      this.todos = todos
      this.leftCount = this.leftCount + 1
      this.save()
    },
    toggleTodoHandle(e) {
      var index = e.currentTarget.dataset.index
      var todos = this.todos
      todos[index].completed = !todos[index].completed
      this.todos = todos
      this.leftCount = this.leftCount + (todos[index].completed ? -1 : 1)
      this.save()
    },
    removeTodoHandle(e) {
      var index = e.currentTarget.dataset.index
      var todos = this.todos
      var remove = todos.splice(index, 1)[0]
      this.todos = todos
      this.leftCount = this.leftCount - (remove.completed ? 0 : 1)
      this.save()
    },
    toggleAllHandle(e) {
      this.allCompleted = !this.allCompleted
      var todos = this.todos
      for (var i = todos.length - 1; i >= 0; i--) {
        todos[i].completed = this.allCompleted
      }
      this.todos = todos
      this.leftCount = this.allCompleted ? 0 : todos.length
      this.save()
    },
    clearCompletedHandle(e) {
      var todos = this.todos
      var remains = []
      for (var i = 0; i < todos.length; i++) {
        todos[i].completed || remains.push(todos[i])
      }
      this.todos = remains
      this.save()
    }
  }
}
</script>

<style>
page {
  background-color: #d2e5d1;
}
.container {
  padding: 30rpx;
  border-top: 1rpx solid #e5e5e5;
}
.header {
  display: flex;
  align-items: center;
  border: 1rpx solid #e0e0e0;
  border-radius: 10rpx;
  box-shadow: 0 0 5rpx #e0e0e0;
  margin-bottom: 30rpx;
  padding: 20rpx;
}

.header > .plus {
  width: 70rpx;
  height: 42rpx;
  margin-right: 20rpx;
}

.header .new-todo {
  flex: 1;
  font-size: 28rpx;
}

.todos {
  border: 1rpx solid #e0e0e0;
  border-radius: 10rpx;
  box-shadow: 0 0 5rpx #e0e0e0;
}

.todos .item {
  display: flex;
  align-items: center;
  padding: 25rpx;
  border-bottom: 1rpx solid #e0e0e0;
}

.todos .item:last-child {
  border-bottom: 0;
}

.todos .item .checkbox {
  margin-right: 20rpx;
}

.todos .item .name {
  flex: 1;
  font-size: 30rpx;
  color: #444;
}

.todos .completed .name {
  text-decoration: line-through;
  color: #888;
}

.todos .item .remove {
  cursor: pointer;
}

.footer {
  display: flex;
  justify-content: space-between;
  margin: 30rpx 0;
  padding: 0 10rpx;
  font-size: 26rpx;
  color: #777;
}

.footer .btn {
  cursor: pointer;
  padding: 10rpx;
  border-radius: 10rpx;
  background-color: #89caed;
  border: 1rpx solid #444;
}

.empty {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.empty .title {
  font-size: 60rpx;
  margin: 200rpx 50rpx 50rpx;
  color: #444;
}

.empty .content {
  color: #666;
  text-align: center;
}
</style>
