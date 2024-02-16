<template>
  <q-card bordered style="border-radius: 16px">
    <q-form
      @submit="onSubmit"
      class="q-gutter-md">
      <q-card-section class="q-gutter-y-md q-pa-sm">

        <q-input
          v-model="empresaObj.nomeEmpresa"
          color="primary"
          bg-color="grey-2"
          rounded
          outlined
          maxlength="50"
          :rules="[ val => val.length > 0|| 'Esse campo é obrigatório !' ]"
          label="Nome da empresa *">
        </q-input>

        <q-input
          v-model="empresaObj.cargo"
          color="primary"
          bg-color="grey-2"
          rounded
          outlined
          maxlength="50"
          :rules="[ val => val.length > 0|| 'Esse campo é obrigatório !' ]"
          label="Último cargo *">
        </q-input>

        <q-input
          v-model="empresaObj.descricao"
          color="primary"
          bg-color="grey-2"
          rounded
          outlined
          type="textarea"
          autogrow
          maxlength="200"
          :rules="[ val => val.length > 0|| 'Esse campo é obrigatório !' ]"
          label="Resumo profissional *">
        </q-input>

        <q-input
          v-model="empresaObj.dataInicio"
          color="primary"
          bg-color="grey-2"
          rounded
          outlined
          type="tel"
          mask="##/####"
          :rules="[ val => val.length > 0|| 'Esse campo é obrigatório !' ]"
          label="Mês/Ano de inicio *">
        </q-input>

        <q-input
          v-model="empresaObj.dataFim"
          color="primary"
          bg-color="grey-2"
          rounded
          outlined
          type="tel"
          mask="##/####"
          label="Mês/Ano de término *"
          bottom-slots>

          <template v-slot:counter>
            <span style="font-size: 16px">Se este for o seu emprego atual deixe o término em branco.</span>
          </template>
        </q-input>

      </q-card-section>
      <q-card-actions class="flex justify-around items-center">
        <button style="height:30px;width:130px" class="text-negative" @click="remove" > <i class="fa fa-remove q-pr-sm"></i>Remover</button>
        <button style="height:40px;width:130px"  type="submit"  > <i class="fa fa-save q-pr-sm"></i>Salvar</button>
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

    formatInput() {
      let input = this.empresaObj.dataInicio;
      input = input.replace(/\D/g, '').slice(0, 6); // Remove non-digits and limit to 6 characters
      if (input.length >= 3) {
        input = input.slice(0, 2) + '/' + input.slice(2);
      }
      this.empresaObj.dataInicio = input;
    },

    formatInput2() {
      let input = this.empresaObj.dataFim;
      input = input.replace(/\D/g, '').slice(0, 6); // Remove non-digits and limit to 6 characters
      if (input.length >= 3) {
        input = input.slice(0, 2) + '/' + input.slice(2);
      }
      this.empresaObj.dataFim = input;
    },

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
