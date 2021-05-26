<template>
  <div>
    <h1>Currency Exchanger</h1>
    <div id="centered_items">
      <div id="label_container">
        <label for="source_currency">Source currency</label>
        <b-form-input list="drop_source" id="source_currency" @input="e => selectedSource = e"></b-form-input>
        <b-form-datalist id="drop_source" :options="values"></b-form-datalist>
      </div>
      <p>exchange to</p>
      <div id="label_container">
        <label for="target_currency">Target currency</label>
        <b-form-input list="drop_target" id="target_currency" @input="e => selectedTarget = e"></b-form-input>
        <b-form-datalist id="drop_target" :options="values"></b-form-datalist>
      </div>
    </div>
    <div id="contained_items">
    <b-input-group :prepend="prependValue" class="mt-3">
      <b-form-input v-model="actualValue" type="number"></b-form-input>
      <b-input-group-append>
        <b-button variant="outline-success" @click="exchangeValues">Exchange value</b-button>
      </b-input-group-append>
    </b-input-group>
    </div>
    <div v-if="currencyValue" id="exchange_result">
      <h3>{{currencyValue}}</h3>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
export default {
  name: 'App',
  data: function() {
    return {
      apiUrl: 'https://openexchangerates.org/api/',
      appid: 'cdede051190b47278de924cff6a125e5',
      selectedSource: '',
      selectedTarget: '',
      values: [],
      rates: [],
      actualValue: '',
      currencyValue: ''
    }
  },
  created: function() {
    axios.get(this.apiUrl + 'currencies.json').then(res => {
      for (const key in res.data) {
        this.values.push({text: key, value: key})
      }
    })
    axios.get(this.apiUrl + `latest.json?app_id=${this.appid}`).then(res => {
      for (const key in res.data.rates) {
        this.rates.push({name: key, value: res.data.rates[key]})
      }
    })
  },
  methods: {
    exchangeValues: function() {
      if (this.selectedSource.length === 3 && this.selectedTarget.length === 3 && this.selectedSource !== this.selectedTarget && this.actualValue) {
        const valSource = this.getValue(this.selectedSource)
        const valTarget = this.getValue(this.selectedTarget)
        this.currencyValue = `${this.actualValue} ${this.selectedSource} is equal to ${(parseFloat(this.actualValue) / valSource) * valTarget} ${this.selectedTarget}`
      } else if (this.selectedSource.length === 3 && this.selectedTarget.length === 3 && this.selectedSource === this.selectedTarget && this.actualValue) {
        this.currencyValue = `${this.actualValue} ${this.selectedSource} is equal to ${this.actualValue} ${this.selectedTarget}`
      } else {
        this.currencyValue = ''
      }
    },
    getValue: function(currency) {
      let result = null
      for (let i = 0; i < this.rates.length && !result; i++) {
        if (this.rates[i].name === currency) {
          result = this.rates[i].value
        }
      }
      return result
    }
  },
  computed: {
    prependValue: function() {
      let val = '$'
      if (this.selectedSource.length === 3 && this.selectedTarget.length === 3) {
        val = this.selectedSource + ' to ' + this.selectedTarget
      }
      return val
    }
  }
}
</script>

<style scoped>
h1 {
  text-align: center;
  color: #2c3e50;
  margin: 30px 0px 10px;
}

#label_container {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
}

label {
  margin: 8px 0 4px;
}

.form-control {
  width: auto;
}

#exchange_result {
  margin: 2% 0 0;
  display: flex;
  justify-content: center;
  align-items: center;
}

#contained_items {
  margin: 0 30%;
}

#centered_items {
  margin: 2% 0 0;
  display: flex;
  justify-content: space-evenly;
  align-items: center;
}

p {
  margin: 0 0;
}

#centered_items div {
  width: 150px;
}
</style>
