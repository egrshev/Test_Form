<template>
  <h2 class="user-header-preview">Перрсональные данные</h2>
  <p class="user-preview">{{userName}}, {{userAge}} {{userPrefix}}</p>
  <h2 class="childs-header-preview">Дети</h2>
  <div v-for="(child, index) in children" :key="child">
    <span class="childs-items-preview">{{children[index].name}}, {{children[index].age}} {{children[index].prefix}}</span><br><br>
  </div>
</template>
<script>
export default {
  props: ['userName', 'userAge', 'children', 'userPrefix'],
  methods: {

  }
}
</script>