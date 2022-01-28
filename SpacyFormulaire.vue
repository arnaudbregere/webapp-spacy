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
          <v-btn
            depressed
            color="primary"
            @click="loadImmList"
          >
            Analyse Figimmo
          </v-btn>
          <v-btn
            depressed
            color="primary"
            @click="analyseInBase"
          >
            Analyse dans Base
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
    <v-btn
      tyle="background: red"
      @click="getTextFromFigimmo"
    >
      Test Figimmo
    </v-btn>
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
      newProfile: [],
      listFingerPrint: []
    }
  },
  methods: {
    async loadImmList(){
      let configAxios = {
        method: 'get',
        url : 'https://fi-classified-search-api.immo.fcms.io/classifieds-with-size?transaction=vente&currentPage=1&pageSize=31&excludedPolePositionIDs=',
        headers: {
          'x-api-key':'AIzaSyDb4PeV9gi5UY_Z3-27ygjOm8PV950j9Us'
        }
      }
      const retListImmo = await axios(configAxios)
      if(retListImmo.status === 200){
        if(retListImmo.data.classifieds){
          retListImmo.data.classifieds.map(async value=>{
            const retItem = await this.loadImmItem(value.id)
            console.log('retItem status ', retItem.data.classified.descriptionFull.substr(0, 200))
            const retSpacyExtract = await this.doAnalyseItem(retItem.data.classified.descriptionFull)
            if(retSpacyExtract.status == 200){
              console.log('retSpacyExtract ', retSpacyExtract)
              this.calcEmpreinte(retSpacyExtract.data.words, retItem.data.classified.descriptionFull)
            }else{
              console.error('Erreur chargement spacy id ',  value.id , ' code status: ',  retSpacyExtract.status)
            }
          })
        }
      }
      console.log('retListImmo ', retListImmo)
    },
    async loadImmItem(itemID){
      let configAxios = {
        method: 'get',
        url : 'https://fi-classified-search-api.immo.fcms.io/web/classifieds/'+itemID,
        headers: {
          'x-api-key':'AIzaSyDb4PeV9gi5UY_Z3-27ygjOm8PV950j9Us'
        }
      }
      const retImmoItem = await axios(configAxios)
      return retImmoItem
    },
    async doAnalyseItem(item){
      let donnees = {
        text: item,
        model: "fr_core_news_sm"
      }
      const reponseSpacy =  await axios({
        method: 'post',
        url: 'http://localhost:8081/dep',
        data: JSON.stringify(donnees),
        headers: {
        },
        maxContentLength: 100000000,
        maxBodyLength: 1000000000
      }).catch(err => {
        console.error(err)
      })
      try{
        if(reponseSpacy.data.words){
          this.analyseSpacy = reponseSpacy.data.words
        let extractItemData =   this.extractItem(reponseSpacy.data.words)
          //this.analyseProfile()
          // this.calcEmpreinte(reponseSpacy.data.words, this.textAnalyse)
          await this.storeInDataBase(item, JSON.stringify(extractItemData))
        }
        if(reponseSpacy.data.arcs){
          this.listArcs = reponseSpacy.data.arcs
        }
      }catch(e){

      }
      return reponseSpacy
    },
    async doAnalyse(){
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
      //console.log('resultat requete : ', reponseSpacy)
      try{
        if(reponseSpacy.data.words){
          this.analyseSpacy = reponseSpacy.data.words
          this.extractItemSpacy()
          this.analyseProfile()
         // this.calcEmpreinte(reponseSpacy.data.words, this.textAnalyse)
          await this.storeInDataBase(this.textAnalyse, JSON.stringify(this.savedAnalysed))
        }
        if(reponseSpacy.data.arcs){
          this.listArcs = reponseSpacy.data.arcs
        }
      }catch(e){

      }
      return reponseSpacy

      //console.log('**********this.listWords************',this.listWords)
    },
    async storeInDataBase(textBrut, textFingerPrint) {
      // console.log('on est dans doAnalyse')
      let donnees = {
        text_base: textBrut,
        text_nlp_finger_print: textFingerPrint
      }
      const reponseDb =  await axios({
        method: 'post',
        url: 'http://localhost:8090/savetextnlp',
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
      try{
        console.log('*********reponseDb************ : ', reponseDb)
      }catch(e){
        console.log('*********e dbStore************ : ', e)
      }
    },
    async getTextFromFigimmo() {
      let data = {

      }
      const reponseFigimmo =  await axios({
        method: 'get',
        url: 'https://fi-classified-search-api0.staging.fcms.io/classifieds-with-size?transaction=vente&currentPage=1&pageSize=31&excludedPolePositionIDs=',
        data: JSON.stringify(data),
        headers: {
          'x-api-key': 'AIzaSyAcUotMkQ-pvaz4G6bMliRrjwJEmefQBcQ'
        },
        maxContentLength: 100000000,
        maxBodyLength: 1000000000
      }).catch(err => {
        console.error(err)
      })
      try{
        console.log('*********reponseDb************ : ', reponseFigimmo)
        if(reponseFigimmo.status === 200) {
          reponseFigimmo.data.classifieds.map(value => {
            console.log('////////////ID/////', value.id)
          })
        }
      }catch(e){
        console.log('*********e dbStore************ : ', e)
      }
    },
    async analyseInBase() {
     const listItem = await this.loadTextNlp(0, 10)
      if (listItem.status === 200) {
        listItem.data.map(value => {
          console.log('**********value********', value)
        })
      }
    },

    async loadTextNlp(start, limit) {
      let configAxios = {
        method: 'get',
        url : `http://localhost:8090/listtextnlp?start=${start}&limit=${limit}`,
        headers: {
         // 'x-api-key':'AIzaSyDb4PeV9gi5UY_Z3-27ygjOm8PV950j9Us'
        }
      }
      const axiosRequest = await axios(configAxios)

      return axiosRequest
    },

    extractItem(listItem) {
      let listItemNoun = []
      let listItemAdj = []
      let listItemPropn = []
      listItem.map(val => {
        if(val.tag === 'PROPN') { listItemPropn.push(val.text)}
        if(val.tag === 'NOUN') { listItemNoun.push(val.text)}
        if(val.tag === 'ADJ') { listItemAdj.push(val.text)}
      })
      return {propn: listItemPropn, noun : listItemNoun, adj: listItemAdj}
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
    },

    calcEmpreinte(listElementFromSpacy, textOrigin){
      var fingerPrint = []
      listElementFromSpacy.map(value=>{
     //   console.log('tag ', value.tag, ' valeur ', value.text)
        if(fingerPrint[value.tag] === undefined){
          fingerPrint[value.tag] = 1
        }else{
          fingerPrint[value.tag] ++
        }
      })
      console.log('resultat empreinte ', fingerPrint)
      var histoElement = {
        originText: textOrigin,
        fingerPrint: fingerPrint
      }
      this.listFingerPrint.push(histoElement)
      console.log('liste éléments historisés : ', this.listFingerPrint.length)
         this.listFingerPrint.map(value=>{
            let resultatFP = this.checkDiffEmpreinte(value.fingerPrint, histoElement.fingerPrint)
            console.log('resultatFP ', resultatFP)
          })
    },

    checkDiffEmpreinte(fp1, fp2) {
      let sommeFp1 = this.calcSomme(fp1)
      let sommeFp2 = this.calcSomme(fp2)

    },
    calcSomme(fp) {
      let listPoids = [['ADP', 3]]
      let listKey = Object.keys(fp)
      let somme = 0
      listKey.map(value => somme += fp[value])
      return somme
    }
  },
}
</script>

<style lang="scss">
.text-area {
  margin: 0 auto;
}
</style>
