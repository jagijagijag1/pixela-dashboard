<template>
  <div class="pixela">
    <h2 class="title is-5">{{ graphName }}</h2>

    <div class="columns">
      <div
        class="column is-8">
        <div
          v-if="loaded"
          v-html="svg"/>
      </div>

      <div class="column">
        <div class="label">Input</div>

        <div class="field has-addons">
          <div class="control has-icons-left">
              <input 
                v-model="date" 
                type="number" 
                class="input"
                placeholder="Date">
            <span class="icon is-left">
              <i class="far fa-calendar-alt"></i>
            </span>
          </div>
          <div class="control">
            <DatePicker v-model="dateObj" :popover="{ placement: 'bottom-end', visibility: 'click' }">
              <button class="button">
                <i class="fas fa-calendar-check"></i>
              </button>
            </DatePicker>
          </div>
        </div>

        <div class="field has-addons">
          <div class="control has-icons-left">
            <input 
              v-model="quantityNum" 
              type="number" 
              class="input"
              placeholder="Quantity">
            <span class="icon is-left">
              <i class="fas fa-th"></i>
            </span>
          </div>
          <div class="control">
            <button class="button" @click="quantityNum = quantityNum ? quantityNum + 1 : 1">
              <i class="fas fa-plus"></i>
            </button>
          </div>
          <div class="control">
            <button class="button" @click="quantityNum = quantityNum ? quantityNum - 1 : -1">
              <i class="fas fa-minus"></i>
            </button>
          </div>
        </div>

        <div class="field">
          <div class="control">
            <button 
              :class="{ 'is-loading': isUpdating }"
              class="button is-link" 
              @click="updateData">Record / Update</button>
          </div>
        </div>

        <p 
          v-if="isSuccess"
          class="help is-success is-center">{{ msg }}</p>
        <p 
          v-if="!isSuccess" 
          class="help is-danger is-center">{{ msg }}</p>

      </div>

      <div class="column">
        <PixelaStreaks
          v-if="svg != ''"
          :svg= "svg" />
      </div>

    </div>

  </div>
</template>

<script>
import PixelaStreaks from './PixelaStreaks.vue'
import axios from 'axios';
import tippy from 'tippy.js';
import DatePicker from 'v-calendar/lib/components/date-picker.umd'

function formDate(dateObj) {
  const yyyy = dateObj.getFullYear()
  const MM = ('0' + (dateObj.getMonth() + 1)).slice(-2)
  const dd = ('0' + dateObj.getDate()).slice(-2)

  return yyyy + MM + dd
}

export default {
  components: {
    PixelaStreaks,
    DatePicker
  },
  props: {
    user: {
      type: String,
      required: true
    },
    token: {
      type: String,
      required: true
    },
    graphName: {
      type: String,
      required: true
    }
  },
  data() {
    return {
      loaded: true,
      graphUrl: '',
      date: '',
      dateObj: new Date(),
      quantityNum: null,
      msg: '',
      isSuccess: null,
      isUpdating: false,
      svg: ''
    }
  },
  computed: {
    quantity: function () {
      return this.quantityNum + ''
    }
  },
  watch: {
    dateObj: function (newDateObj) {
      this.date = formDate(newDateObj)
    }
  },
  created() {
    // set graph URL
    this.graphUrl = "https://pixe.la/v1/users/" + this.user + "/graphs/" + this.graphName

    // get svg content
    axios.get(this.graphUrl).then(response => {
      this.svg = response.data
    });
  },
  updated() {
    tippy('.each-day', { arrow: true });
  },
  methods: {
    updateData() {
      // set target URL
      const putUrl = this.graphUrl + '/' + this.date

      // initialize record/update-related variables
      this.loaded = false
      this.msg = ''
      this.isUpdating = true

      if (this.token === '' || this.date === '' | this.quantity === '') {
        this.updateForm('fail')
        this.msg = 'fill all input (USER-TOKEN, date, quantity)'
        return
      }

      // PUT (update) quantity of this.date to this. quantity on this.graphName
      axios({
        method: 'put',
        url: putUrl,
        headers: { 'X-USER-TOKEN': this.token },
        data: { quantity: this.quantity }
      }).then(response => {
        console.log("update " + response.data.message);
        this.updateForm('success')
      }).catch(error => {
        console.log(error);
        this.updateForm('fail')
      });
    },
    updateForm(isSuccess) {
      // update record/update-related variables
      this.loaded = true
      this.isUpdating = false

      if (isSuccess === 'success') {
        // if action sucessed
        this.isSuccess = true

        // update greaph svg
        axios.get(this.graphUrl).then(response => {
          this.svg = response.data
        });
        this.msg = 'Successfully recorded/updated'
      } else {
        // if action failed
        this.isSuccess = false
        this.msg = 'Error: something wrong'
      }
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.pixela {
  margin-bottom: 1%;
}

input::-webkit-outer-spin-button,
input::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}
</style>
