<template>
  <el-upload
    id="Upload"
    action="/"
    drag
    ref="upload"
    accept=".txt"
    :limit="1"
    :auto-upload="false"
    :file-list="fileList"
    :disabled="this.fileList.length !== 0"
    :on-change="handlePreview"
  >
    <i class="el-icon-upload"></i>
    <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
  </el-upload>
</template>

<script>
export default {
  name: "Upload",
  data() {
    return {
      fileList: [],
    };
  },
  props: ["msg"],
  methods: {
    // 点击预览时 将txt值传回主app
    handlePreview(f) {
      const rawTxt = f.raw;
      let reader = new FileReader();
      if (typeof FileReader === "undefined") {
        this.$message({
          type: "info",
          message: "您的浏览器不支持FileReader接口",
        });
        return;
      }
      reader.readAsText(rawTxt);
      let that = this;
      reader.onload = function(e) {
        let res = e.target.result;
        that.$data.fileList = res;
        that.$emit("txt", res);
      };
    },
  },
};
</script>

<style>
#Upload {
  position: relative;
  margin: 0 auto;
}

.el-upload-dragger {
  /* background-color: #FFFFFF !important; */
  background-color: rgba(255, 255, 255, 0.6) !important;
  border: transparent 1px solid !important;
  border-radius: 30px !important;
  align-items: center;
  justify-items: center;
  box-shadow: 20px 20px 45px #f0f0f0, -20px -20px 45px #ffffff;
}
</style>
