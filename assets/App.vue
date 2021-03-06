<template>
  <v-app>
    <div id="app">
      <GlobalSnackbar></GlobalSnackbar>
      <AppHeader></AppHeader>
      <v-content v-bind:class="{ isHome: isHomeView }">
        <transition name="component-fade" mode="out-in">
          <router-view/>
        </transition>
        <v-footer height="auto" color="primary">
          <router-link :to="{name: 'About', params: { display: 'about' }}">
            Made with
          </router-link>
          <router-link :to="{name: 'About', params: { display: 'about' }}">
            <img src="/egg.svg" class="egg">
          </router-link>
          <v-spacer></v-spacer>
          <router-link :to="{name: 'About', params: { display: 'licenses' }}">
            &copy; 2018
          </router-link>
          &emsp;|&emsp;
          <router-link :to="{name: 'About', params: { display: 'imprint' }}">
            Imprint
          </router-link>
          &emsp;|&emsp;
          <router-link :to="{name: 'About', params: { display: 'privacy' }}">
            Privacy policy
          </router-link>
        </v-footer>
      </v-content>
    </div>
  </v-app>
</template>

<script>
import Meta from 'mixins/meta'
import GlobalSnackbar from 'components/GlobalSnackbar.vue'
import AppHeader from 'components/AppHeader.vue'

import { api } from 'api'

export default {
  name: 'App',
  mixins: [Meta],
  components: {
    GlobalSnackbar,
    AppHeader
  },
  data () {
    return {
      // eslint-disable-next-line no-undef
      gitHash: __COMMIT_HASH__,
      // eslint-disable-next-line no-undef
      commitUrl: 'https://github.com/sne11ius/egghead.space/commits/' + __COMMIT_HASH__,
      // eslint-disable-next-line no-undef
      gitBranch: __BRANCH_NAME__
    }
  },
  mounted () {
    api.auth.onAuthStateChanged(user => {
      if (user) {
        api.fetchPrivateUserData(user.uid)
          .then(snapshot => {
            if (!snapshot.exists) {
              return api.setPrivateUserData(user)
            }
          })
          .then(() => {
            return api.fetchPublicUserData(user.uid).then(snapshot => {
              if (!snapshot.exists) {
                return api.setPublicUserData(user)
              }
            })
          })
          .then(() => {
            this.$store.dispatch('updateCurrentUser', user.uid)
          })
          .catch(error => {
            console.error('Could not update user info', error)
            // eslint-disable-next-line
            api
              .auth
              .signOut()
              .then(() => {
                this.$store.dispatch('removeCurrentUser')
              })
              .catch(error => {
                // eslint-disable-next-line
                console.error('Could not logout after login error.', error)
              })
          })
      } else {
        this.$store.dispatch('removeCurrentUser')
      }
    })
  },
  computed: {
    isHomeView: function () {
      return this.$route.path === '/'
    }
  }
}
</script>

<style scoped lang="scss">
#wait {
  text-align: center;
  padding-top: 30%;
  h3 {
    margin: 0 auto;
    margin-bottom: 1cm;
    font-size: 1cm;
    font-family: "Roboto";
  }
  img {
    max-height: 150px;
  }
}
#app footer {
  width: 100%;
  color: white;
  padding-left: 8px;
  padding-right: 8px;
  a {
    color: white;
    text-decoration: none;
    &:hover {
      text-decoration: underline;
    }
  }
  .egg {
    height: 20px;
    padding-left: 3px;
    padding-right: 1px;
    position: relative;
    top: 4px;
  }
}
.component-fade-enter-active,
.component-fade-leave-active {
  transition: opacity 0.2s ease;
}
.component-fade-enter,
.component-fade-leave-to {
  opacity: 0;
}
main.v-content {
  padding-top: 90px !important;
  height: 0;
}
main.v-content.isHome {
  padding-top: 0px !important;
}
</style>
