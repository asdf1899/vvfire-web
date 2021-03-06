<template>
  <md-app-content style="height:100%;border:none">
      <div class="md-layout md-alignment-center-center">
        <!-- Componente per aggiungere una nuova ricerca persona -->
        <nuovaRicercaDialog  
          v-if="this.newRicerca" 
          :active="this.newRicerca" 
          @ricercaDialogClosed="closeNewRicerca"
          @createNuovaRicerca="createNewRicerca"
        ></nuovaRicercaDialog>
        <!-- Componente per aggiornare il nome di una ricerca persona -->
        <updateRicercaDialog  
          v-if="this.updateDataDialog.active" 
          :data="this.updateDataDialog" 
          @updateRicercaDialogClosed="closeUpdateRicerca"
          @updateRicerca="updateRicerca"
        ></updateRicercaDialog>
        <!-- Componente per rimuovere il nome di una ricerca persona -->
        <deleteRicercaDialog  
          v-if="this.deleteDataDialog.active" 
          :data="this.deleteDataDialog" 
          @deleteRicerca="deleteRicerca"
        ></deleteRicercaDialog>
        <!-- Componente alert per completare la ricerca -->
        <completeRicercaDialog  
          v-if="this.completeRicercaDialog.active" 
          :data="this.completeRicercaDialog" 
          @completeRicerca="completeRicerca"
        ></completeRicercaDialog>
        <!-- Componente per eseguire alert specifici -->
        <Dialog v-if="this.message.active" :data="this.message"></Dialog>
        <md-card style="overflow-x:auto" v-if="!this.loading">
          <md-card-content>
            <div v-if="this.allRicerche.length !== 0 && !this.errored" style="overflow-x:auto">
              <md-table>
                <md-table-toolbar>
                  <h1 class="md-title">Lista ricerche</h1>
                </md-table-toolbar>
                <md-table-row>
                  <md-table-head class="style-table-header">NOME RICERCA</md-table-head>
                  <md-table-head class="style-table-header">INIZIO RICERCA</md-table-head>
                  <md-table-head class="style-table-header">FINE RICERCA</md-table-head>
                  <md-table-head class="style-table-header">STATO</md-table-head>
                  <md-table-head class="style-table-header"></md-table-head>
                  <md-table-head class="style-table-header"></md-table-head>
                  <md-table-head class="style-table-header"></md-table-head>
                </md-table-row>
                <md-table-row v-for="ricerca in this.allRicerche" :key="ricerca.id">
                  <md-table-head md-label="name">{{ricerca.name}}</md-table-head>
                  <md-table-head md-label="startTime">
                    {{new Date(ricerca.startTime).toLocaleString('it', { hour12: false, day: '2-digit', month: '2-digit', year: 'numeric', hour: '2-digit', minute:'2-digit' }).replace(/,/,'')}}
                  </md-table-head>
                  <md-table-head md-label="endTime">
                    {{ (ricerca.completed) ? new Date(ricerca.endTime).toLocaleString('it', { hour12: false, day: '2-digit', month: '2-digit', year: 'numeric', hour: '2-digit', minute:'2-digit' }).replace(/,/,'') : '-'}}
                  </md-table-head>
                  <md-table-head md-label="completed">{{ricerca.completed ? 'COMPLETATO' : 'IN CORSO...'}}</md-table-head>
                  <md-table-head md-label="mostra"><a @click="router.push({name:'Missioni', params: {idRicerca: ricerca.id}})">MOSTRA</a></md-table-head>
                  <md-table-head md-label="modifica"><a @click="openModificaRicerca(ricerca.id, ricerca.name)">MODIFICA</a></md-table-head>
                  <md-table-head md-label="completa" ><a v-bind:class="{'style-green-text': !ricerca.completed, 'style-disabled-link': ricerca.completed}" @click="(ricerca.completed) ? false : alertCompleteRicerca(ricerca.id)">COMPLETA</a></md-table-head>
                  <md-table-head md-label="elimina"><a class="style-red-text" @click="alertDeleteRicerca(ricerca.id)">RIMUOVI</a></md-table-head>
                </md-table-row>
              </md-table>
            </div>
            <div v-else-if="this.errored" style="text-align:center">
              <h3>Errore</h3>
            </div>
            <div v-else-if="!this.datiPresenti" style="text-align:center">
              <md-empty-state
                md-rounded
                md-icon="search_off"
                md-label=""
                md-description="Per creare una nuova ricerca persona, clicca il tasto rosso in basso a destra.">
              </md-empty-state>
            </div>
          </md-card-content>
        </md-card>
        <div v-else>
          <md-progress-spinner class="md-accent" md-mode="indeterminate"></md-progress-spinner>
        </div>
      </div>
    <md-button class="md-fab md-fab-bottom-right" @click="openNewRicerca()" :disabled="this.loading">
      <md-icon>add</md-icon>
    </md-button>
    <br>
  </md-app-content>
