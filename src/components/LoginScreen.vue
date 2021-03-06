<template>
  <f7-login-screen id="login-screen">
    <f7-view>
      <f7-page login-screen>
        <f7-login-screen-title>Вход</f7-login-screen-title>
        <f7-list form>
          <f7-list-input
            type="text"
            name="username"
            placeholder="E-mail"
            :value="email.value"
            :error-message="email.error"
            :error-message-force="!!email.error"
            @input="email.value = $event.target.value"
          ></f7-list-input>
          <f7-list-input
            type="password"
            name="password"
            placeholder="Пароль"
            :value="password.value"
            :error-message="password.error"
            :error-message-force="!!password.error"
            @input="password.value = $event.target.value"
          ></f7-list-input>

          <f7-block>
            <f7-button fill large type="submit" @click.prevent="login">
              Войти
            </f7-button>
          </f7-block>
        </f7-list>
      </f7-page>
    </f7-view>
  </f7-login-screen>
</template>

<script>
import { merge, isEmpty } from 'lodash';
import { auth } from '@/js/firebaseConfig.js';
import notify from '@/js/helpers/notify.js';
import validateEmail from '@/js/helpers/validateEmail.js';
import firebaseErrorToHumanError from '@/js/helpers/firebaseErrorToHumanError.js';

export default {
  name: 'LoginScreen',

  data() {
    return {
      email: { value: '', error: '' },
      password: { value: '', error: '' }
    };
  },

  methods: {
    validate() {
      const errors = {};
      const { email, password } = this;

      email.error = '';
      password.error = '';

      if (!validateEmail(email.value)) {
        errors.email = 'Неправильный формат эл. почты';
      }

      if (!email.value) {
        errors.email = 'Обязательное поле';
      }

      if (!password.value) {
        errors.password = 'Обязательное поле';
      }

      if (password.value.length < 6) {
        errors.password = 'Минимум 6 символов';
      }

      if (isEmpty(errors)) {
        return true;
      }

      Object.keys(errors).forEach(error => {
        errors[error] = {
          error: errors[error]
        };
      });

      merge(this.email, errors.email);
      merge(this.password, errors.password);
    },

    login() {
      if (!this.validate()) {
        return;
      }

      this.$f7.preloader.show();
      const { email, password } = this;

      auth
        .signInWithEmailAndPassword(email.value, password.value)
        .then(user => {
          this.$f7.preloader.hide();
          this.$store.commit('user/SET_USER_ID', user.uid);
          this.$f7.loginScreen.close('#login-screen');
          this.$f7.views.main.router.navigate('/', {
            clearPreviousHistory: true
          });
        })
        .catch(error => {
          this.$f7.preloader.hide();
          firebaseErrorToHumanError(error.code, message => {
            notify(message);
          });
        });
    }
  }
};
</script>
