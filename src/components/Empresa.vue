<template>
  <q-card bordered class="q-ma-sm">
    <q-form
      @submit="onSubmit"
      class="q-gutter-md">
      <q-card-section class="no-padding">
        <q-input
          v-model="empresaObj.nomeEmpresa"
          class="q-pa-sm"
          :rules="[ val => val.length > 0|| 'Esse campo é obrigatório !' ]"
          label="Nome da empresa">
        </q-input>

        <q-input
          v-model="empresaObj.cargo"
          class="q-pa-sm"
          :rules="[ val => val.length > 0|| 'Esse campo é obrigatório !' ]"
          label="Ultimo cargo">
        </q-input>

        <q-input
          v-model="empresaObj.descricao"
          class="q-pa-sm"
          type="textarea"
          :rules="[ val => val.length > 0|| 'Esse campo é obrigatório !' ]"
          label="Resumo profissional">
        </q-input>

        <q-input
          v-model="empresaObj.dataInicio"
          class="q-pa-sm"
          mask="##/####"
          :rules="[ val => val.length > 0|| 'Esse campo é obrigatório !' ]"
          label="Mês/Ano de inicio">
        </q-input>

        <q-input
          v-model="empresaObj.dataFim"
          class="q-pa-sm"
          mask="##/####"
          label="Mês/Ano de Término"
          bottom-slots>

          <template v-slot:counter>
            <span style="font-size: 16px">Se este for o seu emprego atual deixe o término em branco.</span>

          </template>
        </q-input>

      </q-card-section>
      <q-card-actions align="right">
        <q-btn @click="remove" flat icon="delete" label="Remover"></q-btn>
        <q-btn type="submit"  flat icon="save" label="Salvar"></q-btn>
      </q-card-actions>
    </q-form>
  </q-card>
</template>

<script>

const empresaObjModel = {nomeEmpresa: 'studioApp', cargo: '', descricao: '', dataInicio: '', dataFim: ''};

export default {
  name: "Empresa",
  props: {
    empresaObj: empresaObjModel
  },

  methods: {

    onSubmit () {
      this.salvar()
    },

    remove(){
      this.$emit('remover', this.empresaObj)
    },

    salvar(){
      this.$emit('salvar', true)
    }
  }
}
</script>

<style scoped>

</style>
