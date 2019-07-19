<template>
  <div id="app">
    <router-view>
    </router-view>
  </div>
</template>

<script>
import Vue from 'vue'
import VueScrollTo from 'vue-scrollto'
import Vuelidate from 'vuelidate'
import vSelect from 'vue-select'
import Vuex from 'vuex'
import createPersistedState from 'vuex-persistedstate'
import VueNumeric from 'vue-numeric'

import container from './components/general/Container.vue'
import section from './components/general/SectionBlock.vue'
import block from './components/general/RegularSectionBlock.vue'
import button from './components/general/Button.vue'
import modal from './components/general/Modal.vue'
import sectionTitle from './components/general/SectionTitle.vue'
import currencyTable from './components/CurrencyTable.vue'
import converter from './components/Converter.vue'
import functionality from './components/Functionality.vue'
import features from './components/Features.vue'
import metrics from './components/Metrics.vue'
import solutions from './components/Solutions.vue'
import access from './components/Accessibility.vue'
import speed from './components/Speed.vue'
import domiciled from './components/Domiciled.vue'
import quotes from './components/Quotes.vue'
import dropdown from './components/general/Dropdown.vue'
import cSelect from './components/general/CustomSelect.vue'
import openAccount from './components/OpenAccount.vue'

Vue.component('container', container)
Vue.component('section-block', section)
Vue.component('block', block)
Vue.component('gozo-button', button)
Vue.component('modal', modal)
Vue.component('currencyTable', currencyTable)
Vue.component('converter', currencyTable)
Vue.component('functionality', functionality)
Vue.component('features', features)
Vue.component('metrics', metrics)
Vue.component('solutions', solutions)
Vue.component('access', access)
Vue.component('speed', speed)
Vue.component('domiciled', domiciled)
Vue.component('quotes', quotes)
Vue.component('sectionTitle', sectionTitle)
Vue.component('dropdown', dropdown)
Vue.component('v-select', vSelect)
Vue.component('cSelect', cSelect)
Vue.component('open-account', openAccount)


Vue.use(Vuelidate)
Vue.use(VueScrollTo, {
  container: "body",
  offset: 2,
})
Vue.use(Vuex)

var VueCookie = require('vue-cookie')
Vue.use(VueCookie)
Vue.use(VueNumeric)

const store = new Vuex.Store({
  state: {
    logIn: false,
    swiperAutoheight: true,
    invalid: true,
    userEmail: false,
  },
  mutations: {
    toggleLogin (state) {
      state.logIn = !state.logIn
    },
    toggleAutoheight (state) {
      state.swiperAutoheight = !state.logIn
    },
    setInvalid (state) {
      state.invalid = true
    },
    removeInvalid (state) {
      state.invalid = false
    },
    setUserEmail (state, userEmail) {
      state.userEmail = userEmail
    },
  },
  plugins: [createPersistedState()]
});


export default {
  name: 'app',
  metaInfo: {
    title: 'GOZO',
    titleTemplate: '%s | GOZO'
  },
  store,
  components: {
  },
};

</script>

<style lang="scss">

@import "vue-select/src/scss/vue-select.scss";

/* add some normalize rules*/

HTML {
  @include bp(desktop) {
    line-height: $line-height-base;
  }

  font-family: $font-family-base;
  font-size: $font-size-base;
  line-height: $line-height-mobile;

  height: 100%;

  color: $color__text;

  -webkit-text-size-adjust: 100%;
}

BODY {
  position: relative;
  overflow-y: auto;

  height: 100%;
  margin: 0;
  * {
    box-sizing: border-box;
  }

  &.openedMenu,
  &.openedUser {
    overflow-y: hidden !important;
    height: 110% !important;

    .intro ~ *:not(.modal) {
      position: relative;
      z-index: -1;
    }

    &:after {
      content: '';
      position: fixed;
      top: 0;
      bottom: -2px;
      left: 0;
      right: 0;
      background: $color__darkblue;
    }
  }
}

A {
  background-color: transparent;
}

IMG {
  border-style: none;
}

