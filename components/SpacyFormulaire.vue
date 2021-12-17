<template>
  <div>
    <v-card
      class="mx-auto"
      max-width="344"
      outlined
    >
      <v-list-item three-line>
        <v-list-item-content>
          <div class="text-overline mb-4">
            OVERLINE
          </div>
          <v-list-item-title class="text-h5 mb-1">
            Headline 5
          </v-list-item-title>
          <v-list-item-subtitle>Greyhound divisely hello coldly fonwderfully</v-list-item-subtitle>
        </v-list-item-content>

        <v-list-item-avatar
          tile
          size="80"
          color="grey"
        ></v-list-item-avatar>
      </v-list-item>

      <v-card-actions>
        <v-btn
          outlined
          rounded
          text
        >
          Button
        </v-btn>
      </v-card-actions>
    </v-card>
    <v-container fluid>
      <v-row>
        <v-col
          cols="12"
          md="12"
        >
          <v-textarea
            class="text-area"
            name="input-7-1"
            label="Texte à analyser par Spacy"
            v-model="textAnalyse"
          ></v-textarea>
          <v-btn
            depressed
            color="primary"
            @click="doAnalyse"
          >
            Analyser le texte
          </v-btn>
        </v-col>
      </v-row>
    </v-container>
    <v-container>
      <v-data-table
        :headers="headers"
        :items="analyseSpacy"
        :items-per-page="15"
        item-key="name"
        class="elevation-1"
        :footer-props="{
      showFirstLastPage: true,
      firstIcon: 'mdi-arrow-collapse-left',
      lastIcon: 'mdi-arrow-collapse-right',
      prevIcon: 'mdi-minus',
      nextIcon: 'mdi-plus'
    }"
      ></v-data-table>
    </v-container>
    <v-container>
      <v-data-table
        :headers="headers"
        :items="savedAnalysed"
        :items-per-page="15"
        item-key="name"
        class="elevation-1"
        :footer-props="{
      showFirstLastPage: true,
      firstIcon: 'mdi-arrow-collapse-left',
      lastIcon: 'mdi-arrow-collapse-right',
      prevIcon: 'mdi-minus',
      nextIcon: 'mdi-plus'
    }"
      ></v-data-table>
    </v-container>

    <v-container>
      <p>Noun : {{newProfile.noun}}</p>
      <p>Propn : {{newProfile.propn}}</p>
      <p>Verb : {{newProfile.verb}}</p>
      <p>Adj : {{newProfile.adj}}</p>
      <p>Num : {{newProfile.num}}</p>
    </v-container>
  </div>
</template>

<script>
import axios from "axios";
class ProfileText {
  noun
  propn
  verb
  adj
  num

  constructor(noun, propn, verb, adj, num) {
    this.noun = noun
    this.propn = propn
    this.verb = verb
    this.adj = adj
    this.num = num
  }
}
export default {
  name: "spacyformulaire",
  data() {
    return {
      textAnalyse : '',
      analyseSpacy: [],
      headers: [
        {
          text: 'TAG',
          align: 'start',
          value: 'tag',
        },
        {
          text: 'TEXTE',
          value: 'text'
        },
      ],
      listArcs: [],
      listWords : [],
      savedTags : [ 'PROPN', 'VERB', 'NUM', 'NOUN', 'ADJ'],
      savedAnalysed : [],
      newProfile: []
    }
  },
  methods: {
    async doAnalyse(){
      console.log('on est dans doAnalyse')
      let donnees = {
        text: this.textAnalyse,
        model: "fr_core_news_sm"
      }
      const reponseSpacy =  await axios({
        method: 'post',
        url: 'http://localhost:8081/dep',
        data: JSON.stringify(donnees),
        headers: {
          // 'Accept': 'application/json',
          // 'Access-Control-Allow-Origin': '*',
          // 'Content-Type': 'application/json;charset=utf-8'
        },
        maxContentLength: 100000000,
        maxBodyLength: 1000000000
      }).catch(err => {
        console.error(err)
      })
      console.log('resultat requete : ', reponseSpacy)
      try{
        if(reponseSpacy.data.words){
          this.analyseSpacy = reponseSpacy.data.words
          this.extractItemSpacy()
          this.analyseProfile()
        }
        if(reponseSpacy.data.arcs){
          this.listArcs = reponseSpacy.data.arcs
        }
      }catch(e){

      }

      console.log('**********this.listWords************',this.listWords)
    },
    extractItemSpacy() {
      this.savedAnalysed = []
      this.analyseSpacy.map(val => {
        this.savedTags.map(val2 => {
          if(val.tag === val2) {
            this.savedAnalysed.push(val)
          }
        })
      })
    },

    analyseProfile() {
      let noun = 0
      let propn = 0
      let verb = 0
      let adj = 0
      let num = 0
      this.analyseSpacy.map(val => {
        switch (val.tag) {
          case 'NOUN' : noun++; break;
          case 'PROPN' : propn++; break;
          case 'VERB' : verb++; break;
          case 'ADJ' : adj++; break;
          case 'NUM' : num++; break;
        }
      })
      this.newProfile = new ProfileText(noun, propn, verb, adj, num)
    }

/*    async doAnalyse() {
      console.log('on est dans doAnalyse')
      let donnees = {
        text: this.textAnalyse,
        model: "fr_core_news_sm"
      }
      const reponseSpacy =  await axios({
        method: 'post',
        url: 'http://localhost:8080/dep',
        data: JSON.stringify(donnees),
        headers: {
          // 'Accept': 'application/json',
          // 'Access-Control-Allow-Origin': '*',
          // 'Content-Type': 'application/json;charset=utf-8'
        },
        maxContentLength: 100000000,
        maxBodyLength: 1000000000
      }).catch(err => {
        console.error(err)
      })
      console.log('resultat requete : ', reponseSpacy)
      try{
        if(reponseSpacy.data.words){
          this.listWords = reponseSpacy.data.words
        }
        if(reponseSpacy.data.arcs){
          this.listArcs = reponseSpacy.data.arcs
        }
      }catch(e){

      }
    }*/
  },
}
</script>

<style lang="scss">
.text-area {
  margin: 0 auto;
}
</style>
