<template>
    <div class="wrapper">
      <div class="main-container">
        <form>
            <div class="email">
                <label for="password">{{ $t("email") }}</label>
            <input id="email" type="email" v-model="email" required autocomplete="username" :placeholder="$t('yourEmail')">
            </div>
          <br>
          <div class="password">
            <label for="password">{{ $t("password") }}</label>
            <input id="password" type="password" v-model="password" autocomplete="current-password" required :placeholder="$t('yourPassword')">
          </div>
          <p>{{ error }}</p>
          <br>
          <div class="submit">
            <button type="button" @click="login()">{{ $t("logIn") }}</button>
          </div>
          <br>
          <div class="register">
            <p>{{ $t("doYouWantToRegisterNewUser") }}? </p>
            <button type="button" @click="changeRoute('Register')">{{ $t("registerNewUser") }}</button>
          </div>
          <br>
        </form>
      </div>
    </div>
  </template>
  
  <script>
  import axios from 'axios';
  import { useTokenStore } from "../../stores/userToken";
  
  export default {
    data() {
      return {
        email: '',
        password: '',
        error: '',
      }
    },
    setup() {
      const tokenStore = useTokenStore();
      return { tokenStore };
    },
    methods: {
      changeRoute(string) {
        this.$router.push({ name: string })
      },
      async login() {
        if (!this.email || !this.password) {
          this.error = `${this.$t("requiredFields")}`;
          return;
        }
        await this.tokenStore.getTokenAndSaveInStore(this.email, this.password);
        if (this.tokenStore.jwtToken || this.tokenStore.jwtToken) {
          this.changeRoute('Home')
        } else {
          this.error = `${this.$t("loginFailed")}`;
        }
      }
    }
  }
  </script>




<style scoped>

.wrapper{
    display: flex;
    flex-direction: column;
    place-items: center;
    justify-content: center;
    align-items: center;
    width: 100vw; 
    height: 100vh;
}

.main-container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  background-color: var(--color-blue-lighter);
  width: 65%;
  height: 75%;
  border-radius: 15px;
  padding: 1rem;
}

label {
  display: block;
  text-align: center;
  font-size: 1.2rem;
  font-weight: bold;
  margin-bottom: 0.5rem;
}

input {
  width: 100%;
  padding: 0.5rem;
  margin-bottom: 1rem;
  border: none;
  border-radius: 5px;
  box-shadow: 0px 0px 5px 1px rgba(0, 0, 0, 0.1);
  font-size: 1.2rem;
}

.submit {
  display: flex;
  justify-content: center;
  margin-top: 1rem;
}

.error {
  color: red;
  margin-top: 0.5rem;
  text-align: center;
}

.register{
    display: flex;
    flex-direction: column;
    align-items: center;
    margin-top: 1rem;
}


</style>
