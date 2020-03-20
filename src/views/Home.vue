<template>
  <div class="home">
    <div class="form-wrapper">
      <form class="form">
        <div class="form__fields">
          <div class="field field--text">
            <label for="username">Twitter user</label>
            <input type="text" name="username" id="username" required placeholder="@username" v-model="username">
            <svg class="twitter-logo" xmlns="http://www.w3.org/2000/svg" width="30" height="24.333"><path d="M9.5 24.333C20.833 24.333 27 15 27 6.833V6a13.548 13.548 0 003-3.167 13.836 13.836 0 01-3.5 1A6.484 6.484 0 0029.167.5 15.287 15.287 0 0125.333 2a5.952 5.952 0 00-4.5-2 6.262 6.262 0 00-6.167 6.167 3.249 3.249 0 00.167 1.333A17.231 17.231 0 012.167 1a6.383 6.383 0 00-.833 3.167A6.622 6.622 0 004 9.333 5.619 5.619 0 011.167 8.5a6.091 6.091 0 005 6 5.138 5.138 0 01-1.667.167 2.836 2.836 0 01-1.167-.167 6.314 6.314 0 005.833 4.333A12.584 12.584 0 011.5 21.5a4.614 4.614 0 01-1.5-.167 15.731 15.731 0 009.5 3" fill-rule="evenodd"/></svg>
          </div>
          <div class="field field--checkbox">
            <input type="checkbox" id="rhymes" name="rhymes" v-model="rhymes">
            <label for="rhymes"><strong>Try</strong> to make it rhyme</label>
          </div>
          <div class="field field--checkbox">
            <input type="checkbox" id="alexandrine" name="alexandrine" v-model="alexandrine">
            <label for="alexandrine"><strong>Try</strong> to make alexandrines <button class="info-button" @click.prevent="displayInfo = !displayInfo"><span class="screen-reader-text">Help</span>?</button></label>
            <p class="infobox" v-show="displayInfo">“The French alexandrine is a syllabic poetic meter of 12 syllables with a medial caesura dividing the line into two hemistichs (half-lines) of six syllables each.“ says Wikipedia.</p>
          </div>
        </div>
        <div class="form__submit">
          <button type="submit" :class="{'loading': generating}" @click.prevent='testGeneratePoem'>Generate Poem</button>
        </div>
      </form>
    </div>
    <div class="poem" v-if="ready">
      <h2 class="poem__title">{{ title }}</h2>
      <div class="poem__author">A poem by {{ author }}</div>
    </div>
    <div class="debug">
      <div>{{ atUsername }}</div>
      <div>{{ screenname }}</div>
      <div>Rhymes: {{ rhymes }}</div>
      <div>Alexandrine: {{ alexandrine }}</div>
      <div>{{ tweets }}</div>
      <div>{{ sentences }}</div>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator';
// @ is an alias to /src
import Header from '@/components/Header.vue';
import axios from 'axios';
import syllable from 'syllable';

// import { fetchHomeTimeline } from 'twitter-api-ts';
// import * as option from 'fp-ts/lib/Option';

@Component({
  components: {
    Header,
  },
})

export default class Home extends Vue {
  private generating: boolean = false;
  private ready: boolean = false;
  private params: URLSearchParams | null = null;
  private nameParam: string = '';
  private username: string = '@realDonaldTrump';
  private rhymes: boolean = true;
  private alexandrine: boolean = true;
  private displayInfo: boolean = false;
  private tweets: any | null = null;
  private error: any | null = null;
  private sentences: any[] = [];
  private title: string = '';
  private author: string = '';

  private twitterConsumerKey: any = process.env.VUE_APP_TWITTERCONSUMERKEY;
  private twitterConsumerSecret: any = process.env.VUE_APP_TWITTERCONSUMERKEY;
  private twitterAccessToken: any = process.env.VUE_APP_TWITTERTOKEN;
  private twitterAccessTokenSecret: any = process.env.VUE_APP_TWITTERTOKENSECRET;
  private twitterBearerToken: any = process.env.VUE_APP_TWITTERBEARERTOKEN;
  private requestTokenURL: string = 'https://cors-anywhere.herokuapp.com/https://api.twitter.com/oauth/request_token';
  private accessTokenURL: string = 'https://api.twitter.com/oauth/access_token';
  private authorizeURL: string = 'https://api.twitter.com/oauth/authorize';
  private endpointURL: string = 'https://api.twitter.com/labs/2/tweets';
  private requestParams = {
    'ids': '1138505981460193280',
    'tweet.fields': 'created_at',
  };

  public async created() {
    // console.log(this.twitterConsumerKey);
    this.params = new URLSearchParams(document.location.search.substring(1));
    const nameParam = this.params.get('username');
    if (nameParam) {
      this.username = nameParam;
      // this.testGeneratePoem();
    }
    const Twitter = require('twitter-lite');
    const app = new Twitter({
      subdomain: 'cors-anywhere.herokuapp.com/https://api',
      bearer_token: this.twitterBearerToken,
    });
    const params = {
      screen_name: this.screenname,
      count: 200,
      tweet_mode: 'extended',
    };
    try {
      const response = await app.get('statuses/user_timeline', params);
      this.tweets = response;
    } catch (e) {
      this.error = e;
    }
  }

