<template>
  <div class="createTemplate">
    <p class="title">{{ formData.id ? "上传" : "录入" }}检测报告</p>
    <a-divider />

    <a-form
      ref="formRef"
      :model="formData"
      :rules="formRules"
      @finish="OnSubmit"
    >
      <a-row :gutter="24">
        <a-col :span="12">
          <a-form-item label="检测报告" name="files">
            <a-upload
              v-model:file-list="formData.filesUpFiles"
              action="/api/file/upload"
              :max-count="1"
            >
              <a-button>
                <upload-outlined></upload-outlined>
                上传文件
              </a-button>
            </a-upload>
          </a-form-item>
        </a-col>

        <a-col :span="24">
          <a-form-item :wrapper-col="{ span: 10, offset: 10 }">
            <a-button type="primary" html-type="submit">提交</a-button>
            <a-button style="margin-left: 10px" @click="OnReset">重置</a-button>
          </a-form-item>
        </a-col>
      </a-row>
    </a-form>
  </div>
</template>
<script setup>
import { useStore } from "@/store";
import { POST } from "@/utils/axios";
import { message } from "ant-design-vue";
import { Editor, Toolbar } from "@wangeditor/editor-for-vue";
import "@wangeditor/editor/dist/css/style.css";
import dayjs from "dayjs";
const useState = useStore();
const router = useRouter();
const route = useRoute();

// 基础form相关
let formRef = ref(null);
const formData = reactive({});
const formRules = reactive({});
let photoImageUrl = ref();

const PhotoUploadChange = (info) => {
  if (info.file.status === "uploading") {
    return;
  }
  if (info.file.status === "done") {
    // Get this url from response in real world.
    GetBase64(info.file.originFileObj, (base64Url) => {
      photoImageUrl.value = base64Url;
    });
  }
  if (info.file.status === "error") {
    message.error("上传失败！");
  }
};

const GetBase64 = (img, callback) => {
  const reader = new FileReader();
  reader.addEventListener("load", () => callback(reader.result));
  reader.readAsDataURL(img);
};

// 编辑器
const editorRef = shallowRef(); // 编辑器实例，必须用 shallowRef
const mode = "default";
const toolbarConfig = {};
const editorConfig = {
  placeholder: "请输入内容...",
  MENU_CONF: {
    uploadImage: {
      server: "/api/file/upload",
      fieldName: "file",
      methods: "post",
      // 单个文件上传成功之后
      onSuccess(file, res) {},
      // 自定义插入图片
      customInsert(res, insertFn) {
        if (res.code == 0) {
          let { url, alt, href } = res.data;
          insertFn(url, alt, href);
        }
      },
    },
  },
};
const HandleCreated = (editor) => {
  editorRef.value = editor; // 记录 editor 实例，重要！
};

// 提交
const OnSubmit = async (values) => {
  let { id } = formData;
  let data = Object.assign(values, {
    id,

    photo: formData.photo
      ? formData.photo[0]?.url == undefined
        ? formData.photo[0]?.response?.data?.url
        : formData.photo[0]?.url
      : undefined,

    files: formData.filesUpFiles
      ? JSON.stringify(formData.filesUpFiles[0]?.response?.data)
      : formData.files,
  });
  const { code } = await POST(
    `/projectappointment/${id != undefined ? "update" : "save"}`,
    data
  );
  if (code != 0) return message.error("创建失败");
  router.push("/ProjectappointmentList");
};
// 重置表单
const OnReset = () => {
  formRef.value.resetFields();
};

onMounted(() => {
  const { type, editVal } = history.state;
  if (type) {
    nextTick(() => {
      let editObj = JSON.parse(editVal);
      let {} = editObj;

      Object.assign(formData, editObj, {
        photo: [Object.assign({}, { url: editObj.photo })],

        filesUpFiles: editObj.files ? [JSON.parse(editObj.files)] : [],
      });
    });
  }
});

// 组件销毁时，也及时销毁编辑器
onBeforeUnmount(() => {
  const editor = editorRef.value;
  if (editor == null) return;
  editor.destroy();
});
</script>
<style lang="scss" scoped>
.createTemplate {
  .title {
    font-size: 20px;
    font-weight: 800;
  }
}
</style>