button,
input,
OPTGROUP,
SELECT,
TEXTAREA {
  font-family: inherit;
  font-size: 100%;

  margin: 0;
}

button,
input {
  overflow: visible;
}

button,
SELECT {
  text-transform: none;
}

button,
[type='button'],
[type='reset'],
[type='submit'] {
  -webkit-appearance: button;
}

button::-moz-focus-inner,
[type='button']::-moz-focus-inner,
[type='reset']::-moz-focus-inner,
[type='submit']::-moz-focus-inner {
  padding: 0;

  border-style: none;
}

button:-moz-focusring,
[type='button']:-moz-focusring,
[type='reset']:-moz-focusring,
[type='submit']:-moz-focusring {
  outline: 1px dotted inherit;
}

.hidden {
  display: none;
}

/* Change Autocomplete styles in Chrome*/
@-webkit-keyframes autofill {
  to {
    color: $color__darkblue;
    background: $color__white;
  }
}

input:-webkit-autofill,
input:-webkit-autofill:hover,
input:-webkit-autofill:focus {
  -webkit-animation-name: autofill;

  -webkit-animation-fill-mode: both;
}

h1,
.h1 {
  @include h1;
}

h2 {
  @include media-css-lock(24, 30, 40);
  @include media-css-lock(80, 120, 120, margin-top);
  @include media-css-lock(50, 60, 120, margin-bottom);

  position: relative;

  color: $color__darkblue;

  span {
    color: $color__blue;
  }
}

h3 {
  @include media-css-lock(50, 60, 60, margin-top);
  @include media-css-lock(25, 25, 30, margin-bottom);

  font-size: 1rem;
  font-weight: bold;

  color: $color__darkblue;

  & + p,
  & + ul {
    margin-top: 0;
  }
}

ul {
  margin-top: 0;
  padding: 0;
  list-style: none;

  li {
    position: relative;
  }

  & > & {
    padding-left: 40px;
  }

  &.letters {
    counter-reset: list;

    & > li {
      &::before {
        content: counter(list, lower-alpha) ') ';
        counter-increment: list;
      }
    }
  }
  &.numbers--no-style {
    list-style: none;

    & > li {
      padding-left: 0;

      &::before {
        display: none;
      }
    }
  }

  &.numbers.numbers--start {
    counter-reset: list;

    & > li {
      &::before {
        content: attr(data-start) '.' counter(list, decimal);
        counter-increment: list;
      }
    }
  }
}

A {
  text-decoration: none;

  color: $color__blue;

  &:focus,
  &:hover {
    text-decoration: underline;
  }
}

#app {
  overflow-x: hidden; // IE11 fix

  min-height: 100%;
}

.data-number-before {
  &::before {
    margin-right: 40px;

    content: attr(data-number);
  }
}

.small-content {
  @include bp(tablet) {
    width: 100%;
    max-width: 510px;
    margin-right: auto;
    margin-left: auto;
  }
  @include bp(desktop) {
    max-width: 730px;
  }
}

.intro.section-block {
  & > .container {
    position: static;
  }
}

.subtitle {
  font-family: $font-family-secondary;
  color: $color__black;
  font-size: 18px;
  line-height: 1.39;
  display: block;
  margin-bottom: 20px;

  @include bp(mobile) {
    font-size: 20px;
  }
}

.desc {
  font-size: 16px;
  font-weight: 300;
  color: rgba($color__black, .75);
  margin: 0;
}

.vex.vex-theme-exante-signup {
  display: none !important;
}

// SIGNUP FORM styles

