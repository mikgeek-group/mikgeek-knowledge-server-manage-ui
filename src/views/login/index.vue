<template>
  <div class="dowebok dwo">
    <div
      class="container absolute left-1/2 top-1/2 transform -translate-x-1/2 -translate-y-1/2 pb-5 overflow-hidden"
    >
      <div class="relative max-w-md mx-auto">
        <div class="form relative z-20 p-8 bg-white shadow-lg rounded-1xl">
          <div class="flex flex-auto">
            <img :src="websiteConfig.loginImage" alt="" />
          </div>
          <h2 class="mb-6 text-2xl text-center">用户名密码登录</h2>

          <n-form
            ref="formRef"
            label-placement="left"
            size="large"
            :model="formInline"
            :rules="rules"
          >
            <n-form-item path="username">
              <n-input v-model:value="formInline.username" placeholder="请输入用户名">
                <template #prefix>
                  <n-icon size="18" color="#808695">
                    <PersonOutline />
                  </n-icon>
                </template>
              </n-input>
            </n-form-item>
            <n-form-item path="password">
              <n-input
                v-model:value="formInline.password"
                type="password"
                showPasswordOn="click"
                placeholder="请输入密码"
              >
                <template #prefix>
                  <n-icon size="18" color="#808695">
                    <LockClosedOutline />
                  </n-icon>
                </template>
              </n-input>
            </n-form-item>
            <n-form-item class="default-color">
              <div class="flex justify-between">
                <div class="flex-initial">
                  <n-checkbox v-model:checked="autoLogin">自动登录</n-checkbox>
                </div>
              </div>
            </n-form-item>
            <n-form-item>
              <n-button type="primary" @click="handleSubmit" size="large" :loading="loading" block>
                登录
              </n-button>
            </n-form-item>
          </n-form>

          <div class="flex justify-center items-center text-center">
            <span class="h-px flex-1 bg-gray-200"></span>
            <span class="mx-4">或者使用第三方登录</span>
            <span class="h-px flex-1 bg-gray-200"></span>
          </div>
          <div class="grid grid-cols-2 gap-4 mt-4">
            <a
              class="col-span-2 flex justify-center items-center py-2 border border-solid border-cyan-750 rounded shadow hover:bg-gray-100"
              href="javascript:"
            >
              短信验证码登录
            </a>
            <a
              class="flex justify-center items-center py-2 border border-solid border-cyan-750 rounded shadow hover:bg-gray-100"
              href="javascript:"
            >
              微信登录
            </a>
            <a
              class="flex justify-center items-center py-2 border border-solid border-cyan-750 rounded shadow hover:bg-gray-100"
              href="javascript:"
            >
              APP扫码登录
            </a>
          </div>
        </div>
        <img class="hidden md:block lg-bg-1 absolute z-10" src="@/assets/images/svg/1.svg" alt="" />
        <img class="hidden md:block lg-bg-2 absolute z-10" src="@/assets/images/svg/2.svg" alt="" />
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
  import { reactive, ref } from 'vue';
  import { useRoute, useRouter } from 'vue-router';
  import { useUserStore } from '@/store/modules/user';
  import { useMessage } from 'naive-ui';
  import { ResultEnum } from '@/enums/httpEnum';
  import { PersonOutline, LockClosedOutline, LogoGithub, LogoFacebook } from '@vicons/ionicons5';
  import { PageEnum } from '@/enums/pageEnum';
  import { websiteConfig } from '@/config/website.config';
  interface FormState {
    username: string;
    password: string;
  }

  const formRef = ref();
  const message = useMessage();
  const loading = ref(false);
  const autoLogin = ref(true);
  const LOGIN_NAME = PageEnum.BASE_LOGIN_NAME;

  const formInline = reactive({
    username: 'admin',
    password: '123456',
    isCaptcha: true,
  });

  const rules = {
    username: { required: true, message: '请输入用户名', trigger: 'blur' },
    password: { required: true, message: '请输入密码', trigger: 'blur' },
  };

  const userStore = useUserStore();

  const router = useRouter();
  const route = useRoute();

  const handleSubmit = (e) => {
    e.preventDefault();
    formRef.value.validate(async (errors) => {
      if (!errors) {
        const { username, password } = formInline;
        message.loading('登录中...');
        loading.value = true;

        const params: FormState = {
          username,
          password,
        };

        try {
          const { code, message: msg } = await userStore.login(params);
          message.destroyAll();
          if (code == ResultEnum.SUCCESS) {
            const toPath = decodeURIComponent((route.query?.redirect || '/') as string);
            message.success('登录成功，即将进入系统');
            if (route.name === LOGIN_NAME) {
              router.replace('/');
            } else router.replace(toPath);
          } else {
            message.info(msg || '登录失败');
          }
        } finally {
          loading.value = false;
        }
      } else {
        message.error('请填写完整信息，并且进行验证码校验');
      }
    });
  };
</script>

<style lang="less" scoped>
  .text-cyan-750 {
    color: #0a8080;
  }

  .hover_text-cyan-850:hover {
    color: #005961;
  }

  .border-cyan-750 {
    border-color: #0a8080;
  }

  input {
    outline: none;
  }

  .dwo {
    font-size: 0.875rem;
  }

  body {
    background-color: #fbfafa;
  }

  .dowebok .lg-bg-1 {
    left: -400px;
    top: 70px;
  }

  .dowebok .lg-bg-2 {
    right: -375px;
    top: 144px;
  }

  .dwo .ipt {
    background-color: #f4f4f3;
  }

  .dwo .ipt:focus {
    background-color: #f7fcfc;
  }

  .dwo .ipt-line {
    border-bottom: 1px solid #6c6c72;
  }

  .dwo .ipt:focus ~ .ipt-line {
    border-bottom-width: 3px;
    border-color: #0a8080;
    animation: dwo 0.2s ease-in-out;
  }

  .ipt-wrap {
    border-radius: 4px;
    overflow: hidden;
  }

  .dwo .btn-login {
    background-color: #0a8080;
  }

  .dwo .btn-login:hover {
    background-color: #005961;
  }

  @keyframes dwo {
    from {
      transform: scaleX(0.7);
    }
    to {
      transform: scaleX(1);
    }
  }
</style>
