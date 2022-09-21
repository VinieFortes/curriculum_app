<template>
  <q-card bordered class="q-ma-sm">
    <q-form
      @submit="onSubmit"
      class="q-gutter-md">
        <q-card-section class="no-padding">
            <q-input
              v-model="cursoObj.nomeCurso"
              class="q-pa-sm"
              :rules="[ val => val.length > 0|| 'Esse campo é obrigatório !' ]"
              label="Nome do curso">
            </q-input>

            <q-select
              v-model="cursoObj.tipoCurso"
              :options="optionsTipoCurso"
              class="q-pa-sm"
              :rules="[ val => val.length > 0|| 'Esse campo é obrigatório !' ]"
              label="Tipo do Curso">
            </q-select>

            <q-input
              v-model="cursoObj.instituicao"
              class="q-pa-sm"
              :rules="[ val => val.length > 0|| 'Esse campo é obrigatório !' ]"
              label="Instituição">
            </q-input>

            <q-select
              v-model="cursoObj.situacaoCurso"
              :options="optionsSituacaoCurso"
              class="q-pa-sm"
              :rules="[ val => val.length > 0|| 'Esse campo é obrigatório !' ]"
              label="Situação Atual">
            </q-select>

            <q-input
              v-model="cursoObj.dataCurso"
              class="q-pa-sm"
              mask="##/####"
              :rules="[ val => val.length > 0|| 'Esse campo é obrigatório !' ]"
              label="Mês/Ano de Conclusão">
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

const cursoObjModel = {nomeCurso: '', tipoCurso: '', instituicao: '', situacaoCurso: '', dataCurso: ''};

export default {
  name: "Curso",

  props: {
    cursoObj: cursoObjModel
  },

  methods: {

    onSubmit () {
      this.salvar()
    },

    remove(){
      this.$emit('remover', this.cursoObj)
    },

    salvar(){
      this.$emit('salvar', true)
    }
  },

  data(){
    const optionsTipoCurso = ['Formação Escolar Fundamental e Média', 'Ensino Médio Profissionalizante', 'Graduação', 'Pós-Graduação - Especialização', 'Pós-Graduação - MBA', 'Pós-Graduação - Mestrado', 'Pós-Graduação - Doutorado', 'Cursos Complementares'];
    const optionsSituacaoCurso = ['Interrompido', 'Cursando', 'Concluído'];

    return{
      optionsSituacaoCurso: optionsSituacaoCurso,
      optionsTipoCurso: optionsTipoCurso,
    }
  }
}
</script>

<style scoped>

</style>
