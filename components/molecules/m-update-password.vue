<template>
  <div class="m-update-password">
    <p class="message">
      {{ $t('If you want to change the password to access your account, enter the following information:') }}<br>
      <template v-if="currentUser">
        {{ $t('Your current email address is') }}
        <span class="message__label">{{ typeof currentUser.email !== 'undefined' ? currentUser.email : '' }}</span>
      </template>
    </p>
    <div class="form">
      <SfInput
        v-model="oldPassword"
        type="password"
        name="currentPassword"
        :label="$t('Current Password')"
        required
        :valid="!$v.oldPassword.$error"
        :error-message="$t('Field is required.')"
        class="form__element"
      />
      <SfInput
        v-model="password"
        type="password"
        name="newPassword"
        :label="$t('New Password')"
        required
        :valid="!$v.password.$error"
        :error-message="$t('Field is required.')"
        class="form__element form__element--half"
      />
      <SfInput
        v-model="rPassword"
        type="password"
        name="repeatPassword"
        :label="$t('Repeat Password')"
        required
        :valid="!$v.rPassword.$error"
        :error-message="
          !$v.rPassword.required
            ? $t('Field is required.')
            : $t('Passwords must be identical.')
        "
        class="form__element form__element--half form__element--half-even"
      />
      <SfButton class="form__button" @click.native="updatePassword">
        {{ $t('Update password') }}
      </SfButton>
    </div>
  </div>
</template>

<script>
import { SfInput, SfButton } from '@storefront-ui/vue';
import { required, sameAs } from 'vuelidate/lib/validators';

export default {
  name: 'MUpdatePassword',
  components: {
    SfInput,
    SfButton
  },
  data () {
    return {
      oldPassword: '',
      password: '',
      rPassword: ''
    }
  },
  computed: {
    currentUser () {
      return this.$store.state.user.current
    }
  },
  methods: {
    updatePassword () {
      this.$v.$touch();
      if (this.$v.$invalid) {
        this.$store.dispatch('notification/spawnNotification', {
          type: 'danger',
          message: this.$t('Please fix the validation errors'),
          action1: { label: this.$t('OK') }
        });
        return;
      }
      this.$bus.$emit('myAccount-before-changePassword', {
        currentPassword: this.oldPassword,
        newPassword: this.password
      })
    }
  },
  validations: {
    oldPassword: {
      required
    },
    password: {
      required
    },
    rPassword: {
      required,
      sameAsPassword: sameAs('password')
    }
  }
};
</script>

<style lang="scss" scoped>
@import "~@storefront-ui/shared/styles/helpers/breakpoints";

.form {
  @include for-desktop {
    display: flex;
    flex-wrap: wrap;
    align-items: center;
  }
  &__element {
    margin-bottom: var(--spacer-2xl);
    @include for-desktop {
      flex: 0 0 100%;
    }
    &--half {
      @include for-desktop {
        flex: 1 1 50%;
      }
      &-even {
        @include for-desktop {
          padding-left: var(--spacer-2xl);
        }
      }
    }
  }
  &__button {
    width: 100%;
    @include for-desktop {
      width: auto;
    }
  }
}
.message {
  font-family: var(--font-family-primary);
  font-weight: var(--font-normal);
  line-height: 1.6;
}
.message {
  margin: 0 0 var(--spacer-2xl) 0;
  font-size: var(--font-base);
  &__label {
    font-weight: 400;
  }
}
</style>
