<template>
  <div id="app">
    <!--<div style="border: 10px solid red; border-radius: 5px; min-height: 100px;">-->
      <!--<div class="jumbotron">-->
      <div class="container">
        <div class="top-header">
          <h1 class="text-center">Car Finder</h1>
          <p class="text-center" style="font-size: 1.5rem;">Get the latest recalls for your car</p>
        </div>
      </div>
    <!--</div>-->
    <!--</div>-->
    <div class="container">
      
    <div v-if="areRecallsReturned !== 'true'" v-bind:class="{ waiting: (loadingModels === 'true') }">
    <div class="row">
    <div class="col-sm-2 offset-sm-5">

        <!--load a dropdown with car makes-->
        <div>
            <label>Choose a make:</label>
            <br>
            <select v-model="selectedMake" v-on:change="getYearOptions()" style="min-width: 125px;">
                <option v-for="makeOption in makeOptions" v-bind:key="makeOption.make" v-bind:value="makeOption.make">
                    {{makeOption.make.charAt(0).toUpperCase() + makeOption.make.substr(1)}}
                </option>
            </select>
            <div v-if="getMakesError === 'true'" style="color: red;">Error getting makes</div>
            <br>
        </div>

        <div>
            <label>Choose a year:</label><br>
            <select v-model="selectedYear" v-on:change="getModelOptions()" v-bind:disabled="isMakeSelected !== 'true'" style="min-width: 125px;">
                <option v-for="yearOption in yearOptions" v-bind:key="yearOption.year" v-bind:value="yearOption.year">
                    {{ yearOption.year }}
                </option>
            </select>
            <div v-if="getYearsError === 'true'" style="color: red;">Error getting years</div>
            <br>
        </div>

        <div>
            <label>Choose a model:</label><br>
            <div v-if="loadingModels === 'true'" style="color: red;">Loading the models...please wait</div>
            <div v-if="noModelResults === 'true'" style="color: red;">No models found for make and year. Please select again.</div>
            <select v-model="selectedModel" v-on:change="setCarForRequest()" v-bind:disabled="isYearSelected !== 'true'" style="min-width: 125px;">
                <option v-for="modelOption in modelOptions" v-bind:key="modelOption.model" v-bind:value="modelOption.model">
                    {{ modelOption.model }}
                </option>
            </select>
            <div v-if="getModelsError === 'true'" style="color: red;">Error getting models</div>
            <br>
        </div>

        <div>
          <button v-on:click="getRecalls()" v-bind:disabled="isModelSelected !== 'true'" style="margin-top: 20px;">Get Recalls</button>
        </div>
        <div v-if="getRecallsError === 'true'" style="color: red;">Error getting recalls</div>
    </div>
    </div>
  </div>
    <div class="row">
      <div class="col-sm-8 offset-sm-2">
        <div v-if="areRecallsReturned === 'true'">
          <!--show message if no results found-->
          <div v-if="noRecallResults === 'true'"><h3>No results found</h3></div>
          <div>
              <button v-on:click="showSearch()">Search Again</button>
          </div>
          <!--show the recalls if results were found-->
          <div v-if="noRecallResults !== 'true'">
            <div class="text-center">
              <h2>Recalls for {{ selectedYear }} {{ selectedMake }} {{ selectedModel }}</h2>
              <img v-bind:src="imageUrl" v-bind:alt="selectedModel" width="80%" style="margin-bottom: 20px;" />
            </div>
                <div id="recall-list">
                    <div v-for="recall in recalls" v-bind:key="recall.reportReceivedDate">
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
        isMakeSelected: 'false',
        isYearSelected: 'false',
        areRecallsReturned: 'false',
        noModelResults: 'false',
        makeOptions: [],
        yearOptions: [],
        modelOptions: [],
        recalls: [],
        loadingModels: '',
        noRecallResults: 'false',
        getMakesError: 'false',
        getYearsError: 'false',
        getModelsError: 'false',
        getRecallsError: 'false'
      }
  },
  
  methods: {
        getMakeOptions() {
          axios.get("https://arrigucci-carfinder.azurewebsites.net/api/carrecall/makes")
          .then(
            response => {
              this.makeOptions = response.data
          })
          .catch(
              error => {
                //console.log(error);
                this.getMakesError = 'true';
          });
        },

        getYearOptions() {
            axios.get("https://arrigucci-carfinder.azurewebsites.net/api/carrecall/years")
            .then(
              response => {
                //reset the "no model results" message
                this.noModelResults = 'false';
                this.isMakeSelected = 'true';
                this.yearOptions = response.data
              })
            .catch(
              error => {
                //console.log(error);
                this.getYearsError = 'true';
              });
        },

        getModelOptions() {
            this.loadingModels = 'true';
            this.isYearSelected = 'true';
            axios.get("https://arrigucci-carfinder.azurewebsites.net/api/carrecall/" + this.selectedMake + "/" + this.selectedYear)
            .then(
              response => {
                this.modelOptions = response.data;
                this.loadingModels = '';
                //if no results found, show message
                if(response.data.length == 0){
                  this.noModelResults = 'true';
                }
              })
            .catch(
              error => {
                //console.log(error);
                //if error, turn off loading message and show error message
                this.loadingModels = 'false';
                this.getModelsError = 'true';
              });
        },

        getRecalls() {
            axios.get("https://arrigucci-carfinder.azurewebsites.net/api/carrecall/" + this.selectedMake + "/" + this.selectedYear + "/" + this.selectedModel)
            .then(
              response => {
                this.imageUrl = response.data.imageUrl;
                this.recalls = response.data.recallList;
                this.areRecallsReturned = 'true';
                if(response.data.recallList.length == 0){
                  this.noRecallResults = 'true';
                }
            })
            .catch(
              error => {
                //console.log(error);
                this.getRecallsError = 'true';
            });
        },
        setCarForRequest(){
            //model has been chosen, ready to search
            this.isModelSelected = 'true';
        },
        showSearch(){
          //reset to show search form again
          this.selectedMake = '';
          this.selectedYear = '';
          this.selectedModel = '';
          this.imageUrl =  '';
          this.areRecallsReturned =  'false';
          this.makeOptions =  [];
          this.yearOptions =  [];
          this.modelOptions =  [];
          this.recalls =  [];
          this.loadingModels =  '';
          this.isModelSelected = 'false';
          this.isYearSelected= 'false';
          this.isMakeSelected= 'false';
          this.noModelResults = 'false';
          this.noRecallResults = 'false';
          this.getMakesError = 'false';
          this.getYearsError = 'false';
          this.getModelsError = 'false';
          this.getRecallsError = 'false';

          //load makes into dropdown
          axios.get("https://arrigucci-carfinder.azurewebsites.net/api/carrecall/makes")
          .then(
            response => {
            this.makeOptions = response.data
          })
          .catch(
              error => {
                //console.log(error);
                this.getMakesError = 'true';
          });
        }
      },
      mounted() {
        axios.get("https://arrigucci-carfinder.azurewebsites.net/api/carrecall/makes").then(response => {
            this.makeOptions = response.data
          });
      }
}
</script>

<style>
  
</style>
