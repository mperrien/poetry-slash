<template>
  <div class="home">
    <div class="form-wrapper">
      <form class="form">
        <div class="form__field">
          <label for="username">Twitter user</label>
          <input type="text" name="username" id="username" placeholder="@realDonaldTrump" required v-model="username">
        </div>
        <div class="form__submit">
          <button type="submit" :class="{'loading': generating}" @click.prevent='testGeneratePoem'>Generate Poem</button>
        </div>
      </form>
    </div>
    <div class="result">
      <div>{{ sanitizedUsername }}</div>
      <div>{{ sentences }}</div>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator';
// @ is an alias to /src
import Header from '@/components/Header.vue';

@Component({
  components: {
    Header,
  },
})

export default class TokenForm extends Vue {
  private generating: boolean = false;
  private params: URLSearchParams | null = null;
  private nameParam: string = '';
  private username: string = '';
  private sentences: any[] = [];

  public async created() {
    this.params = new URLSearchParams(document.location.search.substring(1));
    const nameParam = this.params.get('username');
    if (nameParam) {
      this.username = nameParam;
      this.testGeneratePoem();
    }
  }

  private testGeneratePoem() {
    this.generating = true;
    console.log(this.nameParam);
    if (this.nameParam === '' && this.params !== null) {
      console.log('yolo');
      // this.params.set('username', this.username);
      // document.location.search = `?username=${this.username}`;
    }
    this.sentences = [];
    // Test data
    const tweets = [
      {
        created_at: 'Thu May 10 15:24:15 +0000 2018',
        id_str: '850006245121695744',
        text: 'Who knows what this means, but it sounds good to me!',
        user: {},
        place: {},
        entities: {},
        extended_entities: {},
      },
      {
        created_at: 'Thu May 10 15:24:15 +0000 2018',
        id_str: '850006245121695744',
        text: 'The New York Times is an embarrassment to journalism. They were a dead paper before I went into politics, and they will be a dead paper afte...',
        display_text_range: [0, 140],
        truncated: true,
        user: {},
        place: {},
        extended_tweet: {
          full_text: 'The New York Times is an embarrassment to journalism. They were a dead paper before I went into politics, and they will be a dead paper after I leave, which will be in 5 years. Fake News is the Enemy of the people!',
          display_text_range: [0, 214],
          entities: {},
        },
        entities: {},
        extended_entities: {},
      },
      {
        created_at: 'Thu May 10 15:24:15 +0000 2018',
        id_str: '850006245121695744',
        text: 'We have a perfectly coordinated and fine tuned plan at the White House for our attack on CoronaVirus. We moved VERY early to close borders t...',
        display_text_range: [0, 140],
        truncated: true,
        user: {},
        place: {},
        extended_tweet: {
          full_text: 'We have a perfectly coordinated and fine tuned plan at the White House for our attack on CoronaVirus. We moved VERY early to close borders to certain areas, which was a Godsend. V.P. is doing a great job. The Fake News Media is doing everything possible to make us look bad. Sad!',
          display_text_range: [0, 279],
          entities: {},
        },
        entities: {},
        extended_entities: {},
      },
      {
        created_at: 'Thu May 10 15:24:15 +0000 2018',
        id_str: '850006245121695744',
        text: 'Thank you @SenatorTimScott!',
        user: {},
        place: {},
        entities: {},
        extended_entities: {},
      },
      {
        created_at: 'Thu May 10 15:24:15 +0000 2018',
        id_str: '850006245121695744',
        text: 'Sleepy Joe in St. Louis, Missouri today: “We can only re-elect @realDonaldTrump.” #KAG2020LandslideVictory',
        user: {},
        place: {},
        entities: {},
        extended_entities: {},
      },
    ];
    // const tweetsContent: string[] = [];
    tweets.forEach( (t) => {
      let tweet: string = '';
      if (t.truncated === true) {
        tweet = t.extended_tweet.full_text;
      } else {
        tweet = t.text;
      }
      const tweetAsSentences: string[] = tweet.replace(/([.?!])\s*(?=[A-Z])/g, '$1|').split('|');
      tweetAsSentences.forEach((s) => {
        this.sentences.push(s);
      });
    });
    this.generating = false;
  }

  private get sanitizedUsername() {
    const first = this.username.charAt(0);
    if (first === '@') {
      return this.username;
    } else {
      return `@${this.username}`;
    }
  }
}
</script>

<style lang="scss">
.form-wrapper {
  padding: 2em;

  background-color: darken($violet, 15);
}
.form {
  margin: 0 auto;
  width: 20em;
  max-width: 90vw;
  &__field,
  &__submit {
    width: 100%;
  }
  &__field {
    margin-bottom: 1em;
    label,
    input {
      display: block;
      width: 100%;
    }
    label {
      color: lighten($violet, 35);
    }
    input {
      height: 2em;
      padding: 0 .5em;

      background-color: $violet;
      border: none;
      border-radius: 3px;
      box-shadow: 0 3px 12px rgba(0,0,0,.3);

      color: white;
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
      box-shadow: 0 3px 12px rgba(0,0,0,.3);
      cursor: pointer;

      color: $violet;
      font-size: .8em;
      font-weight: 700;
      line-height: 2rem;
      text-transform: uppercase;

      transition: all .2s ease-out;
      &:hover {
        box-shadow: 0 2px 6px rgba(0,0,0,.6);
        transform: translateY(1px);
      }
      &:active {
        box-shadow: 0 1px 3px rgba(0,0,0,.7);
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
.result {
  padding: 2em;
}
</style>
