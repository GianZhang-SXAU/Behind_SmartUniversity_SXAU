<template>
  <div class="Login">

    <div class="container">

      <div class="form">
        <h2>登录</h2>
        <a-form
          :model="userInfo.formModel"
          name="basic"
          autocomplete=" off"
          @finish="OnSubmit"
        >
          <a-form-item
            name="username"
            :rules="[{ required: true, message: '请输入你的账号!' }]"
            class="input"
          >
            <i>
              <UserOutlined />
            </i>
            <a-input
              v-model:value="userInfo.formModel.username"
              placeholder="请输入你的账号"
              type="text"
            />
          </a-form-item>
          <a-form-item
            name="password"
            :rules="[{ required: true, message: '请输入你的密码!' }]"
            class="input"
          >
            <i>
              <LockOutlined />
            </i>
            <a-input
              v-model:value="userInfo.formModel.password"
              placeholder="请输入你的密码"
              type="password"
            />
          </a-form-item>
          <button type="primary" html-type="submit" class="login-button">
            登录
          </button>
        </a-form>
      </div>
    </div>
  </div>
</template>
<script setup>
import { POST } from "@/utils/axios";
import { useRouter } from "vue-router";
import { useStore } from "@/store";
import { message } from "ant-design-vue";
import { register } from "swiper/element";
let useState = useStore();
const isLogin = ref(true);
const router = useRouter();
let title = import.meta.env.VITE_WEB_TITLE;
const userInfo = reactive({
  formModel: {
    username: "",
    password: "",
    captcha: "",
    role: "0",
    remember: true,
  },
});
const options = [
  { label: "管理员", value: "0" },
  { label: "Pear", value: "Pear" },
  { label: "Orange", value: "Orange" },
];
const toggleFlag = ref(false);
// 重置按钮
const ResetBtn = (flag = undefined) => {
  toggleFlag.value = flag ? false : true;
};
// 发送邮箱
const SendEmail = async () => {
  if (userInfo.formModel.username == "")
    return message.error({ content: `账号不能为空!` });
  const resultRes = await POST(`/admin/sendEmail`, {
    username: userInfo.formModel.username,
  });
  if (resultRes.code == 0) {
    message.success({ content: `发送成功，请到邮箱查看!` });
  } else {
    message.error({ content: `${resultRes.msg}!` });
  }
};
// 重置密码
const ResetSubmit = async () => {
  let data = {
    username: userInfo.formModel.username,
    captcha: userInfo.formModel.captcha,
  };
  let resultRes = await POST(`/admin/resetPass`, data);
  if (resultRes.code == 500) return message.error(resultRes.msg);
  message.success(resultRes.data, 10);
};
const OnSubmit = async () => {
  let resultFlag = await useState.Login(userInfo);
  if (resultFlag === true) {
    router.push("/");
    message.success({ content: "登陆成功!" });
  } else {
    message.error({ content: `${resultFlag}!` });
  }
};
</script>

<style scoped lang="scss">
:deep(.ant-form-item) {
  margin: 0;
}

:deep(.ant-input:focus) {
  box-shadow: none;
}

.Login {
  width: 100%;
  height: 100%;
  background: url("./img/img.png");
  overflow: hidden;
  background-size: cover;
  display: flex;
  justify-content: center;
  align-items: center;



  .container {
    position: relative;
    padding: 50px;
    width: 360px;
    min-height: 400px;
    display: flex;
    justify-content: center;
    align-items: center;
    background: rgba(0, 0, 0, 0.04);
    backdrop-filter: blur(5px);
    border-radius: 10px;
    box-shadow: 0 25px 45px rgba(0, 0, 0, 0.2);

    .form {
      position: relative;
      width: 100%;
      height: 100%;

      p {
        color: #fff;
        font-size: 15px;
        margin-top: 20px;

        a {
          color: #fff;
          cursor: pointer;
        }

        a:hover {
          color: #434343;
        }
      }

      .login-button {
        width: 100%;
        background: #fff;
        color: #111;
        max-width: 100px;
        padding: 8px 10px;
        box-shadow: none;
        letter-spacing: 1px;
        cursor: pointer;
        transition: 1.5s;
        margin-left: 80px;
      }

      .login-button:hover {
        background: linear-gradient(
          115deg,
          rgba(0, 0, 0, 0.1),
          rgba(255, 255, 255, 0.25)
        );
        color: #fff;
        transition: 0.5s;
        border: 1px solid #fff3;
      }

      h2 {
        color: #010810;
        letter-spacing: 2px;
        margin-bottom: 30px;
        margin-top: 0;
        text-align: center;
      }

      .input {
        position: relative;
        width: 100%;
        margin-bottom: 20px;

        input {
          outline: none;
          border: none;
          border: 1px solid rgba(255, 255, 255, 0.2);
          background: rgba(255, 255, 255, 0.2);
          padding: 8px 10px;
          padding-left: 40px;
          border-radius: 15px;
          color: #090000;
          font-size: 16px;
          box-shadow: 0 5px 15px rgba(0, 0, 0, 0.05);

          &::placeholder {
            color: #010810;
          }
        }

        i {
          color: rgba(0, 0, 0, 0.87);
          width: 32px;
          position: absolute;
          top: 12px;
          left: 12px;
        }
      }
    }
  }

  .container::after {
    content: "";
    position: absolute;
    top: 5px;
    right: 5px;
    bottom: 5px;
    left: 5px;
    border-radius: 5px;
    pointer-events: none;
    background: linear-gradient(
      to bottom,
      rgba(255, 255, 255, 0.1) 0%,
      rgba(255, 255, 255, 0.1) 2%
    );
  }
}
</style>
