<template>
  <div id="ixwebsite">
    <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/font-awesome/4.5.0/css/font-awesome.min.css"/>

    <div id="IXAdServer">

      <div class="appContainer">

        <!-- DISPLAY PROJECT TITLE/LOGO -->
        <div class="titleContainer">
          <img src="https://cdn.admonsters.com/wp-content/uploads/2018/01/index-logo-Jan-2017-1024x306.png" class="IXLogo">
        </div>

        <!-- MAIN CONTAINER FOR FORM AND AD -->
        <div class="mainContainer">

          <!-- CONTAINER FOR FORM -->
          <div class="formContainer">
            <div class="formItem" v-for="item in questions" :key="item.tag">
              <div class="formItemTitle">
                {{item.title}}
              </div>
              <div v-if="item.type == 'radio'" class="radioButtonInputContainer" :style="returnRadioGridStyle(item.options.length)">
                <div v-for="option in item.options" :key="option">
                  <input v-on:click="doItemFunction(item)" v-model="item.value" class="radioButton" type="radio" :id="option" :name="option" :value="option">
                  <label class="radioButton" :for="option">{{option}}</label>
                </div>
              </div>
              <div v-if="item.type == 'dropdown'" class="dropdownContainer">
                <div class="dropDownHeaderContainer" v-on:click="item.shouldReveal = !item.shouldReveal">
                    <div class="dropdownItem dropdownHeader">
                      <div class="dropdownTitle">
                        <span v-if="item.value === ''">{{item.message}}</span>
                        <span v-else>{{item.value}}</span>
                      </div>
                      <div class="dropdownIcon">
                        <i v-if="!item.shouldReveal" class="fa fa-angle-down"></i>
                        <i v-else class="fa fa-angle-up"></i>
                      </div>
                    </div>
                    <div class="dropdownOptionsHolder" v-if="item.shouldReveal">
                      <div class="dropdownItem dropdownOption" v-on:click="selectItemValue(item, option)" v-for="option in item.options" :key="option">{{option}}</div>
                    </div>
                </div>
              </div>
              <div v-if="item.type == 'input'" class="inputContainer">
                <div class="inputHeaderContainer">
                  <input type="number" onkeyup="this.value=this.value.replace(/[^\d]/,'')" class="formTextInput" v-model="item.value">
                </div>
              </div>
            </div>
          </div>

          <!-- CONTAINER FOR AD -->
          <div class="adContainer">
            <div class="adDisplayHolder">
              <div class="adDisplay" :style="getAdContainerDimensions()">
                <div class="adPanel" :style="getAdPanelDimensions()">
                  <div id="borderHeader">
                      <input v-model="ipAddress" placeholder="IP ADDRESS HERE ..." class="ipAddressInput">
                  </div>
                </div>
                <div :style="getAdDimensions()">
                  <div class="noImageYet" v-if="imageSrc === ''">
                  </div>
                  <div class="imageFixed" v-else>
                    <img :src="imageSrc" class="adToDisplay">
                  </div>
                </div>
                <div class="adPanel" :style="getAdPanelDimensions()">
                  <div v-on:click="generateAd(questions)" id="borderFooter">
                    {{adCaption}}
                  </div>
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

import './ix-website.css';
import axios from 'axios';
axios.defaults.headers.post['Access-Control-Allow-Origin'] = '*';

