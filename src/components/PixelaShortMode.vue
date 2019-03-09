<template>
  <div
    class="pixela column is-narrow"
    style="width: 290px;">
    <article class="message">
      <div class="message-header has-background-grey">
        <p>{{ graphName }}</p>
      </div>

      <div class="message-body">
        <!--<p class="title is-5">{{ graphName }}</p>-->
        <div
          class="has-text-centered"
          style="margin-bottom: 15px;">
          <div
            v-if="loaded"
            v-html="svg"/>
        </div>

        <div
          class="columns is-vcentered is-marginless is-centered"
          style="padding-bottom: 5px;">
          <div class="column is-narrow is-paddingless">
            <label
              class="label is-small"
              style="width: 60px;">Data</label>
          </div>
          <div class="column is-narrow is-paddingless">
            <input 
              v-model="date" 
              type="text" 
              class="input is-small">
          </div>
        </div>

        <div
          class="columns is-vcentered is-marginless is-centered"
          style="padding-bottom: 5px;">
          <div class="column is-narrow is-paddingless">
            <label
              class="label is-small"
              style="width: 60px;">Quantity</label>
          </div>
          <div class="column is-narrow is-paddingless">
            <input 
              v-model="quantity" 
              type="text" 
              class="input is-small">
          </div>
        </div>

        <div class="columns is-centered is-marginless">
          <div class="column is-narrow has-text-centered">
            <button 
              :class="{ 'is-loading': isUpdating }"
              class="button is-link is-small" 
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
    </article>
  </div>
</template>

<script>
import axios from 'axios';
import tippy from 'tippy.js';

export default {
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
      quantity: '0',
      msg: '',
      isSuccess: null,
      isUpdating: false,
      shortModeSuffix: '?mode=short',
      svg: ''
    }
  },
  created() {
    // set graph URL
    this.graphUrl = "https://pixe.la/v1/users/" + this.user + "/graphs/" + this.graphName

    // set today's date string
    var dateObj = new Date()
    var yyyy = dateObj.getFullYear()
    var MM = ('0' + (dateObj.getMonth() + 1)).slice(-2)
    var dd = ('0' + dateObj.getDate()).slice(-2)
    this.date = yyyy + MM + dd

    // get svg content
    axios.get(this.graphUrl + this.shortModeSuffix).then(response => {
      this.svg = response.data
    });
  },
  updated() {
    tippy('.each-day', { arrow: true });
  },
  methods: {
    updateData() {
      // set target URL
      var putUrl = this.graphUrl + '/' + this.date
      this.initForm()

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
      })
      .then(response => {
        console.log("update " + response.data.message);
        this.updateForm('success')
      }).catch(error => {
        console.log(error);
        this.updateForm('fail')
      });
    },
    initForm() {
      // initialize record/update-related variables
      this.loaded = false
      this.msg = ''
      this.isUpdating = true
    },
    updateForm(isSuccess) {
      // update record/update-related variables
      this.loaded = true
      this.isUpdating = false

      if (isSuccess === 'success') {
        // if action sucessed
        this.isSuccess = true

        // update greaph svg
        axios.get(this.graphUrl + this.shortModeSuffix).then(response => {
          this.svg = response.data
        });
        this.msg = 'Successfully record/updated'
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
</style>
