<template>
  <form
  class="form"
  :action="action"
  @submit.prevent="submit">
    <div class="form__fields">
      <div
          v-for="(field, index) in $v.dataFields.$each.$iter"
          :key=index
          :class="['field', 'form__field', {'field--focused': dataFields[index].value}, {'field--grey': grey}, {'field--error': field.$error}]">
        <label class="field__label"
            :for="id(dataFields[index].name)">
            <span v-if="hasLabel">{{dataFields[index].label}}</span>
          <input
            class="field__input"
            :id="id(dataFields[index].name)"
            :type="dataFields[index].type"
            :name="dataFields[index].name"
            :value="dataFields[index].value"
            :placeholder="dataFields[index].placeholder"
            v-model.trim="field.value.$model"/>
        </label>
        <div class="field__error" v-if="!field.value.required">Field is required</div>
      </div>
    </div>

    <slot name="after-fields"></slot>

    <div v-if="isSmsStage" class="form__2fa">
      <div
          :class="['field', 'form__field', {'field--focused': code}, {'field--grey': grey}, {'field--error': $v.code.$error || showCodeError}]">
        <label class="field__label" :for="id('code')">
          <input
            class="field__input"
            :id="id('code')"
            type="text"
            name="code"
            v-model.trim="$v.code.$model"/>
          <div class="field__placeholder">Enter the code from the SMS we sent you</div>
        </label>
        <div class="field__error" v-if="!$v.code.required">Field is required</div>
        <div class="field__error" v-if="showCodeError" v-html="formErrorText"></div>
      </div>

      <div v-if="!canResend">You may resend new code after <span v-html="countdown"></span> seconds.</div>

      <gozo-button
        v-if="canResend"
        :class="button_classes"
        type="submit"
        @click.native="stage='resend'"
        :disabled="submitStatus == 'PENDING' && !canResend">Resend</gozo-button>

      <div class="">If you didn't receive the code, contact us at <a href="mailto:support@gozo.pro">support@gozo.pro</a>.</div>
    </div>

    <div v-if="isTotpStage"  class="form__2fa">
      <div>Please enter the current code from the Google Authenticator app on your mobile phone</div>

      <div
          :class="['field', 'form__field', {'field--focused': code}, {'field--grey': grey}, {'field--error': $v.code.$error || showCodeError}]">
        <label class="field__label" :for="id('code')">
          <input
            class="field__input"
            :id="id('code')"
            type="text"
            name="code"
            v-model.trim="$v.code.$model"/>
          <div class="field__placeholder"></div>
        </label>
        <div class="field__error" v-if="!$v.code.required">Field is required</div>
        <div class="field__error" v-if="showCodeError" v-html="formErrorText"></div>
      </div>
    </div>

    <slot name="before-submit">
            <div v-if="submitStatus == 'ERROR' && !$v.$invalid && !codeInvalid" class="form__error" v-html="formErrorText"></div>
    </slot>

    <div class="form__submit-wrap">

      <gozo-button v-if="button_text"
        v-html="button_text"
        :class="['form__submit', button_classes,
                  {'loading': (submitStatus == 'PENDING')}]"
        type="submit"
        :disabled="
          button_modal ?
          $v.$invalid :
          submitStatus == 'ERROR'">
      </gozo-button>
      <gozo-button v-else
        :class="['form__submit', button_classes]"
        type="submit"
        :disabled="submitStatus == 'ERROR'"></gozo-button>
    </div>
  </form>
</template>

<script>
import { required, numeric, requiredIf, email } from 'vuelidate/lib/validators'
import axios from 'axios';
import field from './Field.vue';