export default {
  name: 'ixwebsite',
  data () {
    return {
      // Request Ad
      adCaption: 'GENERATE AD',
      ipAddress: '',
      port: '9000',
      endPoint: '',
      url: '',

      // Ad Display
      imageSrc: '',
      displayDimensions: {
        width: 250,
        height: 400,
        panelHeight: 45,
      },

      // Form Questions
      questions: [
        {
          title: 'Desired Shape',
          type: 'radio',
          options: [
            'Tall',
            'Wide',
          ],
          tag: 's',
          value: 'Tall',
          function: function(app) {
            if (this.value === 'Tall'){
              app.makeAdTall();
            } else if (this.value === 'Wide') {
              app.makeAdWide();
            }
          },
          isParam: true,
        },
        {
          title: "What do you like the most?",
          type: "dropdown",
          message: "Select one",
          options: [
            "Cooking",
            "Movies",
            "Sports",
            "Working",
          ],
          shouldReveal: false,
          tag: 'l',
          value: '',
          isParam: true,
          function: function(app) {
            for (let question of app.questions){
              if (question.tag === 'd'){
                question.options = JSON.parse(JSON.stringify(this.options));
                for (let [index, option] of question.options.entries()){
                  if (option === this.value){
                      question.options.splice(index, 1);
                      question.value = '';
                  }
                }
              }
            }
          },
        },
        {
          title: "What do you dislike the most?",
          type: "dropdown",
          message: "Select one",
          options: [
            "Cooking",
            "Movies",
            "Sports",
            "Working",
          ],
          shouldReveal: false,
          displayOptionConf: {
              compareItemTag: 'l',
          },
          tag: 'd',
          value: '',
          isParam: true,
        },
        {
          title: "Age",
          type: "input",
          message: "Select one",
          options: [
            "Young",
            "Old",
          ],
          tag: 'a',
          value: '',
          isParam: true,
        },
        {
          title: 'Which protocol are we using?',
          type: 'radio',
          options: [
            'IXRTB',
            'Other',
          ],
          tag: 'p',
          value: '',
          isParam: false,
          function: function(app) {
            app.endPoint = this.value;
          },
        },
        {
          title: 'Is this a trusted website?',
          type: 'radio',
          options: [
            'Yes',
            'No',
          ],
          tag: 'trust',
          value: '',
          isParam: true,
        },
      ],
    }
  },
  methods: {

		// Display Dimensions
		returnRadioGridStyle(optionsLength){
			let returnString = 'grid-template-columns:'
			for (let i = 0; i < optionsLength; i++){
				returnString = returnString + ' 1fr';
			}
			return returnString + ";";
		},
		makeAdTall(){
			this.displayDimensions.width = 250;
			this.displayDimensions.height = 400;
		},
		makeAdWide(){
			this.displayDimensions.width = 500;
			this.displayDimensions.height = 250;
		},
		getAdContainerDimensions(){
			let widthString = 'width: ' + this.displayDimensions.width + 'px;';
			let maxWidthString = 'max-width: ' + this.displayDimensions.width + 'px;';
			let heightStrng = 'height: ' + this.displayDimensions.height + 'px;';
			let maxHeightStrng = 'max-height: ' + this.displayDimensions.height + 'px;';
			return widthString + ' ' + heightStrng + ' ' + maxWidthString + ' ' + maxHeightStrng;
		},
		getAdPanelDimensions(){
			return 'height: ' + this.displayDimensions.panelHeight + 'px;';
		},
		getAdDimensions(){
			let height = this.displayDimensions.height - this.displayDimensions.panelHeight;
			return 'width: 100%; height: ' + height + 'px;';
		},

		// Handle Form Input Functions
		selectItemValue(item, option){
			item.value = option;
			if (item.function){
				item.function(this);
			}
		},
		doItemFunction(item){
			if (item.function){
				item.function(this);
			}
		},

		// Generate Ad
		generateAd(parameters){
			if (!this.formIsValidated()){
				return;
			}
			this.imageSrc = '';
			let requestURL = this.buildRequestString();
			axios.get(requestURL).then(response => {
        console.log(response);
        this.imageSrc = 'http://cdn.collider.com/wp-content/uploads/the-avengers-movie-poster-banners-04.jpg';
        // this.imageSrc = response.data.AdURL;
			}).catch(function (error) {
				alert("Error fetching ad: " + error);
			});
		},

		// Validation Methods
		formIsValidated(){
			for (let question of this.questions){
				if (question.value === ''){
					alert("Invalid answer for: " + question.title);
					return false;
				}
			}
			return this.validateIPaddress(this.ipAddress);
		},
		validateIPaddress(ipaddress) {
			let ipformat = /^(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$/;
			if(ipaddress.match(ipformat)) {
				return true;
			} else {
				alert("You have entered an invalid IP address!");
				return false;
			}
		},

		// Build Request URL
		buildRequestString(){
			let url = 'http://' + this.ipAddress + ':' + this.port + '/' + this.endPoint.toLowerCase() + '?';
			let firstDone = false;
			for (let question of this.questions){
				if (question.isParam){
					let param = '';
					if (firstDone){
						param = '&';
					} else {
						firstDone = true;
					}
					param = param + question.tag + '=' + question.value.toLowerCase();
					url = url + param;
				}
			}
			return url;
		},
  },
}
</script>