<template>
  <card type="user">
    <p class="card-text">
    </p>
    <div class="author">
      <div class="block block-one"></div>
      <div class="block block-two"></div>
      <div class="block block-three"></div>
      <div class="block block-four"></div>

      <img class="avatar" src="avatar-placeholder.png" alt="...">
      <h5 class="title">
        {{ user.person ? user.person.name() ? user.person.name() : user.username : "Nameless Person" }}'s {{ $t('general.profile') }}
      </h5>
      <p class="description">
        {{userEmail}}
      </p>
      <p class="description">
        {{userLocation}}
      </p>
    </div>
    <p></p>
    <div class="text-center">
      <p class="card-description">
        {{userNodeInfo}}
      </p>
      <p class="card-description">
        {{userDescription}}
      </p>
    </div>
    <div slot="footer" class="button-container">
      <base-button v-if="signedIn" round @click="addFollow">
        {{ $t('general.follow') }}
      </base-button>
      <base-button round @click="emailUser">
        {{ $t('general.email') }}
      </base-button>
    </div>
  </card>
</template>
<script>
  const logger = require('heroku-logger')
  var PRIVATE_FOLLOW_LIST = 'private/following.json'
  export default {
    props: ['user'],
    data: () => ({
      blockstack: window.blockstack
    }),
    computed: {
      userEmail () {
        if (this.user && this.user.publicInformation && this.user.publicInformation.email) {
          return this.user.publicInformation.email
        }
        return ''
      },
      userLocation () {
        if (this.user && this.user.publicInformation && this.user.publicInformation.location) {
          return this.user.publicInformation.location
        }
        return ''
      },
      userDescription () {
        if (this.user && this.user.publicInformation && this.user.publicInformation.description) {
          return this.user.publicInformation.description
        }
        return ''
      },
      userNodeInfo () {
        if (this.user && this.user.publicInformation && this.user.publicInformation.nodeInformation) {
          return this.user.publicInformation.nodeInformation
        }
        return ''
      },
      signedIn () {
        return this.blockstack.isUserSignedIn()
      }
    },
    methods: {
      addFollow () {
        // first grab follow list
        const blockstack = this.blockstack
        logger.info('Grabbing followingList list')
        blockstack.getFile(PRIVATE_FOLLOW_LIST, {decrypt: true})
          .then((followListJson) => {
            var updateFollowList = false
            var followList = []
            var storeToFollow = {username: this.user.username, name: this.user.person.name(), avatar: this.user.person.avatarUrl()}
            if (followListJson) {
              logger.info('followingList list: ' + followListJson)
              followList = JSON.parse(followListJson || '[]')
              // check if user is already a friend
              var alreadyFollowing = followList.find(f => f.username === this.user.username)
              if (!alreadyFollowing) {
                followList.push(storeToFollow)
                updateFollowList = true
              } else {
                logger.info('user already following')
              }
            } else {
              logger.info('followingList list empty')
              followList = [storeToFollow]
              updateFollowList = true
            }
            if (updateFollowList) {
              logger.info('Saving private file', {file: PRIVATE_FOLLOW_LIST})
              blockstack.putFile(PRIVATE_FOLLOW_LIST, JSON.stringify(followList), {encrypt: true})
                .then(() => {
                  this.notifySuccess(this.$t('profile.alerts.followingUser'), null)
                })
                .catch((error) => {
                  this.notifyFailure(this.$t('profile.alerts.failureFollowingUser'), error)
                })
            } else {
              this.notifyWarning(this.$t('profile.alerts.alreadyFollowingUser'))
            }
          })
          .catch((error) => {
            logger.error('could not get/save followingList list: ' + error)
            this.notifyFailure(this.$t('profile.alerts.failureFollowingUser'), error)
          })
      },
      emailUser () {
        window.location.href = 'mailto:' + this.user.publicInformation.email
      },
      notifySuccess (title, text) {
        this.$vs.notify({
          color: 'success',
          title: title,
          text: text
        })
      },
      notifyWarning (title, text) {
        this.$vs.notify({
          color: 'warning',
          title: title,
          text: text
        })
      },
      notifyFailure (title, text) {
        this.$vs.notify({
          color: 'danger',
          title: title,
          text: text
        })
      }
    }
  }
</script>
<style>
</style>
