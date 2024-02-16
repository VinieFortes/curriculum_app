<template>
  <q-card bordered style="border-radius: 16px">
    <q-form
      @submit="onSubmit"
      class="q-gutter-md">
        <q-card-section class="q-gutter-y-md q-pa-sm">

          <q-input
            v-model="cursoObj.nomeCurso"
            color="primary"
            bg-color="grey-2"
            rounded
            outlined
            maxlength="50"
            :rules="[ val => val.length > 0|| 'Esse campo é obrigatório !' ]"
            label="Nome do curso *">
          </q-input>

          <q-select
            v-model="cursoObj.tipoCurso"
            :options="optionsTipoCurso"
            color="primary"
            bg-color="grey-2"
            rounded
            outlined
            :rules="[ val => val.length > 0|| 'Esse campo é obrigatório !' ]"
            label="Tipo do Curso *">
          </q-select>

          <q-input
            v-model="cursoObj.instituicao"
            color="primary"
            bg-color="grey-2"
            rounded
            outlined
            maxlength="50"
            :rules="[ val => val.length > 0|| 'Esse campo é obrigatório !' ]"
            label="Instituição *">
          </q-input>

          <q-select
            v-model="cursoObj.situacaoCurso"
            :options="optionsSituacaoCurso"
            color="primary"
            bg-color="grey-2"
            rounded
            outlined
            :rules="[ val => val.length > 0|| 'Esse campo é obrigatório !' ]"
            label="Situação Atual *">
          </q-select>

          <q-input
            v-model="cursoObj.dataCurso"
            color="primary"
            bg-color="grey-2"
            rounded
            outlined
            type="tel"
            mask="##/####"
            :rules="[ val => val.length > 0|| 'Esse campo é obrigatório !' ]"
            label="Mês/Ano de Conclusão *">
          </q-input>

        </q-card-section>
      <q-card-actions class="flex justify-around">
        <button style="height:30px;width:130px" class="text-negative" @click="remove" > <i class="fa fa-remove q-pr-sm"></i>Remover</button>
        <button style="height:40px;width:130px"  type="submit"  > <i class="fa fa-save q-pr-sm"></i>Salvar</button>
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

    formatInput() {
      let input = this.cursoObj.dataCurso;
      input = input.replace(/\D/g, '').slice(0, 6); // Remove non-digits and limit to 6 characters
      if (input.length >= 3) {
        input = input.slice(0, 2) + '/' + input.slice(2);
      }
      this.cursoObj.dataCurso = input;
    },

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