  private testGeneratePoem() {
    this.ready = false;
    this.generating = true;
    // console.log(this.nameParam);
    if (this.nameParam === '' && this.params !== null) {
      // console.log('yolo');
      // this.params.set('username', encodeURI(this.username));
      const encodedUsername: string = encodeURI(this.username);
      // console.log(encodedUsername);
      // document.location.search = `?username=${encodedUsername}`;
    }
    this.author = this.tweets[0].user.name;
    this.sentences = [];
    this.tweets.forEach( (t: any) => {
      const tweet: string = t.full_text;
      const tweetAsSentences: string[] = tweet.replace(/([.?!])\s*(?=[A-Z])/g, '$1|').split('|');
      tweetAsSentences.forEach((s) => {
        const sentenceObject = {
          text: s,
          syllables: syllable(s),
        };
        this.sentences.push(sentenceObject);
      });
    });
    this.generateTitle();
    this.generating = false;
    this.ready = true;
  }

  private generateTitle() {
    let syllables: number = 0;
    let s: any | null = null;
    while (syllables < 6 || syllables > 15) {
      const i = this.pickARandomSentenceIndex();
      s = this.sentences[i];
      syllables = s.syllables;
    }
    this.title = s.text;
  }

  private pickARandomSentenceIndex() {
    const index = Math.floor(Math.random() * Math.floor(this.sentences.length));
    return index;
  }

  private get atUsername() {
    const first = this.username.charAt(0);
    if (first === '@') {
      return this.username;
    } else {
      return `@${this.username}`;
    }
  }

  private get screenname() {
    const first = this.username.charAt(0);
    if (first === '@') {
      return this.username.substring(1);
    } else {
      return this.username;
    }
  }
}
</script>

<style lang="scss">
.form-wrapper {
  padding: 2em;

  background-color: $violet-900;
}
.form {
  margin: 0 auto;
  width: 24em;
  max-width: 90vw;
  &__fields,
  &__submit {
    width: 100%;
  }
  &__fields {
    margin-bottom: 1em;
    overflow: hidden;

    border-radius: 3px;
    box-shadow: 0 1px 3px rgba(0,0,0,.2);
  }
  .field {
    label {
      margin-right: .5em;

      color: $violet-300;
    }
    &--text {
      padding: .75em;

      background-color: $violet-800;
      label,
      input
      .twitter-logo {
        display: inline-block;
      }
      label {
        width: 5.5em;
      }
      input {
        width: calc(100% - 5.5em - 40px);

        background-color: transparent;
        border: none;

        color: $violet-100;
        &::placeholder {
          opacity: 1;

          color: $violet-500;
        }
      }
      .twitter-logo {
        vertical-align: bottom;
        path {
          fill: $violet-900;
        }
      }
    }
    &--checkbox {
      padding: .75em .75em .25em;

      background-color: $violet-700;
      &:last-child {
        padding-bottom: .75em;
      }
      input {
        position: absolute;
        top: auto;

        width: 1px;
        height: 1px;
        overflow: hidden;

        clip: rect(1px 1px 1px 1px); /* IE 6/7 */
        clip: rect(1px, 1px, 1px, 1px);

        white-space: nowrap;
        &:focus {
          + label::before {
            border-color: $violet-100;
          }
        }
        &:checked {
          + label::after {
            content: '';

            position: absolute;
            top: 3px;
            left: 4px;

            display: block;
            height: 11px;
            width: 11px;

            background-color: $yellow;
          }
          + label::before {
            background-color: transparent;
            border-color: $yellow;
          }
        }
      }
      label {
        position: relative;

        padding-left: 27px;

        color: $violet-100;
        &::before {
          content: '';

          position: absolute;
          top: -1px;
          left: 0;

          display: block;
          height: 15px;
          width: 15px;

          background-color: $violet-700;
          border: 2px solid $violet-300;
          cursor: pointer;
        }
      }
      .infobox {
        padding: .5em;

        background-color: $violet-900;

        color: $violet-300;
        font-size: .8em;
      }
    }
  }
  &__submit {
    button {
      display: block;
      height: 2rem;
      padding: 2px 1em 0;
      width: 100%;

      background-color: $yellow;
      border: none;
      border-radius: 3px;
      box-shadow: 0 1px 3px rgba(0,0,0,.2);
      cursor: pointer;

      color: $violet-700;
      font-size: .8em;
      font-weight: 700;
      line-height: 2rem;
      text-transform: uppercase;

      transition: all .2s ease-out;
      &:hover {
        box-shadow: 0 1px 2px rgba(0,0,0,.6);
        transform: translateY(1px);
      }
      &:active {
        box-shadow: 0 1px 1px rgba(0,0,0,.7);
        transform: translateY(2px);
      }
      &.loading {
        position: relative;
        color: transparent;
        &::after {
          content: '';

          position: absolute;
          top: 50%;
          left: 50%;
          z-index: 2;

          display: block;
          height: 100%;
          width: 2em;

          background: transparent url('../assets/images/loader-dots.svg') no-repeat center center;
          background-size: 100% auto;

          transform: translate(-50%, -50%);
        }
      }
    }
  }
}
.poem,
.debug {
  padding: 2em;
}
</style>