</template>

<script>
  import DialogAlert from '../components/Dialog.vue'; 
  import nuovaRicercaDialog from '../components/ricercapersona/nuovaRicerca.vue'; 
  import completeRicercaDialog from '../components/ricercapersona/completeRicercaDialog.vue';
  import deleteRicercaDialog from '../components/ricercapersona/deleteRicercaDialog.vue'; 
  import updateRicercaDialog from '../components/ricercapersona/updateRicercaDialog.vue'; 
  import loginController from '../controllers/loginController.js';
  import missioniController from '../controllers/missioniController.js';
  import ricercapersonaController from '../controllers/ricercapersonaController.js';
  import router from '../router/index.js';
  import moment from 'moment';

  export default {
    name: 'RicercaPersona',
    data: () => ({
      showNavigation: false,
      loading: false,
      message: {'active': false, 'content': null, 'url': null},
      deleteDataDialog: {'active': false, 'idRicerca': null},
      updateDataDialog: {'active': false, 'idRicerca': null, 'nameRicerca': null},
      completeRicercaDialog: {'active': false, 'idRicerca': null},
      datiPresenti: false,
      allRicerche: [],
      errored: false,
      newRicerca: false,
      router: router,
      moment: moment
    }),
    components: {
      'Dialog': DialogAlert,
      'nuovaRicercaDialog': nuovaRicercaDialog,
      'updateRicercaDialog': updateRicercaDialog,
      'deleteRicercaDialog': deleteRicercaDialog,
      'completeRicercaDialog': completeRicercaDialog
    },
    mounted(){
      this.fetchRicerche();
    },
    methods: {
      dialog(title, message, url){
        this.message.active = true;
        this.message.title = title;            
        this.message.content = message;
        this.message.url = url;         
      },
      openModificaRicerca(id, name){
        this.updateDataDialog.idRicerca = id;
        this.updateDataDialog.nameRicerca = name;
        this.updateDataDialog.active = true;
      },
      openNewRicerca(){
        // open new ricerca
        this.newRicerca = true;
      },
      closeNewRicerca(){
        this.newRicerca = false;
      },
      fetchRicerche(){
        this.loading = true;
        let idCorpo = loginController.getCorpoVVFData()['id'];
        ricercapersonaController.getRicercheByCorpo(idCorpo).then((response) => {
          let raw = response.data[0];
          this.allRicerche = [];
          if (!raw['error']){
            this.datiPresenti = true;
            this.allRicerche = raw.ricerche;
            this.newRicerca = false;
          }else{
            switch(raw['error']){
              case '401':
                this.dialog('Errore', 'Accesso non autorizzato, non puoi a visualizzare questa pagina', '#/dashboard');
                this.errored = true;
                break; 
              case '404':
                this.datiPresenti = false;
                this.newRicerca = false;
                break;
            }
          }
          this.loading = false;
        }, (error) => {
          this.errored= true;
          console.log(error)
          this.dialog('Errore', 'Errore, se il problema persiste contattare l\'amministratore', '#/dashboard');
          this.loading = false;
        });
        this.updateDataDialog.active = false;
      },
      createNewRicerca(value){
        this.loading = true;
        let name = value;
        let idCorpo = loginController.getCorpoVVFData()['id'];
        this.moment().locale('it');
        let startTime = this.moment().format();
        // endTime all'inizio viene impostato uguale a startTime, quando la ricerca viene completata, allora il valore
        // di endTime viene sovrascritto
        ricercapersonaController.newRicerca(name, startTime, startTime, false, idCorpo).then((response) => {
          let raw = response.data[0];
          if (!raw.error){
            this.fetchRicerche();
            this.loading = false;
          }else{
            this.errored= true;
            this.dialog('Errore', 'Errore, se il problema persiste contattare l\'amministratore', '#/ricercapersona');
          }
        }, (error) => {
          console.log(error);
          this.errored= true;
          this.dialog('Errore', 'Errore, se il problema persiste contattare l\'amministratore', '#/ricercapersona');
          this.loading = false;
        });
      },
      updateRicerca(updateData){
        this.loading = true;
        let id = updateData.id;
        let name = updateData.name;
        ricercapersonaController.updateRicerca(id, name).then((response) => {
          let raw = response.data[0];
          if (!raw.error){
            this.fetchRicerche();
          }else{
            this.errored= true;
            this.dialog('Errore', 'Errore, se il problema persiste contattare l\'amministratore', '#/ricercapersona');
          }
        }, (error) => {
          console.log(error);
          this.errored= true;
          this.dialog('Errore', 'Errore, se il problema persiste contattare l\'amministratore', '#/ricercapersona');
          this.loading = false;
        });
      },
      closeUpdateRicerca(){
        this.updateDataDialog.active = false;
      },
      alertDeleteRicerca(id){
        this.deleteDataDialog.active = true;
        this.deleteDataDialog.idRicerca = id;
      },
      alertCompleteRicerca(id){
        this.completeRicercaDialog.active = true;
        this.completeRicercaDialog.idRicerca = id;
      },
      completeRicerca(){
        this.loading = true;
        let id = this.completeRicercaDialog.idRicerca;
        ricercapersonaController.completeRicerca(id).then( (response) => {
          let raw = response.data[0];
          if (raw.error === false){
            this.fetchRicerche();
            this.completeAllMissioni(id);
            //this.completeRicercaDialog.idRicerca=null;
          }else{
            switch(raw[0]['error']){
              case '404':
                this.dialog('Errore', 'Il server non ha restituito i dati, contatta l\' amministratore', '#/ricercapersona');
                break;
            }
          }
        }, (error) => {
          console.log(error);
          this.dialog('Errore', 'Errore, se il problema persiste contattare l\'amministratore', '#/dashboard');
        });
        this.loading = false;
      },
      deleteRicerca(){
        let id = this.deleteDataDialog.idRicerca;
        ricercapersonaController.deleteRicerca(id).then((response) => {
          let raw = response.data;
          if (raw.error === false){
            this.fetchRicerche();
          }else if(raw.error === true){
            this.dialog('Errore', 'Impossibile cancellare i dati relativi alla ricerca persona, contattare l\'amministratore', '#/ricercapersona') 
          }else{
            switch(raw[0]['error']){
              case '401':
                this.dialog('Errore', 'Non sei autorizzato a cancellare i dati di questa ricerca persona, se credi ci sia stato un errore, contatta l\'amministratore', '#/ricercapersona');
                break;
              case '404':
                this.dialog('Errore', 'Il server non ha restituito i dati, contatta l\' amministratore', '#/ricercapersona');
                break;
            }
          }
        }, (error) => {
          console.log(error);
          this.dialog('Errore', 'Errore, se il problema persiste contattare l\'amministratore', '#/dashboard');
        });
      },
      completeAllMissioni(idRicerca){
        missioniController.getMissioniByRicerca(idRicerca).then((response) => {
          let raw = response.data[0];
          let allMissioni = [];
          if (!raw['error']){
            allMissioni = raw.missioni;
            for (let i=0;i<allMissioni.length;i++){
              missioniController.completeMissione(allMissioni[i].id);
            }
          }
        });
      }
    }
  }
</script>