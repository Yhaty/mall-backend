<template>
  <el-dialog
    :title="type == 'add' ? '添加轮播图' : '修改轮播图'"
    v-model="state.visible"
    width="400px"
  >
    <el-form
      :model="state.ruleForm"
      :rules="state.rules"
      ref="formRef"
      label-width="100px"
      class="good-form"
    >
      <el-form-item label="图片" prop="url">
        <el-upload
          class="avatar-uploader"
          :action="state.uploadImgServer"
          accept="jpg,jpeg,png"
          :headers="{
            token: state.token,
          }"
          :show-file-list="false"
          :before-upload="handleBeforeUpload"
          :on-success="handleUrlSuccess"
        >
          <img
            style="width: 200px; height: 100px; border: 1px solid #e9e9e9"
            v-if="state.ruleForm.url"
            :src="state.ruleForm.url"
            class="avatar"
          />
          <i v-else class="avatar-uploader-icon"><el-icon><Plus /></el-icon></i>
        </el-upload>
      </el-form-item>
      <el-form-item label="跳转链接" prop="link">
        <el-input type="text" v-model="state.ruleForm.link"></el-input>
      </el-form-item>
      <el-form-item label="排序值" prop="sort">
        <el-input type="number" v-model="state.ruleForm.sort"></el-input>
      </el-form-item>
    </el-form>
    <template #footer>
      <span class="dialog-footer">
        <el-button @click="state.visible = false">取 消</el-button>
        <el-button type="primary" @click="submitForm">确 定</el-button>
      </span>
    </template>
  </el-dialog>
</template>

<script setup>
import { reactive, ref } from "vue";
import axios from "@/utils/axios";
// uploadImgServer 公用图片上传接口，我将其统一封装在 /utils/index.js 文件中
import { localGet, uploadImgServer } from "@/utils";
import { ElMessage } from "element-plus";
import { Plus } from "@element-plus/icons-vue";

const props = defineProps({
  type: String, // add 为新增；edit 为编辑
  reload: Function, // table 刷新的方法
});
// formRef 用于表单验证控制
const formRef = ref(null);
const state = reactive({
  uploadImgServer,
  token: localGet("token") || "", // 用于调用上传图片接口是，放在请求头上的 token
  visible: false, // 控制弹窗的显示隐藏
  ruleForm: {
    url: "",
    link: "",
    sort: "",
  },
  rules: {
    url: [{ required: true, message: "图片不能为空", trigger: ["change"] }],
    sort: [{ required: true, message: "排序不能为空", trigger: ["change"] }],
  },
  id: "",
});
// 获取详情
const getDetail = (id) => {
  axios.get(`/carousels/${id}`).then((res) => {
    state.ruleForm = {
      url: res.carouselUrl,
      link: res.redirectUrl,
      sort: res.carouselRank,
    };
  });
};
// 上传之前，控制上传的文件。
const handleBeforeUpload = (file) => {
  const sufix = file.name.split(".")[1] || "";
  if (!["JPG","jpg", "jpeg", "png"].includes(sufix)) {
    ElMessage.error("请上传 jpg、jpeg、png 格式的图片");
    return false;
  }
};
// 上传图片
const handleUrlSuccess = (val) => {
  state.ruleForm.url = val.data || "";
};
// 开启弹窗，此方法将在父组件，通过 ref 直接调用。
const open = (id) => {
  state.visible = true;
  // 如果带着 id，则是编辑，否则为新增
  if (id) {
    state.id = id;
    getDetail(id);
  } else {
    state.ruleForm = {
      url: "",
      link: "",
      sort: "",
    };
  }
};
// 关闭弹窗
const close = () => {
  state.visible = false;
};
// 提交表单方法
const submitForm = () => {
  console.log(formRef.value.validate);
  formRef.value.validate((valid) => {
    // valid 为是否通过表单验证的变量
    if (valid) {
      if (props.type == "add") {
        // 增加用 axios.post
        axios
          .post("/carousels", {
            carouselUrl: state.ruleForm.url,
            redirectUrl: state.ruleForm.link,
            carouselRank: state.ruleForm.sort,
          })
          .then(() => {
            ElMessage.success("添加成功");
            state.visible = false;
            if (props.reload) props.reload();
          });
      } else {
        // 编辑用 axios.put
        axios
          .put("/carousels", {
            carouselId: state.id,
            carouselUrl: state.ruleForm.url,
            redirectUrl: state.ruleForm.link,
            carouselRank: state.ruleForm.sort,
          })
          .then(() => {
            ElMessage.success("修改成功");
            state.visible = false;
            if (props.reload) props.reload();
          });
      }
    }
  });
};
// 后续我们会在外面使用该组件内部的方法属性，通过 <script setup> 形式编写的组件，需通过 defineExpose 方法，将属性暴露出去。
defineExpose({ open, close });
</script>

<style scoped>
.avatar-uploader {
  width: 100px;
  height: 100px;
  color: #d3d3d3;
  font-size: 30px;
}
.avatar-uploader :deep(.el-upload){
  width: 100%;
  text-align: center;
}
.avatar-uploader-icon {
  display: block;
  width: 100%;
  height: 100%;
  border: 1px solid #e9e9e9;
  padding: 32px 17px;
}
</style>
