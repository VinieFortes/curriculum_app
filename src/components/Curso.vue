<template>
  <q-card bordered class="q-ma-sm">
    <q-card-section class="no-padding">
      <q-input
        v-model="nomeCurso"
        class="q-pa-sm"
        label="Nome do curso">
      </q-input>

      <q-select
        v-model="tipoCurso"
        :options="optionsTipoCurso"
        class="q-pa-sm"
        label="Tipo do Curso">
      </q-select>

      <q-input
        v-model="instituicao"
        class="q-pa-sm"
        label="Instituição">
      </q-input>

      <q-select
        v-model="sitacaoCurso"
        :options="optionsSituacaoCurso"
        class="q-pa-sm"
        label="Situação Atual">
      </q-select>

      <q-input
        v-model="conclusaoCurso"
        class="q-pa-sm"
        label="Ano/Mês de Conclusão">
      </q-input>

    </q-card-section>

    <q-card-actions align="right">
      <q-btn flat icon="delete" label="Remover"></q-btn>
      <q-btn flat icon="save" label="Salvar"></q-btn>
    </q-card-actions>

  </q-card>
</template>

<script>
export default {
  name: "Curso",

  data(){
    const nomeCurso = '';
    const tipoCurso = '';
    const instituicao = '';
    const sitacaoCurso = '';
    const conclusaoCurso = '';
    const optionsTipoCurso = [];
    const optionsSituacaoCurso = [];

    return{
      nomeCurso: nomeCurso,
      tipoCurso: tipoCurso,
      instituicao: instituicao,
      sitacaoCurso: sitacaoCurso,
      conclusaoCurso: conclusaoCurso,
      optionsSituacaoCurso: optionsSituacaoCurso,
      optionsTipoCurso: optionsTipoCurso
    }
  }
}
</script>

<style scoped>

</style>
