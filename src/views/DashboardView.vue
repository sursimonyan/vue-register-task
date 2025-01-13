<script setup>
import { ref, reactive } from 'vue'
import { useVuelidate } from '@vuelidate/core'
import { required, maxLength, minLength } from '@vuelidate/validators'
import { useRouter } from 'vue-router'

const router = useRouter();

const userData = JSON.parse(localStorage.getItem('loginData'))
const userPassword = userData.password;

const isOldPasswordNotCorrect = ref(false);
const isOldPasswordCantUse = ref(false);
const passwordChangedSuccess = ref(false);

const userInfo = reactive({
  email: userData.email,
  firstName: userData.firstName,
  lastName: userData.lastName,
  username: userData.username,
  date: userData.date,
});

const formData = reactive({
  oldPassword: '',
  password: '',
  confirmPassword: '',
});

const rules = {
  oldPassword: { required, minLength: minLength(8), maxLength: maxLength(20) },
  password: { required, minLength: minLength(8), maxLength: maxLength(20) },
  confirmPassword: {
    minLength: minLength(8), maxLength: maxLength(20), required: function () {
      return formData.confirmPassword === formData.password;
    }
  },
};

const v$ = useVuelidate(rules, formData);

const resetPassword = async () => {
  const result = await v$.value.$validate();

  if (result) {
    if (formData.oldPassword === userPassword && formData.password !== userPassword) {
      localStorage.setItem('loginData', JSON.stringify({
        firstName: userInfo.firstName,
        lastName: userInfo.lastName,
        date: userInfo.date,
        email: userInfo.email,
        password: formData.password,
        username: `${userInfo.firstName} ${userInfo.lastName}`,
      }));

      formData.oldPassword = '';
      formData.password = '';
      formData.confirmPassword = '';

      passwordChangedSuccess.value = true;
    }

    if (formData.oldPassword !== userPassword) {
      isOldPasswordNotCorrect.value = true
    }

    if (formData.password === userPassword) {
      isOldPasswordCantUse.value = true;
    }
  }
}

const logOut = () => {
  localStorage.removeItem('token');
  router.push({ name: 'login' });
}

</script>

<template>
  <div class="max-w-[30rem] w-full mx-auto">
    <h1 class="mb-8 text-center text-5xl">Dashboard Page</h1>
    <div class="text-2xl text-center">
      <h3 class="mb-2">Welcome: {{ userData.username }}</h3>
      <div v-if="passwordChangedSuccess">
        <p class="mb-5 text-lg text-center font-bold">Your password is reset</p>
        <button @click="logOut" class="w-full h-[2.5rem] bg-green-500 text-white hover:bg-green-600">Log out</button>
      </div>
      <form v-else @submit.prevent="resetPassword"
        class="flex flex-col mb-4 p-3 text-base border border-green-500 rounded-xl">
        <label class="mb-4">
          <span class="mb-2 block text-xl font-bold">Old Password</span>
          <input v-model="formData.oldPassword"
            class="w-full p-2 border border-green-400 rounded-md outline-0 input_focus" type="password"
            :class="{ 'border-red-600': v$.oldPassword.$error && v$.oldPassword.$touch }" @blur="v$.oldPassword.$touch">
          <div v-if="v$.oldPassword.$error" class="text-red-600 font-bold">Old Password is required and min 8 and max 20
            characters
          </div>
        </label>
        <label class="mb-4">
          <span class="mb-2 block text-xl font-bold">New Password</span>
          <input v-model="formData.password" class="w-full p-2 border border-green-400 rounded-md outline-0 input_focus"
            type="password" :class="{ 'border-red-600': v$.password.$error && v$.password.$touch }"
            @blur="v$.password.$touch">
          <div v-if="v$.password.$error" class="text-red-600 font-bold">Password is required and min 8 and max 20
            characters</div>
        </label>
        <label class="mb-4">
          <span class="mb-2 block text-xl font-bold">New Password Confirm</span>
          <input v-model="formData.confirmPassword"
            class="w-full p-2 border border-green-400 rounded-md outline-0 input_focus" type="password"
            :class="{ 'border-red-600': v$.confirmPassword.$error && v$.confirmPassword.$touch }"
            @blur="v$.confirmPassword.$touch">
          <div v-if="formData.password !== formData.confirmPassword" class="text-red-600 font-bold">Confirm password
            must equal to password</div>
          <div v-if="v$.confirmPassword.$error" class="text-red-600 font-bold">Confirm password is required and min 8
            and max 20 characters</div>
        </label>
        <div v-if="isOldPasswordNotCorrect" class="text-red-600 font-bold">Old Password is not correct</div>
        <div v-if="isOldPasswordCantUse" class="text-red-600 font-bold">You can't use old password</div>
        <button type="submit" class="h-10 text-xl bg-green-500 text-white rounded-md hover:bg-green-600">Reset
          password</button>
      </form>
    </div>
  </div>
</template>
