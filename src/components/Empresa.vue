<template>
  <q-card bordered style="border-radius: 16px">
    <q-form
      @submit="onSubmit"
      class="q-gutter-md">
      <q-card-section class="q-gutter-y-md q-pa-sm">

        <div class="flex column">
          <label for="nomeEmpresa">Nome da empresa *</label>
          <input style="height: 35px; border-radius: 16px"  type="text" v-model="empresaObj.nomeEmpresa" maxlength="50" required placeholder="Ex: Goooogle LTDA" />
        </div>

        <div class="flex column">
          <label for="cargo">Ultimo cargo *</label>
          <input style="height: 35px; border-radius: 16px"  type="text" v-model="empresaObj.cargo" maxlength="50" required placeholder="Ex: Diretor de projetos"/>
        </div>

        <div class="flex column">
          <label for="descricao">Resumo profissional *</label>
          <textarea  type="text" style="height: 150px;  border-radius: 16px"  v-model="empresaObj.descricao" maxlength="200" required  />
        </div>

        <div class="flex column">
          <label for="dataCurso">Mês/Ano de inicio *</label>
          <input style="height: 35px; border-radius: 16px"  type="tel" v-model="empresaObj.dataInicio" @input="formatInput" required placeholder="Ex: 01/1980" />
        </div>

        <div class="flex column">
          <label for="dataFim">Mês/Ano de Término</label>
          <input style="height: 35px; border-radius: 16px"  type="tel" v-model="empresaObj.dataFim" @input="formatInput2" placeholder="Ex: 01/2010" />

          <div style="background-color: rgba(84, 83, 95, 0.1); border-radius: 16px" class="q-mt-sm q-pa-sm text-primary">
            <span style="font-size: 16px">Se este for o seu emprego atual deixe o término em branco.</span>
          </div>
        </div>

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
