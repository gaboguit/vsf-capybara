<template>
  <div class="m-update-personal-data">
    <p class="message">
      {{ $t('Feel free to edit any of your details below so your account is always up to date') }}
    </p>
    <div class="form">
      <SfInput
        v-model="firstName"
        name="firstName"
        :label="$t('First Name')"
        required
        :valid="!$v.firstName.$error"
        :error-message="!$v.firstName.required ? $t('Field is required.') : $t('Name must have at least 2 letters.')"
        class="form__element form__element--half"
      />
      <SfInput
        v-model="lastName"
        name="lastName"
        :label="$t('Last Name')"
        required
        :valid="!$v.lastName.$error"
        :error-message="$t('Field is required.')"
        class="form__element form__element--half form__element--half-even"
      />
      <SfInput
        v-model="email"
        type="email"
        name="email"
        :label="$t('Your e-mail')"
        required
        :valid="!$v.email.$error"
        :error-message="
          !$v.email.required
            ? $t('Field is required.')
            : $t('Please provide valid e-mail address.')
        "
        class="form__element"
      />
      <SfButton class="form__button" @click.native="updatePersonalData">
        {{ $t('Update personal data') }}
      </SfButton>
    </div>
    <p class="notice">
      {{ $t('At Brand name, we attach great importance to privacy issues and are committed to protecting the personal data of our users. Learn more about how we care and use your personal data in the') }}
      <a href="">{{ $t('Privacy Policy') }}</a>.
    </p>
  </div>
</template>

<script>
import { SfInput, SfButton } from '@storefront-ui/vue';
import { required, minLength, email } from 'vuelidate/lib/validators';
import { unicodeAlpha } from '@vue-storefront/core/helpers/validators';

export default {
  name: 'MUpdatePersonalData',
  components: {
    SfInput,
    SfButton
  },
  data () {
    return {
      firstName: '',
      lastName: '',
      email: ''
    }
  },
  methods: {
    updatePersonalData () {
      this.$v.$touch();
      if (this.$v.$invalid) {
        this.$store.dispatch('notification/spawnNotification', {
          type: 'danger',
          message: this.$t('Please fix the validation errors'),
          action1: { label: this.$t('OK') }
        });
        return;
      }
      let updatedProfile = JSON.parse(JSON.stringify(this.$store.state.user.current))
      updatedProfile.firstname = this.firstName
      updatedProfile.lastname = this.lastName
      updatedProfile.email = this.email
      this.$bus.$emit('myAccount-before-updateUser', updatedProfile)
    }
  },
  beforeMount () {
    // current user may not be available yet in beforeMount hook so vuex watcher is needed
    const unsubscribeFromStoreWatch = this.$store.watch(
      state => state.user.current,
      currentUser => {
        if (currentUser) {
          this.firstName = currentUser.firstname;
          this.lastName = currentUser.lastname;
          this.email = currentUser.email;
        }
      },
      { immediate: true });

    this.$once('hook:beforeDestroy', unsubscribeFromStoreWatch)
  },
  validations: {
    firstName: {
      required,
      minLength: minLength(2),
      unicodeAlpha
    },
    lastName: {
      required,
      unicodeAlpha
    },
    email: {
      required,
      email
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
.message,
.notice {
  font-family: var(--font-family-primary);
  font-weight: var(--font-normal);
  line-height: 1.6;
}
.message {
  margin: 0 0 var(--spacer-2xl) 0;
  font-size: var(--font-base);
}
.notice {
  margin: var(--spacer-xl) 0 0 0;
  font-size: var(--font-xs);
  @include for-desktop {
    max-width: 70%;
    margin: var(--spacer) 0 0 0;
  }
}
</style>
