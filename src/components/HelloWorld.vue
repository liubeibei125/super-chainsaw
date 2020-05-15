<template>
  <div>
    <el-form ref="infoForm" label-width="130px" :rules="rules">
      <el-form-item label="出生时间:" prop="">
        <kalendarnl :dateArr="dateArr" @changeDay="changeDay" :longInput="longInput"></kalendarnl>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
import kalendarnl from '../assembly/kalendarnl.vue'
export default {
  name: 'HelloWorld',
  data () {
    return {
      longInput: 0,
      dateArr: []
    }
  },
  components: {
    kalendarnl
  },
  methods: {
    changeDay (getBirthdayInfo) {
      console.log(getBirthdayInfo)
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