export default {
  name: 'Form',
  components: {
    field
  },
  props: {
    grey: {
      type: Boolean,
      default: false
    },
    salt: {
      type: String,
      default: ''
    },
    button_text: {
      type: String,
      default: ''
    },
    button_modal: {
      type: Boolean,
      default: false
    },
    button_classes: {
      type: String,
      default: ''
    },
    action: {
      type: String,
      default: '/'
    },
    initial_stage: {
      type: String,
      default: 'create'
    },
    fields: {
      type: Array,
      default: function () {
        return [
          {
            name: 'email',
            placeholder: 'Enter Your Email',
            type: 'email',
            required: 'true',
            value: '',
            label: ''
          },
        ]
      }
    },
    isAuthenticated: {
      type: Boolean,
      default: false
    },
    hasLabel: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      submitStatus: null,
      dataFields: this.fields,
      formErrorTextDefaultText: 'Something went wrong, try again later.',
      formErrorText: 'Something went wrong, try again later.',
      code: '',
      codeInvalid: false,
      stage: this.initial_stage,
      mfa_type: '',
      session_id: '',
      countdownTimer: null,
      countdownDefault: 60,
      countdown: 60,
      canResend: false,
      resendUrl: '/clientsarea/rest/mfa/resend/'
    }
  },
  computed: {
    isSmsStage: function () {
      return (this.stage == '2fa' || this.stage == 'resend') && this.mfa_type == 'sms';
    },
    isTotpStage: function () {
      return this.stage == '2fa' && this.mfa_type == 'totp';
    },
    showCodeError: function () {
      return this.submitStatus == 'ERROR' && this.codeInvalid && this.stage == '2fa'
    }
  },
  methods: {
    getDataByStage () {
      const data = new FormData();

      switch (this.stage) {
        case '2fa':
          data.append('code', this.code);
          data.append('session_id', this.session_id);
          break;
        case 'login':
          data.append('login', this.dataFields[0].value);
          data.append('password', this.dataFields[1].value);
          break;
        case 'resend':
          data.append('session_id', this.session_id);
          break;
        default:
          data.append('email', this.dataFields[0].value);
      }

      return data
    },
    request (url) {
      let _self = this;
      let action = url || _self.action;
      return axios.request({
        method: 'post',
        url: action,
        data: _self.getDataByStage(),
        withCredentials: true,
        config: {
          headers: {
            'Content-Type': 'multipart/form-data'
          }
        }
      })
    },
    actionByStage () {
      let _self = this;
      let req = null;

      switch (_self.stage) {
        case 'create':
          _self.createContact();
          break;
        case '2fa':
          _self.codeInvalid = false
          req = _self.request();
          req.then(response => {
            _self.response = response.data;

            if (_self.response.stage == 'done') {
              _self.stage = 'done';
              window.location.href = _self.response.url;
            } else {
              return _self.submitError();
            }
          })
          .catch(this.requestErrorResult);
          break;
        case 'resend':
          _self.canResend = false;
          _self.countdown = _self.countdownDefault;
          clearTimeout(_self._countdownTimer);

          req = _self.request(_self.resendUrl);
          req.then(response => {
            _self.stage = '2fa';
            _self._countdownTimer = setTimeout(_self.countdownTick, 1000);
            _self.submitStatus = ''

            return false;
          })
          .catch(_self.requestErrorResult);
          break;
        case 'login':
          req = _self.request();
          req.then(response => {
            _self.response = response.data;

            if (_self.response.stage == 'done') {
              _self.stage = 'done';

              window.location.href = _self.response.url;
            } else if(_self.response.stage == '2fa') {
              _self.stage = _self.response.stage;
              _self.mfa_type = _self.response.mfa_type;
              _self.session_id = _self.response.session_id;
              _self.submitStatus = ''
              _self.$v.$reset();

              if (_self.mfa_type == 'sms') {
                clearTimeout(_self._countdownTimer);
                _self._countdownTimer = setTimeout(_self.countdownTick, 1000);
              }

              _self.submitStatus = ''

              return false;
            } else {
              _self.submitStatus = ''
              return _self.submitError();
            }
          })
          .catch(_self.requestErrorResult);
          break;
      }
    },
    countdownTick: function () {
      let _self = this;
      let value = _self.countdown;
      value -= 1;
      if (value <= 0) {
        _self.canResend = true;
        return;
      }
      _self.countdown = value;
      clearTimeout(_self._countdownTimer);
      _self._countdownTimer = setTimeout(_self.countdownTick, 1000);
    },
    requestErrorResult(error) {
      var _self = this;

      if(error.response && error.response.data) {
        _self.response = error.response.data;

        if(_self.response.non_field_errors) {
          return _self.submitError(_self.response.non_field_errors[0]);
        }
      }

      if(error.status == 429){
        return _self.submitError('Too much login attempts! Logging in is locked for few hours.');
      } else {
        return _self.submitError();
      }
    },
    id: function(fieldName) {
      let id = 'field-' + fieldName;
      if(this.salt) id = id + '-' + this.salt;
      return id;
    },
    submitError(formErrorText) {
      if(!this.$v.$invalid) {
        formErrorText ? this.formErrorText = formErrorText : this.formErrorText = this.formErrorTextDefaultText;
        if(this.stage == '2fa') {
          this.codeInvalid = true
        }
        this.submitStatus = 'ERROR';
      }

      if (this.$store.state.logIn) {
        this.$store.commit('toggleLogin');
        this.$store.commit('toggleAutoheight');
      }

      return false;
    },
    createContact() {
      let req = this.request();
      req.then(response => {
        this.response = response
        window.location.href = "/open-account/?email=" + encodeURIComponent(this.dataFields[0].value);
      });
    },
    submit(e) {
      this.$emit('submit', e)
      let _self = this;

      this.$v.$touch();

      if (this.$v.$invalid) {
        return _self.submitError();
      } else {
        // do your submit logic here
        this.submitStatus = 'PENDING';

        this.actionByStage();

        if (!this.$store.state.logIn) {
          _self.$store.commit('toggleLogin');
          _self.$store.commit('toggleAutoheight');
        }
      }
    }
  },
  validations: {
    dataFields: {
      $each: {
        value: {
          required
        }
      }
    },
    code: {
      required: requiredIf(function () {
        return this.stage === '2fa'
      }),
      numeric: true,
    }
  },
  beforeDestroy() {
    clearTimeout(self._countdownTimer);
  }
}
</script>


