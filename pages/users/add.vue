<template>
  <div>
  <a> счет:{{jopa}}</a>
    users
    <button @click="btnclick()">click</button>
  </div>
</template>
<script setup lang="ts">
const jopa = ref<Number>(0) ; 
const btnclick = () => {
console.log("aaaa")
jopa = jopa + 1; 
  
};

</script>
