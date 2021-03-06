<template>
  <section>
    <NavbarLoggedIn v-if="user" :user="user" />
    <NavbarLoggedOut v-else />
    <b-container>
      <div class="text-center py-5">
        <img src="/avatar.png" alt=""/>
        <h1 class="pt-4">{{ $t('index.themagicfrog') }}</h1>
        <h2>{{ $t('index.storymachine') }}</h2>
        <p class="lead pt-5">{{ $t('index.helpremember') }}</p>
      </div>

      <div class="text-center pb-5">
        <h2 class="pt-5 pb-3">{{ $t('index.fullofgold') }}</h2>
        <img src="/pot.png" alt=""/>
        <h5 class="mt-3">{{ $t('index.currentvalue') }}</h5>
        <h1 class="pot-value">{{ $t('index.endollar') }} {{ potValue }} {{ $t('index.frdollar') }}</h1>

        <div class="my-4">
          <LikeButton @voteCasted="updateData" :user="user" :likeLabel="$t('index.generatemore')" :unlikeLabel="$t('index.undogenerate')" :author="latestStoryPost.author" :permlink="latestStoryPost.permlink" v-if="latestStoryPost && user" />
          <b-button variant="primary" class="login-button" v-b-modal.scRedirectModal v-if="!user">
            <svg viewBox="0 0 24 24">
              <path d="M5,9V21H1V9H5M9,21A2,2 0 0,1 7,19V9C7,8.45 7.22,7.95 7.59,7.59L14.17,1L15.23,2.06C15.5,2.33 15.67,2.7 15.67,3.11L15.64,3.43L14.69,8H21C22.11,8 23,8.9 23,10V12C23,12.26 22.95,12.5 22.86,12.73L19.84,19.78C19.54,20.5 18.83,21 18,21H9M9,19H18.03L21,12V10H12.21L13.34,4.68L9,9.03V19Z" />
            </svg>
            {{ $t('index.logintogeneratemore') }}
          </b-button>
        </div>

        <p class="mt-5" v-html="$t('index.itsfree')"></p>
        <p v-html="$t('index.thiswebsite')"></p>
        <p v-html="$t('index.theinfluence')"></p>
      </div>

      <div class="py-5">
        <div class="text-center">
          <h2>{{ $t('index.read') }}</h2>
          <img src="/divider.png" alt="" class="img-fluid"/>
          <div id="currentStory" class="text-center">
            <h1 class="mb-4">{{ latestStoryPostMeta.startPhrase }}</h1>
            <StoryPart v-for="(part, index) in latestStoryPostMeta.commands" :key="index" :part="part" />
            <h3 class="mt-4">{{ $t('index.tobe') }}</h3>
          </div>
          <img src="/divider.png" alt="" class="rotate-180 img-fluid"/>
        </div>
      </div>

      <div class="mx-auto mb-4" style="max-width: 800px;">
        <h2 class="pt-5">{{ $t('index.howwillthestorygoon') }}</h2>
        <p class="text-center mt-4">{{ $t('index.firstread') }}</p>

        <div id="comments">
          <Command v-for="command in currentCommands" :key="command.id" :command="command" :user="user" />
          <p class="text-center" v-if="!currentCommands.length">{{ $t('index.thereareno') }}</p>
        </div>

        <h2 class="pt-5">{{ $t('index.nowitsyourturn') }}</h2>
        <p class="text-center mt-4">{{ $t('index.continuewriting') }}</p>

        <form class="mt-4 p-4 mx-auto" id="command-form" style="max-width: 500px;" v-if="user" @submit.prevent="submitComment">
          <div v-if="!endStory">
            <input class="w-100" id="command" :placeholder="$t('index.form.appendplaceholder')" v-model="commandInput" @keyup="limitCommandCharacters" @keydown="limitCommandCharacters" />
            <sup class="d-block text-center text-muted pt-3"><span id="command-char-count">{{ commandCharactersLeft }}</span> {{ $t('index.form.charactersleft') }}</sup>
            <div v-if="!showImageUpload" class="text-center my-4">
              <p v-html="$t('index.form.youcaneven')"></p>
              <b-button  @click="showImageUpload = true" class="btn btn-outline-success">{{ $t('index.form.yesupload') }}</b-button>
            </div>
            <div v-if="showImageUpload">
              <p class="text-center my-4">
                <b-alert variant="info"
                         dismissible
                         :show="showImageUploadInfo"
                         @dismissed="showImageUploadInfo=false"
                         class="text-left"
                         v-html="$t('index.form.licensenote')">
                </b-alert>
                <input type="file" v-on:change="onImageChange" class="w-100 d-block" ref="image" />
                <img :src="image" v-if="image" alt="uploaded image" class="img-fluid w-100 uploaded-image" />
                <b-button size="sm" class="btn btn-outline-danger mt-3" @click="resetImage">{{ $t('index.form.changedmymind') }}</b-button>
              </p>
              <div class="upload-spinner" v-if="imageIsUploading">
                <div class="dot1"></div>
                <div class="dot2"></div>
              </div>
            </div>
            <div v-if="currentStoryPosts.length > 10">
              <hr>
              <p class="text-center my-4">
                {{ $t('index.form.stopit') }}<br>
                <b-button class="btn btn-outline-danger mt-3 the-end-button" @click="endStory = true">{{ $t('index.form.theend') }}</b-button>
              </p>
            </div>
            <div v-else>
              <p class="text-center my-4">
                <small class="text-muted"><i>{{ $t('index.form.after10days') }}</i></small>
              </p>
            </div>
          </div>
          <div v-if="endStory" class="text-center mb-4">
            <h3><i>{{ $t('index.form.theend') }}</i></h3>
            <div v-if="endCommand">
              <p v-html="$t('index.form.endalreadysuggested', {potValue})"></p>
              <Command :command="endCommand" :user="user" />
            </div>
            <div v-else>
              <p v-html="$t('index.form.ifthecommunity')"></p>
            </div>
            <b-button class="btn btn-outline-success mt-3" @click="endStory = false">{{ $t('index.form.justkidding') }}</b-button>
          </div>
          <div v-if="!(endStory && endCommand)">
            <hr>
            <p class="text-center mt-4 mb-1">{{ $t('index.form.addpersonalnote') }}</p>
            <textarea class="w-100" :placeholder="$t('index.form.commentplaceholder')" v-model="commentInput"></textarea>
            <div v-if="showSuccessMessage" class="text-center alert alert-success">
              {{ $t('index.form.thanksforparticipating') }}
            </div>
            <button class="btn btn-primary d-block w-100 mt-3" v-if="!showSuccessMessage">
              <svg class="spinner" viewBox="0 0 24 24" v-if="submitLoading">
                <path d="M12,4V2A10,10 0 0,0 2,12H4A8,8 0 0,1 12,4Z" />
              </svg>
              {{ $t('index.form.submit') }}
            </button>
          </div>
        </form>
        <div v-if="!user" class="text-center">
          <b-button variant="primary" class="login-button mx-auto" v-b-modal.scRedirectModal>
            <svg viewBox="0 0 24 24">
              <path d="M22,2C22,2 14.36,1.63 8.34,9.88C3.72,16.21 2,22 2,22L3.94,21C5.38,18.5 6.13,17.47 7.54,16C10.07,16.74 12.71,16.65 15,14C13,13.44 11.4,13.57 9.04,13.81C11.69,12 13.5,11.6 16,12L17,10C15.2,9.66 14,9.63 12.22,10.04C14.19,8.65 15.56,7.87 18,8L19.21,6.07C17.65,5.96 16.71,6.13 14.92,6.57C16.53,5.11 18,4.45 20.14,4.32C20.14,4.32 21.19,2.43 22,2Z" />
            </svg>
            {{ $t('index.form.logintowrite') }}
          </b-button>
        </div>
      </div>
    </b-container>

    <Footer />

    <Modals :user="user" :loginUrl="loginUrl" />

    <notifications group="errors" classes="vue-notification error" position="top center" :duration="8000" />
  </section>
