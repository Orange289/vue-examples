<template>
  <div class="converter">
    <form class="converter__form" id="converter">
      <div class="converter__data converter__data--source">
        <div class="converter__select">
          <v-select
            :options="config.options"
            :searchable="false"
            :filterable="false"
            :clearable="false"
            @input="inputSource"
            v-model="config.source">
          </v-select>
        </div>
        <div class="converter__input converter__input--source">
          <input
            ref="converterInputSource"
            type="text"
            @keypress="checkInput($event)"
            @keydown="checkInput($event)"
            v-model="sourceValue"
            @blur="focusOut($event)"
            @focus="focusIn($event)">
          <span v-if="config.source" class="converter__currency">{{ config.source }}</span>
          <span :value="sourceValue | formatNumber" ref="sourceValueFormatted" class="hidden"></span>
        </div>
        <div class="converter__label">
          Transaction Fee: 0.25%
        </div>
      </div>
      <button class="converter__replace-btn" @click="onReplace">
        <svg><use xlink:href="#sprite-convert"></use></svg>
      </button>
      <div class="converter__data converter__data--result">
        <div class="converter__select">
          <v-select
            :options="config.options"
            :searchable="false"
            :filterable="false"
            :clearable="false"
            v-model="config.result"
            @input="inputResult">
          </v-select>
        </div>
        <div class="converter__input converter__input--result">
          <input
            ref="converterInputResult"
            type="text"
            @keypress="checkInput($event)"
            @keydown="checkInput($event)"
            v-model="resultValue"
            @blur="focusOut($event)"
            @focus="focusIn($event)">
          <span v-if="config.result" class="converter__currency">{{ config.result }}</span>
          <span :value="resultValue | formatNumber" ref="resultValueFormatted" class="hidden"></span>
        </div>
        <div class="converter__label">
          1 <span>{{ config.source }}</span> = <span>{{ rate }}</span> <span>{{ config.result }}</span>
        </div>
      </div>
    </form>
    <a
      :href="url"
      :class="['converter__exchange-btn', 'gozo-button', {'open-account': !signIn }]">Exchange</a>
  </div>
</template>

