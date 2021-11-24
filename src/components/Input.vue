<template>
  <el-form :inline="true" v-model="inputForm" id="inputForm">
    <el-form-item>
      <el-input
        v-model="inputForm.start"
        placeholder="您要从哪出发呐~"
        v-on:keyup.enter="onSubmit()"
      ></el-input>
    </el-form-item>
    <el-form-item>
      <el-button type="primary" @click="onSubmit">查询</el-button>
    </el-form-item>
  </el-form>
</template>

<script>
export default {
  name: "inputForm",
  data() {
    return {
      inputForm: {
        start: "",
        // end: ''
      },
    };
  },
  // watch: {
  // 	inputForm: function() {
  // 		this.$emit('input',this.$data.inputForm)
  // 	}
  // },
  methods: {
    onSubmit() {
      this.$emit("input", this.$data.inputForm);
    },
  },
};
</script>

<style>
#inputForm {
  position: relative;
  left: 6px;
}
</style>
