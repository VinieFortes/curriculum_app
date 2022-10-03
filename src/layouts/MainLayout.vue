<template>
  <q-layout>
    <q-header elevated>
      <q-toolbar>
        <q-btn
          flat
          dense
          round
          icon="menu"
          aria-label="Menu"
          @click="leftDrawerOpen = !leftDrawerOpen"
        />

        <q-toolbar-title>
          Currículo PDF App
        </q-toolbar-title>
      </q-toolbar>
    </q-header>

    <q-drawer
      v-model="leftDrawerOpen"
      show-if-above
      bordered
    >
      <q-list>

      </q-list>
    </q-drawer>

    <q-page-container>
      <q-form
        @submit="onSubmit"
      >
        <q-list>
          <q-expansion-item
            expand-separator
            icon="person"
            class="column"
            label="Dados pessoais">

              <q-input
              v-model="nome"
              class="q-pa-sm"
              :rules="[ val => val.length > 0|| 'Nome é obrigatório !' ]"
              label="Nome Completo">
              </q-input>

              <q-input
                v-model="dataNascimento"
                class="q-pa-sm"
                mask="##/##/####"
                label="Data de Nascimento">

                <template v-slot:append>
                  <q-icon name="event" class="cursor-pointer">
                    <q-popup-proxy ref="qDateProxy" transition-show="scale" transition-hide="scale">
                      <q-date mask="DD/MM/YYYY" v-model="dataNascimento" @input="() => $refs.qDateProxy.hide()" ></q-date>
                    </q-popup-proxy>
                  </q-icon>
                </template>
              </q-input>

              <q-select
                v-model="sexo"
                :options="optionsSexo"
                class="q-pa-sm"
                :rules="[ val => val.length > 0|| 'Esse campo é obrigatório !' ]"
                label="Sexo">
              </q-select>

              <q-select
              v-model="estadoCivil"
              :options="optionsEstadoCivil"
              class="q-pa-sm"
              :rules="[ val => val.length > 0|| 'Esse campo é obrigatório !' ]"
              label="Estado Civil">
              </q-select>

              <q-select
                v-model="paisNacionalidade"
                :options="optionsPaisNacionalidade"
                class="q-pa-sm"
                label="País de Nacionalidade">
                <template v-if="paisNacionalidade.flag" v-slot:prepend>
                  <span style="font-size: 17px">{{ this.iso2FlagEmoji(paisNacionalidade.flag)}}</span>
                </template>
              </q-select>

              <q-file
                v-model="file"
                label="Foto"
                class="q-pa-sm"
                accept=".jpg, .png, image/*"
                @update:model-value="handleUpload()"
              >
                <template v-if="!file" v-slot:prepend>
                  <q-icon name="photo_camera"></q-icon>
                </template>
                <template v-else v-slot:prepend>
                  <q-icon name="photo_camera"></q-icon>
                </template>
              </q-file>
                <q-img
                  :src="imageUrl"
                  spinner-color="white"
                  style="height: 140px; max-width: 150px"
                ></q-img>
          </q-expansion-item>

          <q-expansion-item
            expand-separator
            icon="contact_page"
            class="column"
            label="Informações de contato">

            <q-input
              v-model="email"
              class="q-pa-sm"
              :rules="[ val => val.length > 0|| 'Esse campo é obrigatório !' ]"
              label="E-mail">
            </q-input>

            <q-input
              v-model="telefone"
              class="q-pa-sm"
              :rules="[ val => val.length > 0|| 'Esse campo é obrigatório !' ]"
              label="Telefone">
            </q-input>

            <q-input
              v-model="celular"
              class="q-pa-sm"
              :rules="[ val => val.length > 0|| 'Esse campo é obrigatório !' ]"
              label="Celular">
            </q-input>

            <q-input
              v-model="cep"
              class="q-pa-sm"
              :rules="[ val => val.length > 0|| 'Esse campo é obrigatório !' ]"
              label="CEP">
            </q-input>

            <q-select
              v-model="paisLocal"
              :options="optionsPaisLocal"
              class="q-pa-sm"
              label="País"
              @update:model-value="selectState">
              <template v-if="paisLocal.value" v-slot:prepend>
                <span style="font-size: 17px">{{ this.iso2FlagEmoji(paisLocal.value)}}</span>
              </template>
            </q-select>

            <q-select
              v-model="estadoLocal"
              :options="optionsEstadoLocal"
              class="q-pa-sm"
              :disable="!paisLocal.value"
              label="Estado"
              @update:model-value="selectCity">
            </q-select>

            <q-select
              v-model="cidadeLocal"
              :options="optionsCidadeLocal"
              class="q-pa-sm"
              :disable="!estadoLocal.value"
              label="Cidade">
            </q-select>

            <q-input
              v-model="bairroLocal"
              class="q-pa-sm"
              label="Bairro">
            </q-input>

            <q-input
              v-model="endereco"
              class="q-pa-sm"
              label="Endereço">
            </q-input>

          </q-expansion-item>

          <q-expansion-item
            expand-separator
            icon="school"
            class="column"
            label="Formação">

            <q-select
              v-model="nivelEscolaridade"
              :options="optionsNivelEscolaridade"
              class="q-pa-sm"
              :rules="[ val => val.length > 0|| 'Esse campo é obrigatório !' ]"
              label="Nível de escolaridade">
            </q-select>

            <div v-if="cursosObjs.length === 0" class="flex q-pa-sm items-center justify-center bg-red-1">
              <span style="font-weight: bold">Nenhum curso adicionado !</span>
            </div>

            <q-list v-if="!showEditCursoModal.view || cursosObjs.length > 1" separator bordered class="q-ma-sm">
              <q-item v-show="showEditCursoModal.index !== index || showEditCursoModal.view === false" v-for="(curso, index) in cursosObjs" class="flex row justify-between">
                <div class="flex column">
                  <span style="font-weight: bold">{{ curso.tipoCurso }}</span>
                  <span>{{ curso.nomeCurso }}, {{ curso.instituicao }}</span>
                  <span>({{ curso.dataCurso }}), {{ curso.situacaoCurso }}</span>
                </div>
                <q-btn @click="editCurso(index)" :disable="showEditCursoModal.view" flat size="12px" label="Editar Curso" icon="edit"/>
              </q-item>
            </q-list>

            <Curso v-if="showEditCursoModal.view" :curso-obj="cursosObjs[showEditCursoModal.index]" @salvar="saveCurso" @remover="removeCurso"/>

            <div class="q-ma-md flex column items-center">
              <q-btn @click="addCurso" :disable="showEditCursoModal.view" icon="school" label="Adicionar Curso"></q-btn>
            </div>

          </q-expansion-item>

          <q-expansion-item
            expand-separator
            icon="engineering"
            class="column"
            label="Histórico Profissional">

            <div v-if="empresasObj.length === 0" class="flex q-pa-sm items-center justify-center bg-red-1">
              <span style="font-weight: bold">Nenhuma empresa adicionada !</span>
            </div>

            <q-list v-if="!showEditEmpresaModal.view || empresasObj.length > 1" separator bordered class="q-ma-sm">
              <q-item v-show="showEditEmpresaModal.index !== index || showEditEmpresaModal.view === false" v-for="(empresa, index) in empresasObj" class="flex row justify-between">
                <div class="flex column">
                  <span style="font-weight: bold">{{ empresa.cargo }}</span>
                  <span>{{ empresa.nomeEmpresa }}, {{ empresa.descricao }}</span>
                  <span>{{ empresa.dataInicio }} - {{ empresa.dataFim === '' ? 'Até o momento' : empresa.dataFim }}</span>
                </div>
                <q-btn @click="editEmpresa(index)" :disable="showEditEmpresaModal.view" flat size="12px" label="Editar Curso" icon="edit"/>
              </q-item>
            </q-list>

            <Empresa v-if="showEditEmpresaModal.view" :empresa-obj="empresasObj[showEditEmpresaModal.index]" @salvar="saveEmpresa" @remover="removeEmpresa"/>

            <div class="q-ma-md flex column items-center">
              <q-btn @click="addEmpresa" :disable="showEditEmpresaModal.view" icon="business" label="Adicionar Empresa"></q-btn>
            </div>

          </q-expansion-item>

        </q-list>

        <div class="flex column">
          <q-btn type="submit">Gerar PDF</q-btn>
          <button v-if="$q.platform.is.cordova" onclick="window.plugins.socialsharing.share('Here is your PDF file', 'Your PDF','file:///storage/emulated/0/Download/log.pdf','file:///storage/emulated/0/Download/log.pdf')">Share PDF</button>
          <q-btn v-if="$q.platform.is.cordova" @click="openPDF">Open PDF</q-btn>
        </div>
      </q-form>
    </q-page-container>
  </q-layout>