<script>
  import axios from 'axios';

  export default {
    name: 'converter',
    data() {
      return {
        config : {
          source: 'BTC',
          result: 'EUR',
          options: []
        },
        rate: null,
        sourceValue: null,
        resultValue: null,
      }
    },
    filters: {
      formatNumber: function (num) {
        if (num) {
          let numParts = num.toString().split('.');
          numParts[0] = numParts[0].replace(/\B(?=(\d{3})+(?!\d))/g, ',');
          return numParts.join('.');
        }
      }
    },
    computed: {
      source: function() {
        return this.config.source;
      },
      signIn () {
        return this.$store.state.logIn;
      },
      url () {
        let url = this.signIn ? '/clientsarea/account/summary/' : '#';
        return url;
      }
    },
    mounted() {
      let _self = this;
      let bridgeData = 'https://stage.gozo.pro/en/clientsarea/rest/bridge-data/';
      let source = this.sourceValue;
      let result = this.resultValue;

      axios
        .get(bridgeData)
        .then(function(response) {
          let currencies = response.data.data.currencies;

          currencies.forEach(el => {
            _self.config.options.push(el);
            _self.config.options.sort(_self.sortInstruments);
          })
        });

      this.setRate();
    },
    methods: {
      clearFields() {
        let convInputSource = this.$refs.converterInputSource;
        let convInputResult = this.$refs.converterInputResult;

        this.sourceValue = null;
        this.resultValue = null;

        // convInputSource.style.width='2ch';
        // convInputResult.style.width='2ch';
      },
      setRate() {
        let _self = this;
        let source = this.config.source;
        let result = this.config.result;
        let crossrateData = 'https://gozo.pro/en/clientsarea/rest/bridge-data/crossrate/';

        axios
          .get(`${crossrateData}${source}/${result}/`)
          .then(function(response){
            _self.rate = response.data.data.rate;
          });
      },
      decimalsFormat(el, rate, decimals) {
        return (Math.round(rate * el.replace(/,/g, '') * Math.pow(10, decimals)) / Math.pow(10, decimals));
      },
      getDecimalsNumber(num) {
        if (num.toString().includes('.')) {
          return num.toString().split('.').pop().length;
        }
      },
      calculateFromApi(from, to, x, y, ev) {
        const crossrateData = 'https://gozo.pro/en/clientsarea/rest/bridge-data/crossrate/';
        let rate;
        let rateDecimals;
        let _self = this;
        let formatFilter = _self.$options.filters.formatNumber;

        axios
          .get(`${crossrateData}${from}/${to}/`)
          .then(function(response){
            rate = response.data.data.rate;
            rateDecimals = _self.getDecimalsNumber(rate);

            x = ev.value;
            if (rateDecimals <= 8) {
              if(y === _self.resultValue) {
                _self.resultValue = formatFilter(_self.decimalsFormat(x, rate, rateDecimals));
              } else if (x === _self.resultValue) {
                _self.sourceValue = formatFilter(_self.decimalsFormat(x, rate, rateDecimals));
              }
            } else {
              if(y === _self.resultValue) {
                _self.resultValue = formatFilter(_self.decimalsFormat(x, rate, 8));
              } else if(x === _self.resultValue) {
                _self.sourceValue = formatFilter(_self.decimalsFormat(x, rate, 8));
              }
            }
            if((y === 0) || (y === 'NaN')) {
              if(y === _self.resultValue) {
                _self.resultValue = null;
              } else {
                _self.sourceValue = null;
              }
            };
          })
      },
      convertValues(from, to, ev) {
        const crossrateData = 'https://gozo.pro/en/clientsarea/rest/bridge-data/crossrate/';
        let _self = this;

        if (from === this.config.source) {
          _self.calculateFromApi(from, to, _self.sourceValue, _self.resultValue, ev);
        } else if (from === this.config.result) {
          _self.calculateFromApi(from, to, _self.resultValue, _self.sourceValue, ev);
        }
      },
      inputSource() {
        this.setRate();
        this.convertValues(this.config.source, this.config.result, this.$refs.converterInputSource);
      },
      inputResult() {
        this.setRate();
        this.convertValues(this.config.result, this.config.source, this.$refs.converterInputResult);
      },
      onReplace(e) {
        e.preventDefault();

        this.clearFields();

        let newSource;
        let newResult;

        newSource = this.config.result;
        newResult = this.config.source;

        this.config.result = newResult;
        this.config.source = newSource;

        this.setRate();
      },
      checkInput($event) {
        if ($event.charCode === 0 || /\d/.test(String.fromCharCode($event.charCode)) || $event.keyCode === 8 || $event.keyCode === 46) {
          let source = this.config.source;
          let result = this.config.result;

          if($event.target.parentNode.classList.contains('converter__input--source')) {
            this.convertValues(source, result, $event.target);
          } else {
            this.convertValues(result, source, $event.target);
          }
          return true;
        } else {
          $event.preventDefault();
        }
      },
      sortInstruments(a, b) {
        if(a < b) { return -1; }
        if(a > b) { return 1; }
        return 0;
      },
      focusIn($event) {
        if($event.target.parentNode.classList.contains('converter__input--source')) {
          if(this.sourceValue) {
            $event.target.value = this.sourceValue.replace(/,/g, '');
          }
        } else {
          if(this.resultValue) {
            $event.target.value = this.resultValue.replace(/,/g, '');
          }
        }
      },
      focusOut: function($event) {
        if($event.target.parentNode.classList.contains('converter__input--source')) {
          $event.target.value = this.$refs.sourceValueFormatted.getAttribute('value');
        } else {
          $event.target.value = this.$refs.resultValueFormatted.getAttribute('value');
        }
      }
    }
  }
</script>

<style lang="scss">

