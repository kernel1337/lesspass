<style>
#signInButton {
  border-right: none;
}

#registerButton {
  border-left: none;
}
</style>
<template>
  <form v-on:submit.prevent="signIn">
    <div class="form-group">
      <div class="inner-addon left-addon">
        <i class="fa fa-globe"></i>
        <input
          id="baseURL"
          type="text"
          class="form-control"
          autocapitalize="none"
          v-bind:placeholder="$t('LessPass Database Url')"
          v-model="baseURL"
        />
      </div>
    </div>
    <div class="form-group row">
      <div class="col-12">
        <div class="inner-addon left-addon">
          <i class="fa fa-user"></i>
          <input
            id="email"
            class="form-control"
            name="username"
            type="email"
            autocapitalize="none"
            v-bind:placeholder="$t('Email')"
            required
            v-model="email"
          />
        </div>
      </div>
    </div>
    <div class="form-group mb-2">
      <master-password
        v-model="password"
        v-bind:label="$t('Master Password')"
        v-bind:email="email"
        v-bind:showEncryptButton="true"
        v-bind:EncryptButtonText="$t('Encrypt my master password')"
      ></master-password>
    </div>
    <div class="form-group row no-gutters mb-0">
      <div class="col">
        <button id="signInButton" class="btn btn-primary btn-block">
          {{ $t("Sign In") }}
        </button>
      </div>
      <div class="col">
        <button
          id="registerButton"
          class="btn btn-secondary btn-block"
          type="button"
          v-on:click="register"
        >
          {{ $t("Register") }}
        </button>
      </div>
    </div>
    <div class="form-group mb-0">
      <button
        id="login__forgot-password-btn"
        type="button"
        class="btn btn-link btn-sm p-0"
        v-on:click="$router.push({ name: 'passwordReset' })"
      >
        <small>{{ $t("ForgotPassword", "Forgot your password?") }}</small>
      </button>
    </div>
  </form>
</template>
<script type="text/ecmascript-6">
import User from "../api/user";
import { defaultbaseURL } from "../api/default";
import MasterPassword from "../components/MasterPassword.vue";
import message from "../services/message";

export default {
  data() {
    return {
      email: "",
      password: "",
      baseURL: localStorage.getItem("baseURL") || defaultbaseURL
    };
  },
  components: {
    MasterPassword
  },
  methods: {
    formIsValid() {
      if (!this.email || !this.password || !this.baseURL) {
        message.error(
          this.$t(
            "LoginFormInvalid",
            "LessPass URL, email, and password are mandatory"
          )
        );
        return false;
      }
      return true;
    },
    signIn() {
      if (this.formIsValid()) {
        const baseURL = this.baseURL;
        this.$store.dispatch("setBaseURL", { baseURL });
        User.login({ email: this.email, password: this.password })
          .then(response => {
            this.$store.dispatch("login", response.data);
            this.$store.dispatch("cleanMessage");
            this.$router.push({ name: "home" });
          })
          .catch(err => {
            if (err.response === undefined && baseURL !== defaultbaseURL) {
              message.error(
                this.$t("DBNotRunning", "Your LessPass Database is not running")
              );
            } else if (err.response && err.response.status === 401) {
              message.error(
                this.$t(
                  "LoginIncorrectError",
                  "The email and password you entered did not match our records. Please double-check and try again."
                )
              );
            } else {
              message.displayGenericError();
            }
          });
      }
    },
    register() {
      if (this.formIsValid()) {
        const baseURL = this.baseURL;
        this.$store.dispatch("setBaseURL", { baseURL });
        User.register(
          { email: this.email, password: this.password },
        )
          .then(() => {
            message.success(
              this.$t(
                "WelcomeRegister",
                "Welcome {email}, thank you for signing up.",
                { email: this.email }
              )
            );
            this.signIn();
          })
          .catch(err => {
            if ( err.response === undefined && baseURL !== defaultbaseURL) {
              message.error(this.$t("DBNotRunning", "Your LessPass Database is not running"));
            }else if (
              err.response && err.response.data &&
              typeof err.response.data.email !== "undefined"
            ) {
              if (err.response.data.email[0].indexOf("already exists") !== -1) {
                message.error(
                  this.$t(
                    "EmailAlreadyExist",
                    "This email is already registered. Want to login or recover your password?"
                  )
                );
              }
              if (err.response.data.email[0].indexOf("valid email") !== -1) {
                message.error(
                  this.$t("EmailInvalid", "Please enter a valid email")
                );
              }
            } else if (
              err.response && err.response.data &&
              typeof err.response.data.password !== "undefined"
            ) {
              if (err.response.data.password[0].indexOf("too short") !== -1) {
                message.error(this.$t("PasswordTooShort", "This password is too short. It must contain at least 8 characters."));
              }
              if (err.response.data.password[0].indexOf("too common") !== -1) {
                message.error(this.$t("PasswordTooCommon", "This password is too common."));
              }
            } else {
              message.displayGenericError();
            }
          });
      }
    }
  }
};
</script>