.become-client {

  .errorlist{
    color: $color__red;
    font-size: 14px;
    margin-top: 5px;
  }

  .field .field__input[aria-invalid="true"] {
    border-bottom: 2px solid $color__red !important;
  }

  // FLAGS

  .intl-number-input {
    position: relative
  }

  .intl-number-input .hide {
    display: none
  }

  .intl-number-input .flag-dropdown {
    position: absolute;
    cursor: pointer;
    z-index: 3;
  }

  .intl-number-input .flag-dropdown .selected-flag {
    margin: 1px;
    padding: 4px 16px 8px 0 !important;
  }

  .intl-number-input .flag-dropdown .selected-flag .down-arrow {
    position: relative;
    top: 6px;
    left: 20px;
    width: 0;
    height: 0;
    border-top: 4px solid #000;
    border-right: 4px solid transparent;
    border-left: 4px solid transparent
  }

  .intl-number-input .flag-dropdown .country-list {
    position: absolute;
    z-index: 10;
    top: 29px;
    overflow-y: scroll;
    width: 268px;
    height: 200px;
    margin: 0;
    padding: 0;
    list-style: none;
    border: 1px solid #ccc;
    background-color: #fff;
    box-shadow: 1px 1px 4px rgba(0, 0, 0, 0.2)
  }

  .intl-number-input .flag-dropdown .country-list .divider {
    margin-bottom: 5px;
    padding-bottom: 5px;
    border-bottom: 1px solid #ccc
  }

  .intl-number-input .flag-dropdown .country-list .country {
    padding: 4px 10px;
    line-height: 16px;
    font-size: 16px
  }

  .intl-number-input .flag-dropdown .country-list .country .dial-code {
    color: #999
  }

  .intl-number-input .flag-dropdown .country-list .flag {
    display: inline-block;
    margin-right: 6px;
    vertical-align: bottom
  }

  .intl-number-input .flag-dropdown .country-list .country-name {
    margin-right: 6px
  }

  .country-name {
    color: #000
  }

  .f16 .flag {
    width: 16px;
    height: 16px;
    background: url(https://exante.eu/static/i/dest/flags16.png?f734) no-repeat
  }

  .f16 .ad {
    background-position: 0 -352px
  }

  .f16 .ae {
    background-position: 0 -368px
  }

  .f16 .af {
    background-position: 0 -384px
  }

  .f16 .ag {
    background-position: 0 -400px
  }

  .f16 .ai {
    background-position: 0 -416px
  }

  .f16 .al {
    background-position: 0 -432px
  }

  .f16 .am {
    background-position: 0 -448px
  }

  .f16 .ao {
    background-position: 0 -480px
  }

  .f16 .ar {
    background-position: 0 -512px
  }

  .f16 .as {
    background-position: 0 -528px
  }

  .f16 .at {
    background-position: 0 -544px
  }

  .f16 .au {
    background-position: 0 -560px
  }

  .f16 .aw {
    background-position: 0 -576px
  }

  .f16 .az {
    background-position: 0 -592px
  }

  .f16 .ba {
    background-position: 0 -608px
  }

  .f16 .bb {
    background-position: 0 -624px
  }

  .f16 .bd {
    background-position: 0 -640px
  }

  .f16 .be {
    background-position: 0 -656px
  }

  .f16 .bf {
    background-position: 0 -672px
  }

  .f16 .bg {
    background-position: 0 -688px
  }

  .f16 .bh {
    background-position: 0 -704px
  }

  .f16 .bi {
    background-position: 0 -720px
  }

  .f16 .bj {
    background-position: 0 -736px
  }

  .f16 .bm {
    background-position: 0 -752px
  }

  .f16 .bn {
    background-position: 0 -768px
  }

  .f16 .bo {
    background-position: 0 -784px
  }

  .f16 .br {
    background-position: 0 -800px
  }

  .f16 .bs {
    background-position: 0 -816px
  }

  .f16 .bt {
    background-position: 0 -832px
  }

  .f16 .bw {
    background-position: 0 -848px
  }

  .f16 .by {
    background-position: 0 -864px
  }

  .f16 .bz {
    background-position: 0 -880px
  }

  .f16 .ca {
    background-position: 0 -896px
  }

  .f16 .cg {
    background-position: 0 -912px
  }

  .f16 .cf {
    background-position: 0 -928px
  }

  .f16 .cd {
    background-position: 0 -944px
  }

  .f16 .ch {
    background-position: 0 -960px
  }

  .f16 .ci {
    background-position: 0 -976px
  }

  .f16 .ck {
    background-position: 0 -992px
  }

  .f16 .cl {
    background-position: 0 -1008px
  }

  .f16 .cm {
    background-position: 0 -1024px
  }

  .f16 .cn {
    background-position: 0 -1040px
  }

  .f16 .co {
    background-position: 0 -1056px
  }

  .f16 .cr {
    background-position: 0 -1072px
  }

  .f16 .cu {
    background-position: 0 -1088px
  }

  .f16 .cv {
    background-position: 0 -1104px
  }

  .f16 .cy {
    background-position: 0 -1120px
  }

  .f16 .cz {
    background-position: 0 -1136px
  }

  .f16 .de {
    background-position: 0 -1152px
  }

  .f16 .dj {
    background-position: 0 -1168px
  }

  .f16 .dk {
    background-position: 0 -1184px
  }

  .f16 .dm {
    background-position: 0 -1200px
  }

  .f16 .do {
    background-position: 0 -1216px
  }

  .f16 .dz {
    background-position: 0 -1232px
  }

  .f16 .ec {
    background-position: 0 -1248px
  }

  .f16 .ee {
    background-position: 0 -1264px
  }

  .f16 .eg {
    background-position: 0 -1280px
  }

  .f16 .eh {
    background-position: 0 -1296px
  }

  .f16 .er {
    background-position: 0 -1312px
  }

  .f16 .es {
    background-position: 0 -1328px
  }

  .f16 .et {
    background-position: 0 -1344px
  }

  .f16 .fi {
    background-position: 0 -1360px
  }

  .f16 .fj {
    background-position: 0 -1376px
  }

  .f16 .fm {
    background-position: 0 -1392px
  }

  .f16 .fo {
    background-position: 0 -1408px
  }

  .f16 .fr {
    background-position: 0 -1424px
  }

  .f16 .ga {
    background-position: 0 -1440px
  }

  .f16 .gb {
    background-position: 0 -1456px
  }

  .f16 .gd {
    background-position: 0 -1472px
  }

  .f16 .ge {
    background-position: 0 -1488px
  }

  .f16 .gg {
    background-position: 0 -1504px
  }

  .f16 .gh {
    background-position: 0 -1520px
  }

  .f16 .gi {
    background-position: 0 -1536px
  }

  .f16 .gl {
    background-position: 0 -1552px
  }

  .f16 .gm {
    background-position: 0 -1568px
  }

  .f16 .gn {
    background-position: 0 -1584px
  }

  .f16 .gp {
    background-position: 0 -1600px
  }

  .f16 .gq {
    background-position: 0 -1616px
  }

  .f16 .gr {
    background-position: 0 -1632px
  }

  .f16 .gt {
    background-position: 0 -1648px
  }

  .f16 .gu {
    background-position: 0 -1664px
  }

  .f16 .gw {
    background-position: 0 -1680px
  }

  .f16 .gy {
    background-position: 0 -1696px
  }

  .f16 .hk {
    background-position: 0 -1712px
  }

  .f16 .hn {
    background-position: 0 -1728px
  }

  .f16 .hr {
    background-position: 0 -1744px
  }

  .f16 .ht {
    background-position: 0 -1760px
  }

  .f16 .hu {
    background-position: 0 -1776px
  }

  .f16 .id,
  .f16 .mc {
    background-position: 0 -1792px
  }

  .f16 .ie {
    background-position: 0 -1808px
  }

  .f16 .il {
    background-position: 0 -1824px
  }

  .f16 .im {
    background-position: 0 -1840px
  }

  .f16 .in {
    background-position: 0 -1856px
  }

  .f16 .iq {
    background-position: 0 -1872px
  }

  .f16 .ir {
    background-position: 0 -1888px
  }

  .f16 .is {
    background-position: 0 -1904px
  }

  .f16 .it {
    background-position: 0 -1920px
  }

  .f16 .je {
    background-position: 0 -1936px
  }

  .f16 .jm {
    background-position: 0 -1952px
  }

  .f16 .jo {
    background-position: 0 -1968px
  }

  .f16 .jp {
    background-position: 0 -1984px
  }

  .f16 .ke {
    background-position: 0 -2000px
  }

  .f16 .kg {
    background-position: 0 -2016px
  }

  .f16 .kh {
    background-position: 0 -2032px
  }

  .f16 .ki {
    background-position: 0 -2048px
  }

  .f16 .km {
    background-position: 0 -2064px
  }

  .f16 .kn {
    background-position: 0 -2080px
  }

  .f16 .kp {
    background-position: 0 -2096px
  }

  .f16 .kr {
    background-position: 0 -2112px
  }

  .f16 .kw {
    background-position: 0 -2128px
  }

  .f16 .ky {
    background-position: 0 -2144px
  }

  .f16 .kz {
    background-position: 0 -2160px
  }

  .f16 .la {
    background-position: 0 -2176px
  }

  .f16 .lb {
    background-position: 0 -2192px
  }

  .f16 .lc {
    background-position: 0 -2208px
  }

  .f16 .li {
    background-position: 0 -2224px
  }

  .f16 .lk {
    background-position: 0 -2240px
  }

  .f16 .lr {
    background-position: 0 -2256px
  }

  .f16 .ls {
    background-position: 0 -2272px
  }

  .f16 .lt {
    background-position: 0 -2288px
  }

  .f16 .lu {
    background-position: 0 -2304px
  }

  .f16 .lv {
    background-position: 0 -2320px
  }

  .f16 .ly {
    background-position: 0 -2336px
  }

  .f16 .ma {
    background-position: 0 -2352px
  }

  .f16 .md {
    background-position: 0 -2368px
  }

  .f16 .me {
    background-position: 0 -2384px
  }

  .f16 .mg {
    background-position: 0 -2400px
  }

  .f16 .mh {
    background-position: 0 -2416px
  }

  .f16 .mk {
    background-position: 0 -2432px
  }

  .f16 .ml {
    background-position: 0 -2448px
  }

  .f16 .mm {
    background-position: 0 -2464px
  }

  .f16 .mn {
    background-position: 0 -2480px
  }

  .f16 .mo {
    background-position: 0 -2496px
  }

  .f16 .mq {
    background-position: 0 -2512px
  }

  .f16 .mr {
    background-position: 0 -2528px
  }

  .f16 .ms {
    background-position: 0 -2544px
  }

  .f16 .mt {
    background-position: 0 -2560px
  }

  .f16 .mu {
    background-position: 0 -2576px
  }

  .f16 .mv {
    background-position: 0 -2592px
  }

  .f16 .mw {
    background-position: 0 -2608px
  }

  .f16 .mx {
    background-position: 0 -2624px
  }

  .f16 .my {
    background-position: 0 -2640px
  }

  .f16 .mz {
    background-position: 0 -2656px
  }

  .f16 .na {
    background-position: 0 -2672px
  }

  .f16 .nc {
    background-position: 0 -2688px
  }

  .f16 .ne {
    background-position: 0 -2704px
  }

  .f16 .ng {
    background-position: 0 -2720px
  }

  .f16 .ni {
    background-position: 0 -2736px
  }

  .f16 .nl {
    background-position: 0 -2752px
  }

  .f16 .no {
    background-position: 0 -2768px
  }

  .f16 .np {
    background-position: 0 -2784px
  }

  .f16 .nr {
    background-position: 0 -2800px
  }

  .f16 .nz {
    background-position: 0 -2816px
  }

  .f16 .om {
    background-position: 0 -2832px
  }

  .f16 .pa {
    background-position: 0 -2848px
  }

  .f16 .pe {
    background-position: 0 -2864px
  }

  .f16 .pf {
    background-position: 0 -2880px
  }

  .f16 .pg {
    background-position: 0 -2896px
  }

  .f16 .ph {
    background-position: 0 -2912px
  }

  .f16 .pk {
    background-position: 0 -2928px
  }

  .f16 .pl {
    background-position: 0 -2944px
  }

  .f16 .pr {
    background-position: 0 -2960px
  }

  .f16 .ps {
    background-position: 0 -2976px
  }

  .f16 .pt {
    background-position: 0 -2992px
  }

  .f16 .pw {
    background-position: 0 -3008px
  }

  .f16 .py {
    background-position: 0 -3024px
  }

  .f16 .qa {
    background-position: 0 -3040px
  }

  .f16 .re {
    background-position: 0 -3056px
  }

  .f16 .ro {
    background-position: 0 -3072px
  }

  .f16 .rs {
    background-position: 0 -3088px
  }

  .f16 .ru {
    background-position: 0 -3104px
  }

  .f16 .rw {
    background-position: 0 -3120px
  }

  .f16 .sa {
    background-position: 0 -3136px
  }

  .f16 .sb {
    background-position: 0 -3152px
  }

  .f16 .sc {
    background-position: 0 -3168px
  }

  .f16 .sd {
    background-position: 0 -3184px
  }

  .f16 .se {
    background-position: 0 -3200px
  }

  .f16 .sg {
    background-position: 0 -3216px
  }

  .f16 .si {
    background-position: 0 -3232px
  }

  .f16 .sk {
    background-position: 0 -3248px
  }

  .f16 .sl {
    background-position: 0 -3264px
  }

  .f16 .sm {
    background-position: 0 -3280px
  }

  .f16 .sn {
    background-position: 0 -3296px
  }

  .f16 .so {
    background-position: 0 -3312px
  }

  .f16 .sr {
    background-position: 0 -3328px
  }

  .f16 .st {
    background-position: 0 -3344px
  }

  .f16 .sv {
    background-position: 0 -3360px
  }

  .f16 .sy {
    background-position: 0 -3376px
  }

  .f16 .sz {
    background-position: 0 -3392px
  }

  .f16 .tc {
    background-position: 0 -3408px
  }

  .f16 .td {
    background-position: 0 -3424px
  }

  .f16 .tg {
    background-position: 0 -3440px
  }

  .f16 .th {
    background-position: 0 -3456px
  }

  .f16 .tj {
    background-position: 0 -3472px
  }

  .f16 .tl {
    background-position: 0 -3488px
  }

  .f16 .tm {
    background-position: 0 -3504px
  }

  .f16 .tn {
    background-position: 0 -3520px
  }

  .f16 .to {
    background-position: 0 -3536px
  }

  .f16 .tr {
    background-position: 0 -3552px
  }

  .f16 .tt {
    background-position: 0 -3568px
  }

  .f16 .tv {
    background-position: 0 -3584px
  }

  .f16 .tw {
    background-position: 0 -3600px
  }

  .f16 .tz {
    background-position: 0 -3616px
  }

  .f16 .ua {
    background-position: 0 -3632px
  }

  .f16 .ug {
    background-position: 0 -3648px
  }

  .f16 .us {
    background-position: 0 -3664px
  }

  .f16 .uy {
    background-position: 0 -3680px
  }

  .f16 .uz {
    background-position: 0 -3696px
  }

  .f16 .va {
    background-position: 0 -3712px
  }

  .f16 .vc {
    background-position: 0 -3728px
  }

  .f16 .ve {
    background-position: 0 -3744px
  }

  .f16 .vg {
    background-position: 0 -3760px
  }

  .f16 .vi {
    background-position: 0 -3776px
  }

  .f16 .vn {
    background-position: 0 -3792px
  }

  .f16 .vu {
    background-position: 0 -3808px
  }

  .f16 .ws {
    background-position: 0 -3824px
  }

  .f16 .ye {
    background-position: 0 -3840px
  }

  .f16 .za {
    background-position: 0 -3856px
  }

  .f16 .zm {
    background-position: 0 -3872px
  }

  .f16 .zw {
    background-position: 0 -3888px
  }
}

input[type='number'] {
    -moz-appearance:textfield;
    box-shadow: none;
}

input::-webkit-outer-spin-button,
input::-webkit-inner-spin-button {
    -webkit-appearance: none;
}

</style>
