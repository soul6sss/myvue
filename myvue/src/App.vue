<template>
  <div class="app">
    姓名：<input v-model="username"> {{ username }}<br>
    薪水：<input type="number" v-model.number="salary"> {{ salary }}<br>
    <button @click="addSalary">提交</button>
    <hr>
    
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue'

export default defineComponent({
  data() {
    return {
      username: 'roy',
      salary: 15000,
      userInfo:{
        age:0,
        sex:1,
        department:'',
        skill:["java","python","vue"]
      }
    }
  },
  methods: {
    addSalary() {
      this.salary += 1000
      console.log('薪水更新：', this.salary) // 验证是否执行
    }
  }
})
</script>

<style scoped>
 .userInfo span{
    background-color: yellow;
    border: 1px;
    break-after: always;
  }
.app {
  background-color: rgb(229, 229, 241);
 
}
</style>