<script setup lang="ts">
  import { useI18n } from "vue-i18n";
  import { useDarkmode } from "@/stores/darkmode";
  import { useUserSession } from "@/stores/userSession";
  import AuthLayout from "@/layouts/AuthLayout.vue";
  import { useNotyf } from "@/composable/useNotyf";

  type StepId = "login" | "forgot-password";

  const notyf = useNotyf();
  const { t } = useI18n();
  const step = ref<StepId>("login");
  const isLoading = ref(false);
  const darkmode = useDarkmode();
  const router = useRouter();
  const route = useRoute();
  const { loginUser } = useUserSession();
  const redirect = route.query.redirect as string;
  const formErrors = reactive({
    username: [],
    password: [],
  });

  const handleLogin = async (event: Event) => {
    try {
      isLoading.value = true;
      Object.assign(formErrors, {
        username: [],
        password: [],
      });

      const values = Object.fromEntries(
        new FormData(event.target as HTMLFormElement)
      );

      await loginUser(values);

      //@ts-ignore
      notyf.dismissAll();
      //@ts-ignore
      notyf.success(`${t("Welcome_back")}, ${values.username?.toUpperCase()}`);

      if (redirect) {
        router.push(redirect);
      } else {
        router.push({
          name: "/app/dashboard",
        });
      }
    } catch (error: any) {
      Object.assign(formErrors, error.response?.data?.errors);
      error.response?.data?.errors.message &&
        //@ts-ignore
        notyf.error(error.response?.data?.errors.message);
    } finally {
      isLoading.value = false;
    }
  };

  function clearError(fieldName: string) {
    formErrors[fieldName as keyof typeof formErrors] = [];
  }
</script>

<template>
  <AuthLayout>
    <div class="modern-login">
      <form
        data-cy="login-form"
        :class="[step !== 'login' && 'is-hidden']"
        class="login-wrapper"
        @submit.prevent="handleLogin"
      >
        <IFeatherUser style="margin-right: 1rem" color="orange" />
        <input
          type="text"
          name="username"
          :placeholder="$t('Username')"
          autocomplete="username"
          @input="clearError('username')"
        />
        <p class="help has-text-danger">
          {{ formErrors.username[0] }}
        </p>
        <IFeatherLock style="margin-right: 1rem" color="orange" />
        <input
          type="password"
          name="password"
          :placeholder="$t('Password')"
          autocomplete="current-password"
          @input="clearError('current-password')"
        />
        <p class="help has-text-danger">
          {{ formErrors.password[0] }}
        </p>

        <div class="button-wrap has-help">
          <button
            id="login-button"
            :loading="isLoading"
            :disabled="isLoading"
            color="primary"
            type="submit"
            size="big"
            rounded
            raised
            bold
          >
            {{ $t("Confirm") }}
          </button>
        </div>
      </form>
    </div>
  </AuthLayout>
</template>

<!-- <style lang="scss" scoped>
  @import "@/scss/pages/auth";
  @import "@/scss/pages/login";
</style> -->