</template>

<script>
import { defineComponent, ref } from 'vue'
import jsPDF from "jspdf";
import listaPaises from "assets/countriesList.json"
const countries = require ("i18n-iso-countries");
import { Country, State, City }  from 'country-state-city';
import { Platform } from 'quasar'
import Curso from "components/Curso";
import Empresa from "components/Empresa";


export default defineComponent({
  name: 'MainLayout',

  components: {
    Empresa,
    Curso
  },

  methods: {

    editEmpresa(index){
      this.showEditEmpresaModal.view = true;
      this.showEditEmpresaModal.index = index;
    },

    removeEmpresa(args){
      this.empresasObj = this.empresasObj.filter(function (item){return item !== args})
      this.showEditEmpresaModal.view = false;
    },

    saveEmpresa(args){
      this.showEditEmpresaModal.view = false;
    },

    addEmpresa(){
      this.empresasObj.push({nomeEmpresa: 'studioApp', cargo: 'Desenvolvedor Front-End', descricao: 'Trabalho com Vue.js', dataInicio: '12/2021', dataFim: ''});
      this.showEditEmpresaModal.view = true;
      this.showEditEmpresaModal.index = this.empresasObj.length - 1;
    },

    removeCurso(args){
      this.cursosObjs = this.cursosObjs.filter(function (item){return item !== args})
      this.showEditCursoModal.view = false;
    },

    saveCurso(args){
      this.showEditCursoModal.view = false;
    },

    addCurso(){
      this.cursosObjs.push({nomeCurso: 'Ciência da Computação', tipoCurso: 'Graduação', instituicao: 'UFJF - Universidade Federal de Juiz de Fora', situacaoCurso: 'Cursando', dataCurso: '01/2024'});
      this.showEditCursoModal.view = true;
      this.showEditCursoModal.index = this.cursosObjs.length - 1;
    },

    editCurso(index){
      this.showEditCursoModal.view = true;
      this.showEditCursoModal.index = index;
    },

    onSubmit () {
      this.geratePDF();
    },

    selectCity(){
      City.getCitiesOfState(countries.alpha3ToAlpha2(this.paisLocal.value), this.estadoLocal.value).forEach(cidade =>{
        this.optionsCidadeLocal.push(cidade.name)
      })
    },

    selectState(){
      State.getStatesOfCountry(countries.alpha3ToAlpha2(this.paisLocal.value)).forEach(estado =>{
        this.optionsEstadoLocal.push({label: estado.name, value: estado.isoCode})
      })
    },

    openPDF(){
      cordova.plugins.fileOpener2.open(
        'file:///storage/emulated/0/Download/log.pdf',
        'application/pdf',
        {
          error : function(e) {
            console.log('Error status: ' + e.status + ' - Error message: ' + e.message);
          },
          success : function () {
            console.log('file opened successfully');
          }
        }
      );
    },

    calcularIdade(aniversario) {
      const nascimento = aniversario.split ("/");
      const dataNascimento = new Date (parseInt (nascimento[2], 10),
        parseInt (nascimento[1], 10) - 1,
        parseInt (nascimento[0], 10));

      const diferenca = Date.now () - dataNascimento.getTime ();
      const idade = new Date (diferenca);

      return Math.abs(idade.getUTCFullYear() - 1970);
    },

    handleUpload(){
      if (this.file) {
        this.imageUrl = URL.createObjectURL(this.file);
      }
    },
    iso2FlagEmoji(iso){
      if(!iso) return null;
      iso = countries.alpha3ToAlpha2(iso);
      return String.fromCodePoint(...[...iso.toUpperCase()].map(char => char.charCodeAt(0) + 127397))
    },

    getBase64(file) {
      return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.readAsDataURL(file);
        reader.onload = () => resolve(reader.result);
        reader.onerror = error => reject(error);
      });
    },

    async geratePDF() {
      const doc = new jsPDF ("p", "mm", "a4");

      this.$watch('lastY', (newY, oldY)=> {
        console.log(newY, oldY)
        if(newY >= 248){
          this.lastY = 22;
          doc.addPage('a4', 'p');
        }
      }, {immediate: true, flush: 'sync'})

      doc.setFont('helvetica', 'normal')
      if (this.nome.length > 30) {
        doc.setFontSize (20)
      } else {
        doc.setFontSize (25)
      }

      doc.text (this.nome, 20, this.lastY = 22);

      doc.setFontSize(12);

      if(this.sexo === 'Masculino'){
        this.estadoCivil = this.estadoCivil.replace('(a)', '')
      }
      else{
        this.estadoCivil = this.estadoCivil.replace('o(a)', 'a')
      }

      if(this.dataNascimento && this.estadoCivil){
        this.lastY = this.lastY + 8;
        doc.text(this.calcularIdade(this.dataNascimento).toString() + ' anos, ' + this.estadoCivil, 20, this.lastY)
      } else if (!this.dataNascimento && this.estadoCivil){
        this.lastY = this.lastY + 8
        doc.text(this.estadoCivil, 20, this.lastY)
      }

      doc.setFontSize(6)
      doc.setTextColor(210, 215, 211)
      this.lastY = this.lastY + 5;
      doc.text('__________________________________________________________________________________________________________________________________________________', 20, this.lastY, {maxWidth: 170})

      doc.setFontSize (23)
      doc.setTextColor(0, 0, 0)
      this.lastY = this.lastY + 15;
      doc.text('Dados pessoais', 20, this.lastY)

      doc.setFontSize(12);

      doc.setFont('helvetica', 'bold')
      this.lastY = this.lastY + 8;
      doc.text('E-mail: ', 22, this.lastY)
      doc.setFont('helvetica', 'normal')
      doc.text(this.email, 37, this.lastY)

      doc.setFont('helvetica', 'bold')
      this.lastY = this.lastY + 5;
      doc.text('País de Nacionalidade: ', 22, this.lastY)
      doc.setFont('helvetica', 'normal')
      doc.text(this.paisNacionalidade.label, 69, this.lastY)

      doc.setFont('helvetica', 'bold')
      this.lastY = this.lastY + 5;
      doc.text('Telefone: ', 22, this.lastY)
      doc.setFont('helvetica', 'normal')
      doc.text(this.telefone, 42, this.lastY)

      doc.setFont('helvetica', 'bold')
      this.lastY = this.lastY + 5;
      doc.text('Celular: ', 22, this.lastY)
      doc.setFont('helvetica', 'normal')
      doc.text(this.celular, 40, this.lastY)

      doc.setFont('helvetica', 'bold')
      this.lastY = this.lastY + 5;
      doc.text('Endereço: ', 22, this.lastY)
      doc.setFont('helvetica', 'normal')
      doc.text(`${this.endereco} ${this.bairroLocal} ${this.cep} ${this.cidadeLocal} ${this.estadoLocal.label ? this.estadoLocal.label : ''} ${this.paisLocal.label ? this.paisLocal.label : ''}`, 42, this.lastY,{maxWidth: 170})

      doc.setFontSize (23)
      doc.setTextColor(0, 0, 0)
      this.lastY = this.lastY + 15;
      doc.text('Formação', 20, this.lastY)

      doc.setFontSize(12);

      doc.setFont('helvetica', 'bold')
      this.lastY = this.lastY + 8;
      doc.text('Escolaridade: ', 22, this.lastY)
      doc.setFont('helvetica', 'normal')
      doc.text(this.nivelEscolaridade, 51, this.lastY)

      if(this.cursosObjs.length > 0){
        await this.cursosObjs.forEach((curso, index) =>{
          doc.setFont('helvetica', 'bold')
          this.lastY = this.lastY + 10;
          doc.text(curso.tipoCurso, 22, this.lastY);
          doc.setFont('helvetica', 'normal')
          this.lastY = this.lastY + 5;
          doc.text(`${curso.nomeCurso}, ${curso.instituicao}\n(${curso.dataCurso}) - ${curso.situacaoCurso}`, 22, this.lastY);
          this.lastY = this.lastY + 5;
        })
      }

      doc.setFontSize (23)
      doc.setTextColor(0, 0, 0)
      doc.text('Histórico profissional', 20,this.lastY = this.lastY + 20)

      doc.setFontSize(12);

      if(this.empresasObj.length > 0){
        await this.empresasObj.forEach((empresa, index) =>{
          doc.setFont('helvetica', 'bold')
          this.lastY = this.lastY + 10;
          doc.text(empresa.nomeEmpresa, 22, this.lastY);
          doc.setFont('helvetica', 'normal')
          this.lastY = this.lastY + 5;
          doc.text(`${empresa.cargo}, ${empresa.descricao}\n${empresa.dataInicio} - ${empresa.dataFim === '' ? 'Até o momento' : empresa.dataFim}`, 22, this.lastY);
          this.lastY = this.lastY + 5;
        })
      }


      if(this.file){
        await this.getBase64 (this.file).then (data => {
          doc.addImage (data.toString (), 'PNG', 15, 50, 50, 50)
        });
      }

      const pdfOutput = doc.output ("arraybuffer");

      if(!Platform.is.cordova){
        doc.save('teste.pdf');
      }else {
        window.resolveLocalFileSystemURL ('file:///storage/emulated/0/Download', successCallback, errorCallback)

        function successCallback(fs) {
          fs.getFile ('log.pdf', {create: true}, function (fileEntry) {

            fileEntry.createWriter (function (fileWriter) {
              fileWriter.onwriteend = function (e) {
                alert ('Write completed.' + fs.name);
              };

              fileWriter.onerror = function (e) {
                alert ('Write failed: ' + e.toString ());
              };
              fileWriter.write (pdfOutput);
            }, errorCallback);
          }, errorCallback);
        }

        function errorCallback(error) {
          alert ("ERROR: " + error.code)
        }
      }
    },
  },

  mounted() {
    listaPaises.forEach(pais =>{
      this.optionsPaisNacionalidade.push({label: pais.label, value: pais.label, flag: pais.code, dialCode: pais.dialCode});
      this.optionsPaisLocal.push({label: pais.label, value: pais.code})
    })

  },

  data(){
    const nome = 'Vinicius da Silva Fortes';
    const dataNascimento = '13/11/2000';
    const estadoCivil = 'Solteiro(a)';
    const sexo = 'Masculino'
    const optionsSexo = ['Masculino', 'Feminino'];
    const optionsEstadoCivil = ['Solteiro(a)', 'Casado(a)', 'Separado(a)', 'Divorciado(a)', 'Viúvo(a)'];
    const paisNacionalidade = {label: null, value: null, flag: null, dialCode: null};
    const optionsPaisNacionalidade = [];
    const email = 'viniciussilvafortes@gmail.com';
    const telefone = '32998033583';
    const celular = '32998033583';
    const cep = '36090290';
    const paisLocal = {label: null, value: null};
    const optionsPaisLocal = [{label: null, value: null}]
    const estadoLocal = {label: null, value: null};
    const optionsEstadoLocal = [{label: null, value: null}];
    const cidadeLocal = '';
    const optionsCidadeLocal = [];
    const bairroLocal = '';
    const endereco = '';
    const file = null;
    const leftDrawerOpen = null;
    const toggleLeftDrawer = null;
    const showEditCursoModal = {view: false, index: null};
    const showEditEmpresaModal = {view: false, index: null};
    const nivelEscolaridade = 'Formação Superior Incompleta';
    const optionsNivelEscolaridade = ['Ensino Fundamental Incompleto', 'Ensino Fundamental Completo', 'Ensino Médio Incompleto', 'Ensino Médio Completo', 'Formação Superior Incompleta', 'Formação Superior Completa', 'Pós-graduação no nível Especialização', 'Pós-graduação no nível Mestrado', 'Pós-graduação no nível Doutorado'];
    let cursosObjs = [];
    let empresasObj = [];
    let imageUrl = '';
    return{
      nome: nome,
      dataNascimento: dataNascimento,
      estadoCivil: estadoCivil,
      sexo: sexo,
      optionsSexo: optionsSexo,
      optionsEstadoCivil: optionsEstadoCivil,
      paisNacionalidade: paisNacionalidade,
      optionsPaisNacionalidade: optionsPaisNacionalidade,
      email: email,
      telefone: telefone,
      celular: celular,
      cep: cep,
      paisLocal: paisLocal,
      optionsPaisLocal: optionsPaisLocal,
      estadoLocal: estadoLocal,
      optionsEstadoLocal: optionsEstadoLocal,
      cidadeLocal: cidadeLocal,
      optionsCidadeLocal: optionsCidadeLocal,
      bairroLocal: bairroLocal,
      endereco: endereco,
      nivelEscolaridade: nivelEscolaridade,
      optionsNivelEscolaridade: optionsNivelEscolaridade,
      cursosObjs: cursosObjs,
      empresasObj: empresasObj,
      file: file,
      showEditCursoModal: showEditCursoModal,
      showEditEmpresaModal: showEditEmpresaModal,
      leftDrawerOpen: leftDrawerOpen,
      toggleLeftDrawer: toggleLeftDrawer,
      imageUrl: imageUrl,
      lastY: 0
    }
  }

})
</script>
