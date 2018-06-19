<template>
  <div class="container py-5">
    <div class="row">
      <div class="col-12 col-md-6">
        <form>
          <div class="form-group">
            <label for="betAmount">Bet amount</label>
            <input
              v-model="betData.amount"
              :max="betData.balance"
              type="number"
              class="form-control"
              id="betAmount"
              min="0.1"
              step="0.1"
              required>
            <small class="form-text text-muted">
              Enter number between 0.1 and {{ betData.balance }}
            </small>
          </div>
          <div class="form-group">
            <label for="number">Number</label>
            <input
              v-model="betData.number"
              type="number"
              class="form-control"
              id="number"
              min="1"
              max="100"
              required>
            <small class="form-text text-muted">
              Enter number from 1 to 100
            </small>
          </div>
          <div class="form-group form-check">
            <input
              v-model="autoBetEnabled"
              type="checkbox"
              class="form-check-input"
              id="autoBet">
            <label class="form-check-label" for="autoBet">Auto Bet</label>
          </div>
          <div class="form-group">
            <label for="numberOfBets">Number of Bets</label>
            <input
              v-model="betData.autoBetCount"
              type="number"
              class="form-control"
              id="numberOfBets"
              min="1"
              :disabled="!autoBetEnabled"
              :required="autoBetEnabled">
            <small class="form-text text-muted">
              Enter any number from 1
            </small>
          </div>
          <div class="form-group form-check">
            <input type="checkbox" class="form-check-input" id="martingale">
            <label class="form-check-label" for="martingale">Use Martingale Strategy</label>
          </div>
          <div class="d-flex justify-content-around">
            <div class="d-flex flex-column">
              <button
                name="betHi"
                type="submit"
                class="btn btn-success btn-lg"
                @click.prevent="placeBet({ bet: 'hi' })"
              >Bet Hi</button>
              <template v-if="betData.number">
                <small>number >= {{ betData.number }}</small>
                <small>chance: <strong>{{ chanceNumberWillBeHiger }}%</strong></small>
                <small>payout: <strong>{{ payoutOnHigher }}x</strong></small>
              </template>
            </div>
            <div class="d-flex flex-column ml-3">
              <button
                name="betLo"
                type="submit"
                class="btn btn-danger btn-lg"
                @click.prevent="placeBet({ bet: 'lo' })"
              >Bet Lo</button>
              <template v-if="betData.number">
                <small>number =&lt; {{ betData.number }}</small>
                <small>chance: <strong>{{ chanceNumberWillBeLower }}%</strong></small>
                <small>payout: <strong>{{ payoutOnLower }}x</strong></small>
              </template>
            </div>
          </div>
        </form>
      </div>
      <div class="col-12 col-md-6">
        <div class="d-flex flex-column justify-content-between h-100">
          <div class="text-center">
            <h2>Provably Fair Hash</h2>
            <p>{{ betData.hash }}</p>
            <template v-if="showResult">
              <p class="lead">Result</p>
              <h2>{{ betData.result }}</h2>
            </template>
          </div>
          <div>
            <h2>Balance: {{ betData.balance }}</h2>
            <button
              :disabled="betData.balance > 0"
              type="button"
              class="btn btn-info btn-lg"
              @click="addCredits"
            >Free Credits</button>
          </div>
        </div>
      </div>
    </div>
    <div class="row">
      <v-table :history="history" @clear="tableClearCallback"/>
    </div>
  </div>
</template>

<script>
import Vue from 'vue';
import md5 from 'md5';

import vTable from '@/components/blocks/table';

export default {
  name: 'game-form',
  data() {
    return {
      autoBetEnabled: false,
      betData: {
        amount: 0,
        autoBetCount: 0,
        balance: 100,
        hash: '',
        number: 0,
        result: 0,
      },
      history: [],
      showResult: false,
    };
  },
  components: {
    vTable,
  },
  methods: {
    addCredits() {
      this.betData.balance = 100;
    },
    tableClearCallback() {
      this.history = [];
    },
    generateNumber() {
      this.betData.result = Math.round(Math.random() * 100);
      this.betData.hash = md5(this.betData.result + Date.now());
    },
    placeBet({ bet }) {
      const betOnHigher = bet === 'hi';
      const isHigher = this.betData.number >= this.betData.result;

      switch (true) {
        // bet on higher and number is higher
        case (isHigher && betOnHigher):
          this.betData.balance += this.betData.amount * this.payoutOnHigher;
          break;
        // bet on lower but number is higher
        case (isHigher && !betOnHigher):
          this.betData.balance -= this.betData.amount * this.payoutOnLower;
          break;
        // bet on higher and number is lower
        case (!isHigher && betOnHigher):
          this.betData.balance -= this.betData.amount * this.payoutOnHigher;
          break;
        // bet on lower and number is lower
        case (!isHigher && !betOnHigher):
        default:
          this.betData.balance += this.betData.amount * this.payoutOnLower;
          break;
      }

      Vue.set(this.history, this.history.length, { bet, ...this.betData });
      this.saveUserBalance();
      this.generateNumber();
      this.showResult = true;
    },
    getUserBalance() {
      this.betData.balance = window.localStorage.getItem('balance') || 100;
    },
    saveUserBalance() {
      return window.localStorage.setItem('balance', String(this.betData.balance));
    },
  },
  computed: {
    chanceNumberWillBeHiger() {
      return 100 - this.betData.number;
    },
    chanceNumberWillBeLower() {
      return 100 - (100 - this.betData.number);
    },
    payoutOnHigher() {
      if (this.chanceNumberWillBeHiger === 0) {
        return 0;
      }
      return +((1 / this.chanceNumberWillBeHiger) * 100).toFixed(2);
    },
    payoutOnLower() {
      return +((1 / this.chanceNumberWillBeLower) * 100).toFixed(2);
    },
  },
  created() {
    this.getUserBalance();
    this.generateNumber();
  },
};
</script>

<style lang="scss" scoped>
  @import "@/assets/styles/_settings.scss";
  @import "~bootstrap/scss/forms";
</style>