.converter {
  padding-top: 40px;
  padding-bottom: 37px;

  @include bp(tablet) {
    padding-top: 44px;
    padding-bottom: 126px;
  }

  @include bp(desktop) {
    padding-bottom: 20px;
  }

  // Vue-select styles override

  .v-select {
    width: 90px;
  }

  .vs__selected {
    font-weight: 600;
    letter-spacing: .3px;
    color: $color__blue;
  }

  .vs__dropdown-toggle {
    max-height: 36px;
    border-color: #1f374a;
    border-radius: 18px;
    background: #1f374a;
    padding: 2px 0 5px 0;
  }

  .vs__selected {
    margin-left: 8px;
  }

  .vs__actions {
    position: relative;

    svg {
      display: none;
    }

    &:before {
      position: absolute;
      top: 9px;
      right: 15px;

      width: 6px;
      height: 6px;

      content: '';
      transition: transform .3s;
      transform: rotate(-45deg);

      border-bottom: 1px solid $color__blue;
      border-left: 1px solid $color__blue;
    }
  }

  .vs--open .vs__actions:before {
    top: 13px;

    transform: rotate(135deg);
  }

  .vs--single {
    &.vs--open .vs__selected {
      opacity: 1;
    }
  }

  @include bp(mobile--small mobile) {
    .vs__dropdown-toggle {
      max-height: 30px;
      margin-bottom: 5px;
    }

    .v-select {
      max-width: 83px;
    }

    .vs__actions:before {
      top: 7px;
    }

    .vs__selected {
      margin-top: -2px;
    }

    .vs--open .vs__selected {
      margin-top: 1px;
    }
  }

  .vs__dropdown-menu {
    height: 400px;

    @include bp(0 tablet) {
      max-height: 200px;
    }

    @include bp(desktop) {
      right: 0;
      left: auto;
    }
  }

  .vs__dropdown-option {
    padding: 5px 20px;
    height: 31px;
  }
}

.converter__form {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding-bottom: 50px;

  @include bp(tablet) {
    // padding-bottom: 0;
  }

  @include bp(desktop) {
    width: 834px;
    margin: auto;
    // margin-bottom: 50px;
    justify-content: space-between;
  }
}

.converter__input {
  margin-bottom: 15px;
  border-bottom: 2px solid $color__blue;

  // @include bp(0 desktop) {
  //   input {
  //     width: 100% !important;
  //   }
  // }

  input {
    font-size: 20px;
    padding: 10px 0;
    font-weight: 300;
    color: $color__white;
    border: none;
    background: transparent;
    width: 100%;

    &:focus {
      outline: none;
    }
  }

  @include bp(mobile) {
    input {
      font-size: 35px;
      padding: 12px 8px;
    }
  }

  @include bp(desktop) {
    display: flex;
    align-items: flex-end;
    justify-content: space-between;

    input {
      text-align: right;
      width: 90%;
    }
  }
}

.converter__label {
  @include bp(0 mobile) {
    font-size: 12px;
  }

  font-size: 14px;
  font-weight: 500;
  color: $color__white;
}

.converter__data {
  width: 40%;

  &--source {
    margin-right: 15px;
  }

  &--result {
    margin-left: 15px;
  }

  @include bp(0 mobile) {
    &--result {
      align-self: flex-start;

      .converter__label {
        display: none;
      }
    }

    .converter__label {
      width: 100%;
      text-align: center;
      position: absolute;
    }
  }

  @include bp(mobile) {
    width: 45%;
  }

  @include bp(desktop) {
    flex: 1;
    max-width: 350px;
    margin-left: 0;
    margin-right: 0;
    text-align: right;
  }
}

.converter__replace-btn {
  position: relative;
  width: 30px;
  height: 30px;
  cursor: pointer;
  transition: background-color .3s;
  opacity: 1;
  border: none;
  border-radius: 100%;
  background: $color__blue;
  box-shadow: 0 3px 6px 0 rgba(0, 0, 0, .16);
  flex-shrink: 0;

  &:hover {
    background: #006C82;
  }

  svg {
    position: absolute;
    top: -8px;
    left: -10px;
    width: 50px;
    height: 50px;
  }

  @include bp(tablet) {
    width: 50px;
    height: 50px;

    svg {
      top: 2px;
      left: 1px;
    }
  }

  @include bp(desktop) {
    margin-top: 52px;
    margin-right: 40px;
    margin-left: 40px;
  }
}

.converter__currency {
  font-size: 20px;
  text-transform: uppercase;
  color: rgba($color__white, .3);
  padding-bottom: 15px;

  @include bp(0 desktop) {
    display: none;
  }
}

.converter__exchange-btn {
  display: block;
  margin: auto;
  padding: 13px 24px 13px;
  width: 130px;

  @include bp(mobile) {
    width: 210px;
    padding: 18px 60px 18px;
  }

}

.converter__select {
  width: 90px;

  @include bp(desktop) {
    margin-left: auto;
  }
}

</style>