</template>

<script>
import axios from 'axios'
import steem from 'steem'
import sc2 from 'sc2-sdk'
import marked from 'marked'
import Cookies from 'js-cookie'

import NavbarLoggedIn from '~/components/NavbarLoggedIn'
import NavbarLoggedOut from '~/components/NavbarLoggedOut'
import LikeButton from '~/components/LikeButton'
import Command from '~/components/Command'
import StoryPart from '~/components/StoryPart'
import Footer from '~/components/Footer'
import Modals from '~/components/Modals'

// TODO: wallet integration
// TODO: edit comments/submissions

export default {
  components: {
    NavbarLoggedIn,
    NavbarLoggedOut,
    LikeButton,
    Command,
    StoryPart,
    Footer,
    Modals
  },
  data() {
    return {
      user: null,
      endStory: false,
      commandInput: '',
      commentInput: '',
      submitLoading: false,
      showSuccessMessage: false,
      image: null,
      imageIsUploading: false,
      showImageUpload: false,
      showImageUploadInfo: true
    }
  },
  async asyncData(context) {
    const getPosts = (accountName, limit = 100) => {
      return new Promise((resolve, reject) => {
        steem.api.getDiscussionsByBlog({tag: accountName, limit: limit}, (err, posts) => {
          if (err) {
            reject(err);
          } else {
            resolve(posts);
          }
        });
      });
    };

    const getComments = (accountName, permlink) => {
      return new Promise((resolve, reject) => {
        steem.api.getContentReplies(accountName, permlink, function(err, comments) {
          if (err) {
            reject(err);
          } else {
            resolve(comments);
          }
        });
      });
    };

    let posts = await getPosts(context.app.account);
    let comments = [];
    for (let i = 0; i < posts.length; i++) {
      let post = posts[i];
      let meta = JSON.parse(post.json_metadata);
      if (post.author === context.app.account && meta.hasOwnProperty('day') && meta.hasOwnProperty('storyNumber')) {
        comments = await getComments(context.app.account, posts[i].permlink);
        break;
      }
    }

    return { posts, comments }
  },
  computed: {
    sc2() {
      const api = sc2.Initialize({
        app: 'themagicfrog.app',
        callbackURL: this.redirectUrl,
        scope: ['vote', 'comment']
      });

      const accessToken = Cookies.get('frog_token');
      if (accessToken) {
        api.setAccessToken(accessToken);
        api.me((err, user) => {
          if (err) {
            console.log(err);
          } else {
            this.user = user;
          }
        });
      }
      return api;
    },
    redirectUrl() {
      if (process.env.NODE_ENV === 'development') {
        return process.env.scheme + '://' + process.env.host + (process.env.port ? ':' + process.env.port : '') + '/auth';
      } else if (this.$i18n.fallbackLocale === this.$i18n.locale) {
        return 'https://the-magic-frog.com/auth'
      }

      return 'https://' + this.$i18n.locale + '.the-magic-frog.com/auth'
    },
    loginUrl() {
      return this.sc2.getLoginURL();
    },
    potValue() {
      let pot = 0;
      for (let i = 0; i < this.currentStoryPosts.length; i++) {
        pot += parseFloat(this.getPostPot(this.currentStoryPosts[i]));
      }
      pot *= 0.95; // 5 % goes to beneficiaries
      return pot.toFixed(2);
    },
    allStoryPosts() {
      return this.posts.filter(post => {
        let meta = JSON.parse(post.json_metadata);
        return post.author === this.$account && meta.hasOwnProperty('day') && meta.hasOwnProperty('storyNumber');
      });
    },
    currentStoryPosts() {
      return this.allStoryPosts.filter(post => {
        let meta = JSON.parse(post.json_metadata);
        return parseInt(meta.storyNumber) === this.currentStoryNumber
      });
    },
    currentStoryNumber() {
      return this.latestStoryPostMeta.storyNumber
    },
    latestStoryPost() {
      return this.allStoryPosts[0];
    },
    latestStoryPostMeta() {
      return JSON.parse(this.latestStoryPost.json_metadata);
    },
    currentCommands() {
      let canEnd = this.latestStoryPostMeta.day > 10;

      return this.comments.filter(comment => {
        if (comment.json_metadata) {
          let meta = JSON.parse(comment.json_metadata);
          return meta.hasOwnProperty('type') && ((meta.type === 'end' && canEnd) || meta.type === 'append');
        }
        return false;
      });
    },
    endCommand() {
      let endCommand = null;
      this.currentCommands.forEach(comment => {
        let meta = JSON.parse(comment.json_metadata);
        if (meta.type === 'end') {
          endCommand = comment;
        }
      });
      return endCommand;
    },
    commandCharactersLeft() {
      return Math.max(250 - this.commandInput.length, 0);
    }
  },
  methods: {
    async updateData() {
      console.log('updated data');
      const getPosts = (accountName, limit = 100) => {
        return new Promise((resolve, reject) => {
          steem.api.getDiscussionsByBlog({tag: accountName, limit: limit}, (err, posts) => {
            if (err) {
              reject(err);
            } else {
              resolve(posts);
            }
          });
        });
      };

      const getComments = (accountName, permlink) => {
        return new Promise((resolve, reject) => {
          steem.api.getContentReplies(accountName, permlink, function(err, comments) {
            if (err) {
              reject(err);
            } else {
              resolve(comments);
            }
          });
        });
      };

      let posts = await getPosts(this.$account);
      let comments = [];
      for (let i = 0; i < posts.length; i++) {
        let meta = JSON.parse(posts[i].json_metadata);
        if (meta.hasOwnProperty('day') && meta.hasOwnProperty('storyNumber')) {
          comments = await getComments(this.$account, posts[i].permlink);
          break;
        }
      }

      this.posts = posts;
      this.comments = comments;
    },
    limitCommandCharacters() {
      this.commandInput = this.commandInput.substr(0, 250);
    },
    getPostPot(post) {
      if (post.last_payout === '1970-01-01T00:00:00') {
        return parseFloat(post.pending_payout_value.replace(' SBD', '')) * 0.75 / 2;
      }

      return (parseFloat(post.total_payout_value.replace(' SBD', '')) / 2).toFixed(2);
    },
    logout() {
      this.user = null;
      Cookies.remove('frog_token');
      return null;
    },
    submitComment() {
      let meta = {
        type: 'append',
        appendText: this.commandInput.trim(),
        comment: this.commentInput.trim(),
        image: this.image || '', // don't set to null, would be removed if edited via steemit.com
        author: this.user.name
      };

      if (this.endStory) {
        meta.type = 'end';
        meta.appendText = '# The End!';
      }

      if (meta.appendText || meta.image) {
        let body = '';
        if (meta.appendText) {
          body += '> ' + meta.appendText + '\n\n';
        }

        if (meta.image) {
          body += '> ![image-' + (new Date()).getTime() + '](' + meta.image + ')\n\n';
        }

        if (meta.comment) {
          body += meta.comment;
        }

        let permlink = 're-' + this.latestStoryPost.permlink + '-command-' + (new Date()).getTime();

        this.submitLoading = true;
        this.sc2.comment(
          this.$account,
          this.latestStoryPost.permlink,
          this.user.name,
          permlink,
          '',
          body,
          meta,
          (err) => {
            if (err) {
              console.log(err);
            } else {
              this.commandInput = '';
              this.commentInput = '';
              this.submitLoading = false;
              this.showSuccessMessage = true;
              this.image = null;
              this.$refs.image.value = null;

              steem.api.getContentReplies(this.$account, this.latestStoryPost.permlink, (err, comments) => {
                if (err) {
                  console.log(err);
                } else {
                  this.comments = comments;
                }
              });
            }
          }
        );
      }
    },
    onImageChange() {
      // TODO: add validation of filesize and type
      if (!window || !window.File || !window.FileReader || !window.FileList || !window.Blob) {
        alert('The File APIs are not fully supported in this browser.');
      } else if (!this.$refs.image.files) {
        alert('This browser doesn\'t seem to support the `files` property of file inputs.');
      } else if (!this.$refs.image.files[0]) {
        alert("No file selected.");
      } else {
        let file = this.$refs.image.files[0];

        let fr = new FileReader();
        fr.onload = () => {
          this.imageIsUploading = true;

          let data = fr.result;
          let base64image = data.replace('data:image/png;base64,', '')
            .replace('data:image/jpg;base64,', '')
            .replace('data:image/jpeg;base64,', '')
            .replace('data:image/gif;base64,', '');

          axios({
            method: 'post',
            url: 'https://api.imgur.com/3/image',
            data: {
              image: base64image,
              type: 'base64'
            },
            headers: {
              'Authorization': 'Client-ID a57bbb06e896db0',
              'content-type': 'application/json'
            },
          }).then(result => {
            this.imageIsUploading = false;
            this.image = result.data.data.link;
          });
        };
        fr.readAsDataURL(file);
      }
    },
    resetImage() {
      this.image = null;
      this.showImageUpload = false;
      this.$refs.image.value = null;
    }
  },
  events: {
    onVoteCasted: function () {
      this.updateData();
    },
  }
}
</script>
