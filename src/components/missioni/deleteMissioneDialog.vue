<template>
  <div>
    <md-dialog-confirm
      :md-active.sync="data.active"
      md-title="Sei sicuro di voler eliminare la missione selezionata?"
      md-content="Verranno eliminati <strong>definitivamente</strong> tutti i dati (posizioni ecc.) relativi alla missione."
      md-confirm-text="CONTINUA"
      md-cancel-text="ANNULLA"
      @md-cancel="onCancel"
      @md-confirm="onConfirm" />
  </div>
</template>

<script>
  export default {
    name: 'deleteMissioneDialog',
    data: () => ({
      active: false
    }),
    props: {
      data: {
        type: Object
      }
    },
    methods: {
      onConfirm () {
        this.$emit('deleteMissione');
        this.active = false;
      },
      onCancel () {
        this.active = false;
      }
    }
  }
</script>

<style>
  .md-primary{
    color:white!important;
    z-index: 20!important;
    background-color: #e53935!important;
  }
</style>