<style lang="scss">
.form {
  position: relative;

  @include bp(mobile) {
    display: flex;
  }

  .form__fields {
    display: flex;
    flex-direction: column;

    width: 100%;

    align-items: center;

    @include bp(mobile) {
      align-items: flex-start;
      max-width: 318px;
    }

    @include bp(tablet) {
      max-width: 600px;
    }
  }

  .form__field {
    width: 100%;
  }

  .form__submit-wrap {
    text-align: center;

    @include bp(desktop) {
      text-align: right;
    }
  }

  .form__forgot {
    @include bp(tablet) {
      max-width: calc(50% - 15px);
      margin-left: auto;
    }

    width: 100%;
  }

  .form__2fa {
    margin-top: 20px;

    & > * {
      margin-bottom: 20px;
    }
  }

  .form__error {
    margin-top: 10px;
    color: $color__red;
  }

}



@mixin focused () {
  font-size: 14px;
  min-height: calc(100% + 20px);

  &:after {
    visibility: visible;

    width: 100%;
  }
}

.field {
  font-size: $font-size-title;
  position: relative;
  width: 100%;
  max-width: 600px;

  text-align: left;

  .field__label {
    position: relative;
    display: inline-block;
    width: 100%;

    &[for="field-email"] {
      display: flex;
    }
  }

  .field__placeholder {
    position: absolute;
    z-index: 1;
    bottom: 0;

    width: 100%;
    padding: 0 0 16px 0;

    transition: all .3s ease-in-out;

    color: rgba($color__black, .75);

    &:before,
    &:after {
      position: absolute;
      bottom: 0;
      left: 50%;

      height: 2px;

      content: '';
      transition-timing-function: cubic-bezier(.4,0,.2,1);
      transition-duration: .2s;
      transform: translateX(-50%);
    }

    &:before {
      width: 100%;

      background-color: rgba($color__white, .3);
    }

    &:after {
      visibility: hidden;

      width: 0;

      background-color: $color__white;
    }
  }

  .field__input {
    position: relative;
    z-index: 2;

    width: 100%;
    max-width: 100%;
    padding: 13px 30px 12px;

    font-size: 18px;
    line-height: 1.39;
    color: $color__text;
    border: 0;
    background: $color__white;
    border-top-left-radius: $border-radius-base;
    border-bottom-left-radius: $border-radius-base;

    &[value]:not([value='']),
    &:not(:placeholder-shown)
    &:active,
    &:focus {
      outline: none;

      & + .field__placeholder {
        @include focused;
      }
    }
  }

  // FOR THE 1ST RELEASE ONLY

  //

  .field__error {
    font-size: 12px;

    display: none;

    margin: 10px 0;

    color: $color__red;
  }

  &.field--focused {
    .field__placeholder {
      @include focused;

      pointer-events: none;
    }
  }

  &.field--grey {
    .field__placeholder {
      color: rgba($color__darkblue, .3);

      &:before {
        background-color: rgba($color__darkblue, .3);
      }

      &:after {
        background-color: $color__darkblue;
      }
    }

    .field__input {
      color: rgba(0, 0, 0, 0.75);
      border: solid 1px rgba(0, 0, 0, 0.1);
      padding: 12px 30px 11px;
      background: rgba(0, 27, 49, 0.05);
    }
  }

  &.field--error {
    .field__error {
      display: block;

      .contact &,
      .form--intro & {
        top: 100%;
        left: 0;
      }

      .contact & {
        position: absolute;
      }

      .form--intro & {
        position: absolute;
        color: $color__white;
      }
    }

    .field__placeholder {
      color: rgba($color__red, .3);

      .contact & {
        color: rgba($color__white, .3);
      }

      &:before {
        background-color: rgba($color__red, .3);

        .contact & {
          background-color: rgba($color__white, .3);
        }
      }
      &:after {
        background-color: $color__red;

        .contact & {
          background-color: rgba($color__white, .3);
        }
      }
    }
  }
}


.form--intro,
.form--contact {
  .form__field {
    max-width: 100%;
    display: flex;
  }
}

</style>

