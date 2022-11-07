<template>
  <div>{{name}}</div>
  <div>{{count}}</div>
  <button @click="sum">count++</button>
</template>

<script lang="ts">
import { defineComponent, ref, toRefs,onMounted, reactive} from 'vue';
import CryptoJS from "crypto-js";
export default defineComponent({
  name: 'HelloWorld',
  props: {
    msg: String,
  },
  setup(){
    let name: string = '李哈哈';
    let count = ref(0);
    function sum(){
      count.value++
     
    };
    

    let person = reactive({
      name:'li',
      age:20
    })
    

    return {
      sum,
      name,
      count
    }
  }
});
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">

</style>
