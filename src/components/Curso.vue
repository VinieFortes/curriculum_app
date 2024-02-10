<template>
  <q-card bordered style="border-radius: 16px">
    <q-form
      @submit="onSubmit"
      class="q-gutter-md">
        <q-card-section class="q-gutter-y-md q-pa-sm">

          <div class="flex column">
            <label for="nomeCurso">Nome do curso *</label>
            <input style="height: 35px; border-radius: 16px" type="text" v-model="cursoObj.nomeCurso" maxlength="50" required placeholder="Ex: Ciência da computação" />
          </div>

          <div class="flex column">
            <label for="tipoCurso">Tipo do Curso *</label>
            <select style="height: 35px" required v-model="cursoObj.tipoCurso">
              <option v-for="option in optionsTipoCurso" :value="option">{{option}}</option>
            </select>
          </div>

          <div class="flex column">
            <label for="instituicao">Instituição *</label>
            <input style="height: 35px; border-radius: 16px" type="text" v-model="cursoObj.instituicao" maxlength="50" required placeholder="Ex: Universidade Federal do Brasil" />
          </div>

          <div class="flex column">
            <label for="situacaoCurso">Situação Atual *</label>
            <select style="height: 35px" required v-model="cursoObj.situacaoCurso">
              <option v-for="option in optionsSituacaoCurso" :value="option">{{option}}</option>
            </select>
          </div>

          <div class="flex column">
            <label for="dataCurso">Mês/Ano de Conclusão *</label>
            <input style="height: 35px; border-radius: 16px" type="tel" v-model="cursoObj.dataCurso" @input="formatInput" required placeholder="Ex: 01/2040" />
          </div>

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
