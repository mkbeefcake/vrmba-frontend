<template>
  <div class="container" style="padding-top:120px">

  <div class="row">
    <div class="col-lg-8 m-auto">
      <card :title="$t('login')">
        <form @submit.prevent="login" @keydown="form.onKeydown($event)">
          <!-- Email -->
          <div class="form-group row">
            <label class="col-md-3 col-form-label text-md-right">{{ $t('email') }}</label>
            <div class="col-md-7">
              <input v-model="form.email" :class="{ 'is-invalid': form.errors.has('email') }" class="form-control" type="email" name="email">
              <has-error :form="form" field="email" />
            </div>
          </div>

          <!-- Password -->
          <div class="form-group row">
            <label class="col-md-3 col-form-label text-md-right">{{ $t('password') }}</label>
            <div class="col-md-7">
              <input v-model="form.password" :class="{ 'is-invalid': form.errors.has('password') }" class="form-control" type="password" name="password">
              <has-error :form="form" field="password" />
            </div>
          </div>

          <!-- Remember Me -->
          <div class="form-group row">
            <div class="col-md-3" />
            <div class="col-md-7 d-flex">
              <checkbox v-model="remember" name="remember">
                {{ $t('remember_me') }}
              </checkbox>

              <router-link :to="{ name: 'password.request' }" class="small ml-auto my-auto">
                {{ $t('forgot_password') }}
              </router-link>
            </div>
          </div>

          <div class="form-group row">
            <div class="col-md-7 offset-md-3 d-flex">
              <!-- Submit Button -->
              <v-button :loading="form.busy">
                {{ $t('login') }}
              </v-button>
              <a class="btn btn-md btn-google btn-block text-uppercase btn-outline" 
                 href="#" v-on:click="onGoogleSignup($event)" style="padding-left:10px; width:200px;margin-left:10px">
                <img src="https://img.icons8.com/color/16/000000/google-logo.png"> Sign in with Google
              </a>

            </div>
          </div>
        </form>
      </card>
    </div>
  </div>
  </div>
</template>

<script>
import Form from 'vform'
import Cookies from 'js-cookie'
import firebase from 'firebase'

export default {
  components: {
  },

  middleware: 'guest',

  metaInfo () {
    return { title: this.$t('login') }
  },

  data: () => ({
    form: new Form({
      email: '',
      password: ''
    }),
    remember: false
  }),

  methods: {
    onGoogleSignup: function(event) {
      var provider = new firebase.auth.GoogleAuthProvider();
      firebase.auth()
        .signInWithPopup(provider)
        .then((result) => {
          /** @type {firebase.auth.OAuthCredential} */
          var credential = result.credential;
          var token = credential.accessToken;
          var user = result.user;

          debugger;
         
          user.getIdToken()
            .then(idToken => {

              console.log(idToken);

              // Save the token.
              this.$store.dispatch('auth/saveToken', {
                token: idToken,
                user: user
              })

              // Redirect home.
              // window.location.assign('/home')
              this.redirect()

            })
            .catch(err => {
              alert(err.message);
            });

        }).catch((error) => {
          // Handle Errors here.
          // var errorCode = error.code;
          var errorMessage = error.message;
          // var email = error.email;
          // var credential = error.credential;
          alert(err.message);
        });
    },
    login () {
      firebase.auth().signInWithEmailAndPassword(this.form.email, this.form.password)
        .then(data => {

          const user = data.user;
          user.getIdToken()
            .then(idToken => {

              console.log(idToken);
              
              // Save the token.
              this.$store.dispatch('auth/saveToken', {
                token: idToken,
                user: user
              })

              // Redirect home.
              // window.location.assign('/home')
              this.redirect()

            })
            .catch(err => {
              alert(err.message);
            });
        })
        .catch(err => {
          alert(err.message);
        });

    },

    redirect () {
      const intendedUrl = Cookies.get('intended_url')

      if (intendedUrl) {
        Cookies.remove('intended_url')
        this.$router.push({ path: intendedUrl })
      } else {
        this.$router.push({ name: 'home' })
      }
    }
  }
}
</script>
