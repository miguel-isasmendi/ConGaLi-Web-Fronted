<template>
  <div id='app'>
    <div id='topMiniBar'>
      <label for='locale'>Locale</label>
      <select v-model='locale'>
        <option value='en'>English</option>
        <option value='es'>Español</option>
      </select>
    </div>
    <img class='mainLogo' src='./assets/logo.png'>
    <session-manager-modal v-if='showSessionManagerModal || !cgStorage.isAuthenticated()' v-on:cancel='toggleSessionModalActive' v-on:forgotPassword='forgotPassword' v-on:signUp='signUp' v-on:logIn='logIn' />
    <logout-modal v-if='showLogoutModal' v-on:cancel='toggleLogoutModalActive' v-on:confirmed='logOut' />
    <nav class='navbar navbar-default'>
      <div class='container'>
        <ul class='nav navbar-nav'>
          <li><router-link class='actionButton' to='/'>{{ $t('landingPage.title') }}</router-link></li>
          <li><router-link class='actionButton' to='/game'>{{ $t('availableGames.title') }}</router-link></li>
          <li v-if='cgStorage.isAuthenticated()'><a class='actionButton noselect' @click.prevent='toggleLogoutModalActive'>{{ $t('label.logOut') }}</a></li>
          <li v-else><router-link class='actionButton' to='/logIn'>{{ $t('label.logIn') }}</router-link></li>
        </ul>
      </div>
    </nav>
    <router-view>
    </router-view>
  </div>
</template>

<script>

import SessionManagerModal from './components/modal/SessionManagerModal'
import LogoutModal from './components/modal/LogoutModal'

export default {
  name: 'app',
  components: {
    'session-manager-modal': SessionManagerModal,
    'logout-modal': LogoutModal
  },
  props: {
  },
  methods: {
    forgotPassword (userData) {
    },
    signUp (userData) {
      this.$socket.emit('signUp', userData)
      console.log('Data sent: ' + JSON.stringify(userData))
    },
    logIn (userData) {
      this.$socket.emit('logIn', userData)
    },
    logOut () {
      this.$socket.emit('logOut', {
        user: {
          id: this.cgStorage.readLocalUserData().id
        }
      })
    },
    toggleSessionModalActive () {
      this.$data.showSessionManagerModal = !this.$data.showSessionManagerModal
    },
    toggleLogoutModalActive () {
      this.$data.showLogoutModal = !this.$data.showLogoutModal
    }
  },
  socket: {
    events: {
      error (error) {
        console.log('Websocket error!', JSON.stringify(error))
        if (error.type === 'UnauthorizedError' || error.code === 'invalid_token') {
          console.log('User\'s token has expired')
          this.$router.push('/')
        }
      },
      appException (err) {
        console.error('Application error:', JSON.stringify(err))

        err.bodyKey = this.$i18n.t(err.bodyKey)
        err.titleKey = this.$i18n.t(err.titleKey)
        console.error('Translated Application error:', JSON.stringify(err))

        this.$toast(this.ErrorBuilder.buildFor(err), {mode: 'queve', className: 'et-alert'})
      },
      signedUp (data) {
        console.log('SignedUp user data received: ', JSON.stringify(data))

        data.isAuthenticated = true
        this.cgStorage.saveLocalUserData(data)
        this.$data.showSessionManagerModal = false
        this.$router.push('/game')
      },
      loggedIn (data) {
        console.log('LoggedIn user data received: ', JSON.stringify(data))

        data.isAuthenticated = true
        this.cgStorage.saveLocalUserData(data)
        this.$data.showSessionManagerModal = false
        this.$router.push('/game')
      },
      loggedOut (data) {
        console.log('server LoggedOut!')
        this.cgStorage.saveAppData()
        this.$data.showSessionManagerModal = true
        this.$data.showLogoutModal = false
        this.$router.push('/')
        console.log('client LoggedOut!')
      },
      unauthorized (error) {
        if (error.data.type === 'UnauthorizedError' || error.data.code === 'invalid_token') {
          console.log('User\'s token has expired')
          this.$router.push('/')
        }
      }
    }
  },
  data () {
    return {
      showSessionManagerModal: this.showSessionModal,
      showLogoutModal: this.showLogoutModal,
      locale: 'en'
    }
  },
  watch: {
    locale (val) {
      this.$i18n.locale = val
    }
  }
}
</script>

<style>
#app {
  /* font-family: 'Avenir', Helvetica, Arial, sans-serif; */
  font-family: 'Merriweather', serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

h1, h2 {
  font-weight: normal;
}

ol {
  list-style-type: none;
  padding: 0;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
  cursor: pointer;
}

.mainLogo {
  width: 8em;
  height: 8em;
}

.noselect {
  -webkit-touch-callout: none; /* iOS Safari */
    -webkit-user-select: none; /* Safari */
     -khtml-user-select: none; /* Konqueror HTML */
       -moz-user-select: none; /* Firefox */
        -ms-user-select: none; /* Internet Explorer/Edge */
            user-select: none; /* Non-prefixed version, currently
                                  supported by Chrome and Opera */
}

#fake-nav {
  text-align: center;
  margin: 25px 0;
  -webkit-touch-callout: none; /* iOS Safari */
    -webkit-user-select: none; /* Safari */
     -khtml-user-select: none; /* Konqueror HTML */
       -moz-user-select: none; /* Firefox */
        -ms-user-select: none; /* Internet Explorer/Edge */
            user-select: none; /* Non-prefixed version, currently
                                  supported by Chrome and Opera */
}

#fake-nav a, .actionButton {
  color: #f6f6f6;
  border: 0;
  background-color: #3fb67b;
  padding: 10px 25px;
  text-decoration: none;
  cursor: pointer;
  margin-left: 0.5px;
  margin-right: 0.5px;
  -webkit-touch-callout: none; /* iOS Safari */
    -webkit-user-select: none; /* Safari */
     -khtml-user-select: none; /* Konqueror HTML */
       -moz-user-select: none; /* Firefox */
        -ms-user-select: none; /* Internet Explorer/Edge */
            user-select: none; /* Non-prefixed version, currently
                                  supported by Chrome and Opera */
}

#topMiniBar {
  width: 100%;
}
</style>
