<template>
  <main>
    <div id="topiosearch-wrapper">
      
      <header>
        <hgroup>
          <h1><a style="text-decoration:none; color:#000;" href="">TopioSearch</a></h1>
          <h2>
            <strong><a style="text-decoration:none; color:var(--muted-color)" href="">Outil de recherche suisse romand</a></strong>
            <br /><br />
            <small>Recherchez la définition d'un terme <br />ou l'origine d'un nom de lieu <br /> sur plusieurs lexiques simultanément.</small>
            <br />
            <small><small><em>Exemples: huitante, Lausanne, clédar, caïon, niolu...</em></small></small>
          </h2>
          
        </hgroup>
      </header>
      <br />
      <form id="topiosearch-form" @submit.prevent="initTopioSearch">
          
          <input v-model="topiosearch" type="search" id="topiosearch" name="topiosearch" required v-bind:placeholder="placeholder" />
          <button type="submit">Rechercher</button>
      
      </form>
    </div>
    <div id="topiosearch-loading" v-if="topioSearchLoading">
      <h6>Recherche sur : {{ topioSearchLoading }}...</h6>
    </div>
   
    <div id="topiosearch-results" v-if="topioSearchedTerm && topioSearchResults.length >= 1 && topioSearchResults[0].params">
        <h6>Résultats de la recherche pour: {{ topioSearchedTerm }}</h6>
        <article v-for="entry in topioSearchResults" v-bind:key="entry.url">
          <h3>{{ entry.params.term }} 
            <small v-if="entry.params.meta.type">
              <i>{{ entry.params.meta.type }}</i>
            </small>
          </h3>
          
          <p v-html="entry.params.definition"></p>

          <p v-if="entry.params.meta.alt && entry.params.meta.alt.length > 1">
            <small>
              Variantes: {{ entry.params.meta.alt.join(', ') }}
            </small>
          </p>

          <p>
            <small>
              <strong v-html="entry.params.copyright"></strong>
              <br />
              <span v-if="entry.params.source == 'remote'">Données tirées du site</span>
              <span v-else>Données en cache</span>
              &ndash;
              <a v-bind:href="entry.params.origin" target="_blank">Lien vers la page source</a>
            </small>
          </p>
        
        </article>
    </div>
    <div v-else-if="topioSearchedTerm && topioSearchResults.length == 0">
      <h6>
        Aucun résultat pour: {{ topioSearchedTerm }}<br />
        <small style="font-weight:normal;">Essayez avec une orthographe différente (avec ou sans les accents, majuscules, prépositions etc.)</small>
      </h6>
    </div>

    <!--<div id="topiosearch-results" v-if="topioSearchResults?.params">
      
        <h6>Résultats de la recherche pour: {{ topioSearchedTerm }}</h6>
        <article>
          <h3>{{ topioSearchResults?.params.term }}</h3>
          <p>
            <i v-if="topioSearchResults.params?.meta?.type">
              {{ topioSearchResults.params.meta.type }} &ndash;
            </i>
            <span v-html="topioSearchResults.params.definition"></span>
          </p>
        </article>
    </div>-->
    <footer>
      <p style="text-align:center; color:var(--muted-color)"><small><small>&copy; 2023 Lucius</small></small></p>
    </footer>
  </main>
</template>

<script>
export default {
  name: 'TopioForm',
  props: {
    placeholder: String
  },
  data(){ //two way data binding with v-model variables
    return{
      topiosearch:'',
      topioSearchedTerm:'',
      topioSearchLoading: '',
      topioSearchResults: []
    }
  },
  methods:{
    initTopioSearch(){
      this.topioSearchedTerm = '';
      this.topioSearchResults = [];
      

      //Fetch from henrysuter.ch words
      this.topioSearchLoading = 'henrysuter.ch (lexique)';
      fetch('https://api.oppidumweb.net/topiosearch/hsuter?term='+this.topiosearch, {
         headers : { 
        'Content-Type': 'application/json',
        'Accept': 'application/json'
         }
      })
        .then( response => response.json())
        .then(data => {
          
          if(data.status != "error"){
            this.topioSearchResults.push(data); //data is an Object, but topioSearchResults is a Proxy (can be used as an object).
          }

          
          fetch('https://api.oppidumweb.net/topiosearch/hsuternames?term='+this.topiosearch, {
              headers : { 
              'Content-Type': 'application/json',
              'Accept': 'application/json'
              }
            })
            .then( response => response.json() )
            .then(data => {
              console.log(data);
              

              if(data.status != "error"){
                this.topioSearchResults.push(data); //data is an Object, but topioSearchResults is a Proxy (can be used as an object).
              }


              //Fetch from topio.ch
              this.topioSearchLoading = 'topio.ch (lexique)';
              fetch('https://api.oppidumweb.net/topiosearch/topio?term='+this.topiosearch, {
                  headers : { 
                  'Content-Type': 'application/json',
                  'Accept': 'application/json'
                  }
                })
                .then( response => response.json())
                .then(data => {
                  console.log(data);
                  

                  if(data.status != "error"){
                    this.topioSearchResults.push(data); //data is an Object, but topioSearchResults is a Proxy (can be used as an object).
                  }

                  this.topioSearchLoading = '';
                  this.topioSearchedTerm = this.topiosearch;
            
              }); //endfetch topio.ch
          }); //endfetch hsuter names
        }); //endfetch hsuter
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>

/* Green Light scheme (Default) */
/* Can be forced with data-theme="light" */
[data-theme="light"],
:root:not([data-theme="dark"]) {
  --primary: #43a047;
  --primary-hover: #388e3c;
  --primary-focus: rgba(67, 160, 71, 0.125);
  --primary-inverse: #FFF;
}

/* Green Dark scheme (Auto) */
/* Automatically enabled if user has Dark mode enabled */
@media only screen and (prefers-color-scheme: dark) {
  :root:not([data-theme="light"]) {
    --primary: #43a047;
    --primary-hover: #4caf50;
    --primary-focus: rgba(67, 160, 71, 0.25);
    --primary-inverse: #FFF;
  }
}

/* Green Dark scheme (Forced) */
/* Enabled if forced with data-theme="dark" */
[data-theme="dark"] {
  --primary: #43a047;
  --primary-hover: #4caf50;
  --primary-focus: rgba(67, 160, 71, 0.25);
  --primary-inverse: #FFF;
}

/* Green (Common styles) */
:root {
  --form-element-active-border-color: var(--primary);
  --form-element-focus-color: var(--primary-focus);
  --switch-color: var(--primary-inverse);
  --switch-checked-background-color: var(--primary);
}


body>main{
  background-color:#fff;
  padding:24px;
}

article{
  margin:36px 0;
  padding: 24px;
}

@media (min-width: 1200px){

  body>main{
    max-width: 576px;
  }
}
@media (min-width: 992px){
  body>main{
    max-width: 576px;
  }
}
@media (min-width: 768px){
  body>main{
    max-width: 576px;
  }
}
@media (min-width: 576px){  
  html{
  background-color: var(--primary);
  }
  body{
    border:none;
  }
  body>main{

    border: 8px solid var(--primary-hover);
    margin-top:24px;
    max-width: 576px;
  }
}

h1,h2,#topiosearch-loading, #topiosearch-results h6{
  text-align:center;
}
#topiosearch-form button{
  width:200px;
  margin:auto;
}
</style>
