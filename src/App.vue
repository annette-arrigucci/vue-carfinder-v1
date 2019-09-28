<template>
  <div id="app">
    <div class="container">
            <h1 class="jumbotron text-center">Car Finder</h1>
    <div v-if="areRecallsReturned !== 'true'" v-bind:class="{ waiting: (loadingModels === 'true') }">
    <div class="row">
    <div class="col-sm-2 offset-sm-5">

        <!--load a dropdown with car makes-->
        <div>
            <label>Choose a make:</label>
            <br>
            <select v-model="selectedMake" v-on:change="getYearOptions()" style="min-width: 125px;">
                <option v-for="makeOption in makeOptions" v-bind:key="makeOption.make" v-bind:value="makeOption.make">
                    {{ makeOption.make }}
                </option>
            </select>
            <br>
        </div>

        <div>
            <label>Choose a year:</label><br>
            <select v-model="selectedYear" v-on:change="getModelOptions()" style="min-width: 125px;">
                <option v-for="yearOption in yearOptions" v-bind:key="yearOption.year" v-bind:value="yearOption.year">
                    {{ yearOption.year }}
                </option>
            </select><br>
        </div>

        <div>
            <label>Choose a model:</label>
            <div v-if="loadingModels === 'true'">Loading the models...please wait</div>
            <select v-model="selectedModel" v-on:change="setCarForRequest()" style="min-width: 125px;">
                <option v-for="modelOption in modelOptions" v-bind:key="modelOption.model" v-bind:value="modelOption.model">
                    {{ modelOption.model }}
                </option>
            </select><br>
        </div>

        <!--<div>{{ selectedYear }} {{ selectedMake }} {{ selectedModel }}</div>-->

        <div>
          <button v-on:click="getRecalls()" v-bind:disabled="isModelSelected !== 'true'">Get Recalls</button>
        </div>
    </div>
    </div>
  </div>
    <div class="row">
      <div class="col-sm-8 offset-sm-2">
        <div v-if="areRecallsReturned === 'true'">
          <div>
          <button v-on:click="showSearch()">Search Again</button>
        </div>
          <h2>Recalls for {{ selectedYear }} {{ selectedMake }} {{ selectedModel }}</h2>
            <img v-bind:src="imageUrl" v-bind:alt="selectedModel" width="600" height="400" />
            <div id="recall-list">
                <div v-for="recall in recalls" v-bind:key="recall.component">
                  <h3>Component: {{ recall.component }}</h3>
                    <ul>
                        <li class="text-left">Summary: {{ recall.summary }}</li>
                        <li class="text-left">Consequence: {{ recall.conequence }}</li>
                        <li class="text-left">Remedy: {{ recall.remedy }}</li>
                        <li class="text-left">Notes: {{recall.notes}}</li>
                    </ul>
                </div>
            </div>
        </div>
      </div>
    </div>
    </div>
  </div>
</template>

<script>

import axios from 'axios';

export default {
  
  name: 'app',
  data() {
    return {
        selectedMake: '',
        selectedYear: '',
        selectedModel: '',
        imageUrl: '',
        isModelSelected: 'false',
        areRecallsReturned: 'false',
        makeOptions: [],
        yearOptions: [],
        modelOptions: [],
        recalls: [],
        loadingModels: ''
      }
  },
  
  methods: {
        getMakeOptions() {
          axios.get("https://localhost:44390/api/carrecall/makes").then(response => {
            this.makeOptions = response.data
          });
        },

        getYearOptions() {
            axios.get("https://localhost:44390/api/carrecall/years").then(response => {
            this.yearOptions = response.data
          });
        },

        getModelOptions() {
            this.loadingModels = 'true';
            axios.get("https://localhost:44390/api/carrecall/" + this.selectedMake + "/" + this.selectedYear).then(response => {
            this.modelOptions = response.data;
            this.loadingModels = '';
          });
        },

        getRecalls() {
            axios.get("https://localhost:44390/api/carrecall/" + this.selectedMake + "/" + this.selectedYear + "/" + this.selectedModel).then(response => {
            this.imageUrl = response.data.imageUrl;
            this.recalls = response.data.recallList;
            this.areRecallsReturned = 'true';
          });
        },
        setCarForRequest(){
            //model has been chosen, ready to search
            this.isModelSelected = 'true';
        },
        showSearch(){
          //reset to show search form again
          this.selectedMake = '',
          this.selectedYear = '',
          this.selectedModel = '',
          this.imageUrl =  '',
          this.areRecallsReturned =  'false',
          this.makeOptions =  [],
          this.yearOptions =  [],
          this.modelOptions =  [],
          this.recalls =  [],
          this.loadingModels =  ''
          this.isModelSelected = 'false';

          //load makes into dropdown
          axios.get("https://localhost:44390/api/carrecall/makes").then(response => {
            this.makeOptions = response.data
          });
        }
      },
      mounted() {
        axios.get("https://localhost:44390/api/carrecall/makes").then(response => {
            this.makeOptions = response.data
          });
      }
}
</script>

<style>

</style>
