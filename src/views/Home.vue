<template>
  <div class="home">
    <div class="form-wrapper">
      <form class="form">
        <div class="form__fields">
          <div class="fetch-options">
            <div class="field field--text field--dark">
              <label for="username">Twitter user</label>
              <input type="text" name="username" id="username" required placeholder="@username" v-model="username">
              <svg class="twitter-logo" xmlns="http://www.w3.org/2000/svg" width="30" height="24.333"><path d="M9.5 24.333C20.833 24.333 27 15 27 6.833V6a13.548 13.548 0 003-3.167 13.836 13.836 0 01-3.5 1A6.484 6.484 0 0029.167.5 15.287 15.287 0 0125.333 2a5.952 5.952 0 00-4.5-2 6.262 6.262 0 00-6.167 6.167 3.249 3.249 0 00.167 1.333A17.231 17.231 0 012.167 1a6.383 6.383 0 00-.833 3.167A6.622 6.622 0 004 9.333 5.619 5.619 0 011.167 8.5a6.091 6.091 0 005 6 5.138 5.138 0 01-1.667.167 2.836 2.836 0 01-1.167-.167 6.314 6.314 0 005.833 4.333A12.584 12.584 0 011.5 21.5a4.614 4.614 0 01-1.5-.167 15.731 15.731 0 009.5 3" fill-rule="evenodd"/></svg>
            </div>
            <div class="field field--select field--dark">
              <label for="number" class="screen-reader-text">Number of tweets to retrieve</label>
              <span>Fetch</span>
              <select name="number" id="number" required v-model="numberOfTweets">
                <option value="1">200</option>
                <option value="2">400</option>
                <option value="3">600</option>
                <option value="4">800</option>
                <option value="5">1000</option>
              </select>
              <span>tweets</span>
            </div>
            <div class="field field--checkbox field--dark">
              <input type="checkbox" id="shortlines" name="shortlines" v-model="shortlines">
              <label for="shortlines">Remove sentences longer than 20 syllables (recommended).</label>
            </div>
          </div>
          <div class="poem-options">
            <div class="field field--checkbox field--light" v-if="false">
              <input type="checkbox" id="rhymes" name="rhymes" v-model="rhymes">
              <label for="rhymes"><strong>Try</strong> to make it rhyme</label>
            </div>
            <div class="field field--checkbox field--light">
              <input type="checkbox" id="norepeats" name="norepeats" v-model="noRepeats">
              <label for="norepeats">Don't use the same line twice</label>
            </div>
            <div class="field field--radio field--light">
              <input type="radio" id="freeform" name="style" value="freeform" v-model="style">
              <label for="freeform">No particular rules (recommended)</label>
            </div>
            <div class="field field--radio field--light">
              <input type="radio" id="haiku" name="style" value="haiku" v-model="style">
              <label for="haiku">Haiku <button class="info-button" @click.prevent="displayHaikuInfo = !displayHaikuInfo"><span class="screen-reader-text">Help</span>?</button></label>
              <p class="infobox" v-show="displayHaikuInfo">“Haiku is a very short form of Japanese poetry. Traditional haiku often consist of 17 on (also known as morae though often loosely translated as "syllables"), in three phrases of 5, 7, and 5 on, respectively.“ says Wikipedia.</p>
            </div>
            <div class="field field--radio field--light">
              <input type="radio" id="alexandrine" name="style" value="alexandrine" v-model="style">
              <label for="alexandrine">Alexandrines (at your own risk...) <button class="info-button" @click.prevent="displayAlexandrinesInfo = !displayAlexandrinesInfo"><span class="screen-reader-text">Help</span>?</button></label>
              <p class="infobox" v-show="displayAlexandrinesInfo">“The French alexandrine is a syllabic poetic meter of 12 syllables with a medial caesura dividing the line into two hemistichs (half-lines) of six syllables each.“ says Wikipedia.</p>
            </div>
          </div>
        </div>
        <div class="form__submit">
          <button type="submit" :class="{'loading': generating}" @click.prevent='generatePoem'>Generate Poem</button>
        </div>
      </form>
    </div>
    <div class="container">
      <div class="error" v-if="error.length !== 0">
        <p>
          <strong>The poem couldn't be generated.</strong><br/>
          Here's what our advanced errors-handling system has to say about it:
          <ul>
            <li v-for="e in error" :key="e">
              {{ e }}
            </li>
          </ul>
        </p>
      </div>
      <div class="poem" v-if="ready">
        <h2 class="poem__title" v-if="title !== ''">{{ title }}</h2>
        <div class="poem__author">A <slot v-if="style === 'haiku'">haiku</slot><slot v-else>poem</slot> by {{ author }}</div>
        <div v-html="poem"></div>
      </div>
      <div class="status" v-if="displayStatus">
        <p v-html="status"></p>
      </div>
      <div class="debug" v-if="debug">
        <div>Environment: {{ environment }}</div>
        <div>{{ atUsername }}</div>
        <div>{{ screenname }}</div>
        <div>Rhymes: {{ rhymes }}</div>
        <div>Style: {{ style }}</div>
        <div>Exclude longer sentences: {{ shortlines }}</div>
        <div v-if="tweets !== null">Number of tweets retrieved: {{ tweets.length }}</div>
        <div v-if="sentences !== null">Number of sentences: {{ sentences.length }}</div>
        <div v-if="style === 'alexandrine'">
          Number of alexandrines: {{ alexandrines.length }}<br/>
          Number of half-alexandrines: {{ halfAlexandrines.length }}<br/>
        </div>
        <div v-if="style === 'haiku'">
          Number of Sevens: {{ sevenSyllables.length }}<br/>
          Number of Fives: {{ fiveSyllables.length }}<br/>
        </div>
      </div>
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
  private numberOfTweets: number = 1;
  private oldestTweet: number = 0;
  private twitterParams: {} = {};

  private rhymes: boolean = false;
  private noRepeats: boolean = true;
  private shortlines: boolean = true;
  private style: string = 'freeform';
  private displayAlexandrinesInfo: boolean = false;
  private displayHaikuInfo: boolean = false;
  private tweets: any[] = [];
  private error: any[] = [];
  private sentences: any[] = [];
  private alexandrines: any[] = [];
  private halfAlexandrines: any[] = [];
  private sevenSyllables: any[] = [];
  private fiveSyllables: any[] = [];
  private title: string = '';
  private author: string = '';
  private poem: string = '';
  private displayStatus: boolean = true;
  private status: string = '';

  private debug: boolean = true;
  private environment: string = process.env.NODE_ENV;

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
    this.params = new URLSearchParams(document.location.search.substring(1));
    const nameParam = this.params.get('username');
    if (nameParam) {
      this.username = nameParam;
      // this.testGeneratePoem();
    }
  }

  private async fetchTweets() {
    this.status = this.status.concat('Fetching tweets...', '<br/>');
    // TODO Maybe use users/show endpoint to check if user exists or is private...
    const Twitter = require('twitter-lite');
    const app = new Twitter({
      subdomain: 'cors-anywhere.herokuapp.com/https://api',
      bearer_token: this.twitterBearerToken,
    });
    for (let i: number = 0; i < this.numberOfTweets; i++ ) {
      if (this.oldestTweet === 0) {
        this.twitterParams = {
          screen_name: this.screenname,
          count: 200,
          tweet_mode: 'extended',
          include_rts: false,
        };
      } else {
        this.twitterParams = {
          screen_name: this.screenname,
          count: 200,
          tweet_mode: 'extended',
          include_rts: false,
          max_id: this.oldestTweet,
        };
      }
      try {
        const response = await app.get('statuses/user_timeline', this.twitterParams);
        if (response.error) {
          // Because I don't seem to be catching error codes from the API...
          this.error.push(`Problem while fetching tweets from ${this.atUsername}. Maybe try again with another user.`);
        }
        // We're going to do another API call to get more tweets to work from.
        this.oldestTweet = response[response.length - 1].id;
        const tempArray = this.tweets;
        this.tweets = tempArray.concat(response);
      } catch (e) {
        // Twitter API errors are catched earlier.
        // Errors here probably a consequence of those.
        // Fail silently
        this.generating = false;
      }
    }
    this.generating = false;
    this.status = this.status.concat(`Finished fetching ${this.tweets.length} tweets.`, `<br/>`);
  }

  private async extractSentences() {
    this.status = this.status.concat(`Extracting sentences from ${this.tweets.length} tweets...`, `<br/>`);
    this.tweets.forEach( (t: any) => {
      const text = t.full_text;
      // TODO Better filter usernames at begining of tweets.
      const tweetAsSentences: string[] = text.replace(/([.?!])\s*(?=[A-Z])/g, '$1|').split('|');
      tweetAsSentences.forEach( async (s) => {
        if (!s.includes('://')) { // Filter out sentences with links.
          const cleanSentence = this.removeUsernamesAtSentenceStart(s);
          const sentenceObject = {
            text: cleanSentence,
            syllables: syllable(s),
          };
          if (sentenceObject.syllables > 2) {
            if (this.style === 'freeform') {
              if (this.shortlines) {
                if (sentenceObject.syllables < 21) {
                  this.sentences.push(sentenceObject);
                }
              } else {
                this.sentences.push(sentenceObject);
              }
            } else if (this.style === 'alexandrine') {
              if (sentenceObject.syllables === 16) {
                this.alexandrines.push(sentenceObject);
              } else if (sentenceObject.syllables === 8) {
                this.halfAlexandrines.push(sentenceObject);
              }
            } else if (this.style === 'haiku') {
              if (sentenceObject.syllables === 7) {
                this.sevenSyllables.push(sentenceObject);
              } else if (sentenceObject.syllables === 5) {
                this.fiveSyllables.push(sentenceObject);
              }
            }
          }
        }
      });
    });
    if (this.style === 'freeform') {
      this.status = this.status.concat(`${this.sentences.length} sentences extracted.`, `<br/>`);
    } else if (this.style === 'alexandrine') {
      this.status = this.status.concat(`${this.alexandrines.length} alexandrines and ${this.halfAlexandrines.length} half alexandrines extracted.`, `<br/>`);
    } else if (this.style === 'haiku') {
      this.status = this.status.concat(`${this.sevenSyllables.length} seven-syllable lines and ${this.fiveSyllables.length} five-syllable lines extracted.`, `<br/>`);
    }
  }

  private async generatePoem() {
    // TODO Don't make the alexandrines ? a button, because it prevents submitting the form with return key
    this.ready = false;
    this.generating = true;
    this.resetData();
    if (this.nameParam === '' && this.params !== null) {
      // this.params.set('username', encodeURI(this.username));
      const encodedUsername: string = encodeURI(this.username);
      // document.location.search = `?username=${encodedUsername}`;
    }
    try {
      await this.fetchTweets();
      this.author = this.tweets[0].user.name;
      await this.extractSentences();
      if (this.style === 'freeform') {
        this.status = this.status.concat('Creating poem...', '<br/>');
        const numberOfPossibleStrophes = Math.floor(this.sentences.length / 4);
        const numberOfStrophesToGenerate = Math.min(numberOfPossibleStrophes, 4);
        for (let i = 0; i < numberOfStrophesToGenerate; i++) {
          this.poem = this.poem.concat('<p>');
          for (let j = 0; j < 4; j++) {
            const index = this.pickARandomSentenceIndex(this.sentences);
            this.poem = this.poem.concat(this.sentences[index].text, '<br/>');
            if (this.noRepeats === true) {
              this.sentences.splice(index, 1);
            }
          }
          this.poem = this.poem.concat('</p>');
        }
        this.generateTitle(this.sentences);
      } else if (this.style === 'alexandrine') {
        this.status = this.status.concat('Creating poem in alexandrines...', '<br/>');
        const numberOfLines = this.alexandrines.length + Math.floor(this.halfAlexandrines.length / 2);
        const numberOfFullStrophes = Math.floor(numberOfLines / 4);
        const numberOfStrophesToGenerate = Math.min(numberOfFullStrophes, 4);
        for (let i = 0; i < numberOfStrophesToGenerate; i++) {
          this.poem = this.poem.concat('<p>');
          for (let j = 0; j < 4; j++) {
            if (j % 2 === 0) {
              // For first and third lines, try to pick a full alexandrine.
              // If none are availables, pick two half-alexandrines.
              if (this.alexandrines.length > 0) {
                const index = this.pickARandomSentenceIndex(this.alexandrines);
                this.poem = this.poem.concat(this.alexandrines[index].text, '<br/>');
                if (this.noRepeats === true) {
                  this.alexandrines.splice(index, 1);
                }
              } else {
                const indexA = this.pickARandomSentenceIndex(this.halfAlexandrines);
                this.poem = this.poem.concat(this.halfAlexandrines[indexA].text);
                if (this.noRepeats === true) {
                  this.halfAlexandrines.splice(indexA, 1);
                }
                const indexB = this.pickARandomSentenceIndex(this.halfAlexandrines);
                this.poem = this.poem.concat(this.halfAlexandrines[indexB].text, '<br/>');
                if (this.noRepeats === true) {
                  this.halfAlexandrines.splice(indexB, 1);
                }
              }
            } else {
              // For second and fourth line, do the opposite.
              if (this.halfAlexandrines.length > 1) { // We need two to make a line
                const indexA = this.pickARandomSentenceIndex(this.halfAlexandrines);
                this.poem = this.poem.concat(this.halfAlexandrines[indexA].text);
                if (this.noRepeats === true) {
                  this.halfAlexandrines.splice(indexA, 1);
                }
                const indexB = this.pickARandomSentenceIndex(this.halfAlexandrines);
                this.poem = this.poem.concat(this.halfAlexandrines[indexB].text, '<br/>');
                if (this.noRepeats === true) {
                  this.halfAlexandrines.splice(indexB, 1);
                }
              } else {
                const index = this.pickARandomSentenceIndex(this.alexandrines);
                this.poem = this.poem.concat(this.alexandrines[index].text, '<br/>');
                if (this.noRepeats === true) {
                  this.alexandrines.splice(index, 1);
                }
              }
            }
          }
          this.poem = this.poem.concat('</p>');
        }
        if (numberOfStrophesToGenerate < 3) {
          // If we could only generate two full strophes,
          // Generate a third, incomplete, one.
          const numberOfRemainingLines = numberOfLines % numberOfFullStrophes;
          this.poem = this.poem.concat('<p>');
          for (let j = 0; j < numberOfRemainingLines; j++) {
            if (j % 2 === 0) {
              // For first and third lines, try to pick a full alexandrine.
              // If none are availables, pick two half-alexandrines.
              if (this.alexandrines.length > 0) {
                const index = this.pickARandomSentenceIndex(this.alexandrines);
                this.poem = this.poem.concat(this.alexandrines[index].text, '<br/>');
                if (this.noRepeats === true) {
                  this.alexandrines.splice(index, 1);
                }
              } else {
                const indexA = this.pickARandomSentenceIndex(this.halfAlexandrines);
                this.poem = this.poem.concat(this.halfAlexandrines[indexA].text);
                if (this.noRepeats === true) {
                  this.halfAlexandrines.splice(indexA, 1);
                }
                const indexB = this.pickARandomSentenceIndex(this.halfAlexandrines);
                this.poem = this.poem.concat(this.halfAlexandrines[indexB].text, '<br/>');
                if (this.noRepeats === true) {
                  this.halfAlexandrines.splice(indexB, 1);
                }
              }
            } else {
              // For second and fourth line, do the opposite.
              if (this.halfAlexandrines.length > 1) { // We need two to make a line
                const indexA = this.pickARandomSentenceIndex(this.halfAlexandrines);
                this.poem = this.poem.concat(this.halfAlexandrines[indexA].text);
                if (this.noRepeats === true) {
                  this.halfAlexandrines.splice(indexA, 1);
                }
                const indexB = this.pickARandomSentenceIndex(this.halfAlexandrines);
                this.poem = this.poem.concat(this.halfAlexandrines[indexB].text, '<br/>');
                if (this.noRepeats === true) {
                  this.halfAlexandrines.splice(indexB, 1);
                }
              } else {
                const index = this.pickARandomSentenceIndex(this.alexandrines);
                this.poem = this.poem.concat(this.alexandrines[index].text, '<br/>');
                if (this.noRepeats === true) {
                  this.alexandrines.splice(index, 1);
                }
              }
            }
          }
          this.poem = this.poem.concat('</p>');

        }
        this.generateTitle(this.sentences);
      } else if (this.style === 'haiku') {
        this.status = this.status.concat('Creating haiku...', '<br/>');
        if (this.fiveSyllables.length > 1 && this.sevenSyllables.length > 0) {
          this.poem = this.poem.concat('</p>');
          const indexA = this.pickARandomSentenceIndex(this.fiveSyllables);
          this.poem = this.poem.concat(this.fiveSyllables[indexA].text, '<br/>');
          if (this.noRepeats === true) {
            this.fiveSyllables.splice(indexA, 1);
          }
          const indexB = this.pickARandomSentenceIndex(this.sevenSyllables);
          this.poem = this.poem.concat(this.sevenSyllables[indexB].text, '<br/>');
          if (this.noRepeats === true) {
            this.sevenSyllables.splice(indexB, 1);
          }
          const indexC = this.pickARandomSentenceIndex(this.fiveSyllables);
          this.poem = this.poem.concat(this.fiveSyllables[indexC].text, '<br/>');
          if (this.noRepeats === true) {
            this.fiveSyllables.splice(indexC, 1);
          }
          this.poem = this.poem.concat('</p>');
        } else {
          this.error.push('Not enough lines with 5 or 7 syllables. Couldn\'t generate a haiku.');
        }
      }
      this.generating = false;
      this.ready = true;
    } catch (e) {
      this.error.push('Error while generating poem. Maybe try again with a different user or different settings.');
      this.generating = false;
    }
    this.status = this.status.concat('Done!', '<br/>');
  }

  private generateTitle(source: any[]) {
    let syllables: number = 0;
    let s: any | null = null;
    while (syllables < 6 || syllables > 15) {
      const i = this.pickARandomSentenceIndex(source);
      s = this.sentences[i];
      syllables = s.syllables;
    }
    this.title = s.text;
  }

  private resetData() {
    this.oldestTweet = 0;
    this.tweets = [];
    this.error = [];
    this.sentences = [];
    this.halfAlexandrines = [];
    this.sevenSyllables = [];
    this.fiveSyllables = [];
    this.poem = '';
    this.title = '';
    this.status = '';
  }

  private pickARandomSentenceIndex(source: any[]) {
    const index = Math.floor(Math.random() * Math.floor(source.length));
    return index;
  }

  private removeUsernamesAtSentenceStart(str: string) {
    if (str.charAt(0) === '@') {
      while (str.charAt(0) === '@') {
        const newStr = str.split(' ').slice(1).join(' ');
        str = newStr;
      }
      return str;
    } else {
      return str;
    }
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
    &--dark {
      background-color: $violet-800;
    }
    &--light {
      background-color: $violet-700;
    }
    &--text {
      padding: .75em .75em .25em;
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
    &--select {
      padding: .75em;
      span,
      select {
        display: inline-block;
        margin-right: .5em;
      }
      span {
        color: $violet-100;
      }
      select {
        width: 4.5em;

        text-align: center;
      }
    }
    &--checkbox,
    &--radio {
      padding: .75em .75em .25em;
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
    &--radio {
      input {
        &:checked {
          + label::after {
            border-radius: 50%;
          }
        }
      }
      label {
        &::before {
          border-radius: 50%;
        }
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
.container {
  margin-right: auto;
  margin-left: auto;
  max-width: 40em;
}
.error {
  margin: 2em 0;
  padding: 0 2em 0 1.5em;

  border-left: .5em solid $error;
}
.poem,
.status,
.debug {
  margin: 2em 0;
  padding: 2em;
}
</style>
