<template>
  <q-layout>
    <q-header elevated class="rounded-borders">
      <q-toolbar class="bg-green rounded-borders">
        <q-btn
          flat
          dense
          round
          icon="menu"
          aria-label="Menu"
          @click="leftDrawerOpen = !leftDrawerOpen"
        />

        <q-toolbar-title>
          Gerador de Currículo PDF
        </q-toolbar-title>
      </q-toolbar>
    </q-header>
    <q-drawer
      v-model="leftDrawerOpen"
      show-if-above
      bordered
    >
      <q-list>
        <q-item class="flex column">
          <q-item-label class="text-bold text-green">
            Preenchimento do currículo
          </q-item-label>
          <q-linear-progress dark stripe rounded :value="step / 6" size="20px"  color="green" class="q-mt-sm" />
          <q-item-label class="q-mt-sm text-green">O QUE FALTA ?</q-item-label>

          <q-expansion-item v-for="item in itensMenu" class="q-mt-sm" expand-separator :header-class="item.fields.every(val => val.status === true) ? 'text-green' : 'text-negative'" :icon="item.icon" :label="item.label" >
            <div class="q-ml-lg" v-for="field in item.fields">
              <q-icon :color="field.status ? 'positive' : 'negative'" :name="field.status ? 'check' : 'clear'"/>
              <span :class="field.status ? 'text-green' : 'text-negative'">{{field.label}}</span>
            </div>
          </q-expansion-item>
        </q-item>
      </q-list>
    </q-drawer>

    <q-page-container>
      <q-stepper
        v-model="step"
        vertical
        :header-nav="step === 7"
        bordered
        color="green"
        animated>

        <span v-if="step === 7" class="note flex flex-center bg-light-green-1 q-pa-sm text-green text-bold q-ma-sm">Caso precise fazer alguma mudança no seu currículo apenas clique em dos estagios abaixo.</span>

        <q-step
          :name="1"
          title="Dados pessoais"
          icon="person"
          active-color="primary"
          done-color="green"
          :done="step > 1"
        >
          <q-form
            @submit="nextStep">
            <q-input
              v-model="nome"
              class="q-pa-sm"
              maxlength="80"
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

            <q-stepper-navigation>
              <q-btn type="submit" color="green" label="Continuar" />
            </q-stepper-navigation>
          </q-form>

        </q-step>

        <q-step
          :name="2"
          title="Informações de Contato"
          active-color="primary"
          done-color="green"
          icon="contact_page"
          :done="step > 2"
        >
          <q-form
            @submit="nextStep">
            <q-input
              v-model="email"
              class="q-pa-sm"
              :rules="[val => !!val || 'Campo obrigatório',
                        val => validEmail(val) || 'E-mail inválido']"
              label="E-mail">
            </q-input>

            <q-input
              v-model="telefone"
              class="q-pa-sm"
              label="Telefone">
            </q-input>

            <q-input
              v-model="celular"
              class="q-pa-sm"
              mask="(##) #####-####"
              :rules="[ val => val.length > 0|| 'Esse campo é obrigatório !' ]"
              label="Celular">
            </q-input>

            <q-input
              v-model="cep"
              class="q-pa-sm"
              mask="#####-###"
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
              class="q-pa-sm"
              use-input
              input-debounce="0"
              :options="filtroCidadeOptions"
              @filter="filterCidade"
              :disable="!estadoLocal.value"
              label="Cidade">
            </q-select>

            <q-input
              v-model="bairroLocal"
              class="q-pa-sm"
              maxlength="50"
              label="Bairro">
            </q-input>

            <q-input
              v-model="endereco"
              class="q-pa-sm"
              maxlength="100"
              label="Endereço">
            </q-input>

            <q-stepper-navigation>
              <q-btn type="submit" color="green" label="Continuar" />
              <q-btn flat @click="step = 1" color="green" label="Voltar" class="q-ml-sm" />
            </q-stepper-navigation>
          </q-form>
        </q-step>

        <q-step
          :name="3"
          title="Formação"
          icon="school"
          active-color="primary"
          done-color="green"
          :done="step > 3"
        >
          <q-form
            @submit="nextStep">
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
              <q-item v-show="showEditCursoModal.index !== index || showEditCursoModal.view === false" v-for="(curso, index) in cursosObjs" class="flex row">
                <div style="flex: 1" class="flex column">
                  <span style="font-weight: bold">{{ curso.tipoCurso }}</span>
                  <span>{{ curso.nomeCurso }}, {{ curso.instituicao }}</span>
                  <span>({{ curso.dataCurso }}), {{ curso.situacaoCurso }}</span>
                </div>
                <q-btn v-if="$q.platform.is.cordova" style="flex: 0" @click="editCurso(index)" :disable="showEditCursoModal.view" flat size="10px" icon="edit"/>
                <q-btn v-else @click="editCurso(index)" style="flex: 0" :disable="showEditCursoModal.view" flat size="12px" label="Editar Curso" icon="edit"/>
              </q-item>
            </q-list>

            <Curso v-if="showEditCursoModal.view" :curso-obj="cursosObjs[showEditCursoModal.index]" @salvar="saveCurso" @remover="removeCurso"/>

            <div class="q-ma-md flex column items-center">
              <q-btn @click="addCurso" :disable="showEditCursoModal.view" icon="school" label="Adicionar Curso"></q-btn>
            </div>

            <q-stepper-navigation>
              <q-btn type="submit" color="green" label="Continuar" />
              <q-btn flat @click="step = 2" color="green" label="Voltar" class="q-ml-sm" />
            </q-stepper-navigation>
          </q-form>

        </q-step>

        <q-step
          :name="4"
          title="Histórico Profissional"
          icon="engineering"
          active-color="primary"
          done-color="green"
          :done="step > 4"
        >
          <q-form
            @submit="nextStep">
            <div v-if="empresasObj.length === 0" class="flex q-pa-sm items-center justify-center bg-red-1">
              <span style="font-weight: bold">Nenhuma empresa adicionada !</span>
            </div>

            <q-list v-if="!showEditEmpresaModal.view || empresasObj.length > 1" separator bordered class="q-ma-sm">
              <q-item v-show="showEditEmpresaModal.index !== index || showEditEmpresaModal.view === false" v-for="(empresa, index) in empresasObj" class="flex row">
                <div style="flex: 1" class="flex column">
                  <span style="font-weight: bold">{{ empresa.cargo }}</span>
                  <span>{{ empresa.nomeEmpresa }}, {{ empresa.descricao }}</span>
                  <span>{{ empresa.dataInicio }} - {{ empresa.dataFim === '' ? 'Até o momento' : empresa.dataFim }}</span>
                </div>
                <q-btn v-if="$q.platform.is.cordova" style="flex: 0" @click="editEmpresa(index)" :disable="showEditEmpresaModal.view" flat size="12px"  icon="edit"/>
                <q-btn v-else style="flex: 0" @click="editEmpresa(index)" :disable="showEditEmpresaModal.view" flat size="12px" label="Editar Curso" icon="edit"/>
              </q-item>
            </q-list>

            <Empresa v-if="showEditEmpresaModal.view" :empresa-obj="empresasObj[showEditEmpresaModal.index]" @salvar="saveEmpresa" @remover="removeEmpresa"/>

            <div class="q-ma-md flex column items-center">
              <q-btn @click="addEmpresa" :disable="showEditEmpresaModal.view" icon="business" label="Adicionar Empresa"></q-btn>
            </div>

            <q-stepper-navigation>
              <q-btn type="submit" color="green" label="Continuar" />
              <q-btn flat @click="step = 3" color="green" label="Voltar" class="q-ml-sm" />
            </q-stepper-navigation>
          </q-form>
        </q-step>

        <q-step
          :name="5"
          title="Objetivos"
          icon="checklist"
          active-color="primary"
          done-color="green"
          :done="step > 5"
        >
          <q-form
            @submit="nextStep">
            <q-input
              v-model="cargoDesejado"
              class="q-pa-sm"
              maxlength="40"
              :rules="[ val => val.length > 0|| 'Esse campo é obrigatório !' ]"
              label="Cargo desejado">
            </q-input>

            <q-input
              v-model="areaInteresse"
              class="q-pa-sm"
              maxlength="80"
              label="Área de interesse">
            </q-input>

            <q-field
              v-model="pretensaoSalarial"
              label="Pretensão salarial"
              class="q-pa-sm"
              prefix="R$"
            >
              <template v-slot:control="{ id, floatingLabel, modelValue, emitValue }">
                <money3-component :id="id" class="q-field__input" :model-value="modelValue" @update:model-value="emitValue" v-bind="moneyFormatForDirective" v-show="floatingLabel" />
              </template>
            </q-field>

            <q-stepper-navigation>
              <q-btn type="submit" color="green" label="Continuar" />
              <q-btn flat @click="step = 4" color="green" label="Voltar" class="q-ml-sm" />
            </q-stepper-navigation>
          </q-form>
        </q-step>

        <q-step
          :name="6"
          title="Informações Complementares"
          icon="post_add"
          active-color="primary"
          done-color="green"
          :done="step > 6"
        >
          <q-form
            @submit="nextStep">
            <span class="q-pa-sm">Aceitaria viajar pela empresa?</span>
            <div class="q-pa-sm">
              <q-radio v-model="viajarEmpresa" label="Sim" val="sim"/>
              <q-radio v-model="viajarEmpresa" label="Não" val="nao"/>
            </div>

            <span class="q-pa-sm">Consideraria trabalhar em outra cidade?</span>
            <div class="q-pa-sm">
              <q-radio v-model="trabalharOutraCidadeEmpresa" label="Sim" val="sim"/>
              <q-radio v-model="trabalharOutraCidadeEmpresa" label="Não" val="nao"/>
            </div>

            <q-input
              v-model="informacoesComplementares"
              class="q-pa-sm"
              type="textarea"
              maxlength="500"
              label="Informações complementares">
            </q-input>

            <q-stepper-navigation>
              <q-btn type="submit" color="green" label="Gerar PDF" />
              <q-btn flat @click="step = 5" color="green" label="Voltar" class="q-ml-sm" />
            </q-stepper-navigation>
          </q-form>
        </q-step>
      </q-stepper>
    </q-page-container>
    <q-dialog v-model="showFileDialog">
      <q-card>
        <q-card-section class="flex row items-center">
          <q-icon size="md" color="positive" name="check"/>
          <span class="q-ml-md text-bold text-positive">Currículo Salvo Com Sucesso !</span>
        </q-card-section>
        <q-card-actions vertical>
          <q-btn v-if="$q.platform.is.cordova" @click="sharePDF" color="positive" icon="share" label="Compartilhar"></q-btn>
          <q-btn v-if="$q.platform.is.cordova" @click="openPDF" color="primary" icon="visibility" label="Visualizar PDF"></q-btn>
        </q-card-actions>
      </q-card>
    </q-dialog>
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
import {Money3Component} from "v-money3";
import { useQuasar } from 'quasar'

let doc = new jsPDF ("p", "mm", "a4");
let contador = 0;

export default defineComponent({
  name: 'MainLayout',

  components: {
    Money3Component,
    Empresa,
    Curso
  },

  methods: {
    sharePDF(){
      window.plugins.socialsharing.share(`Currículo PDF`, 'Currículo PDF',`file:///storage/emulated/0/Documents/${this.fileName}.pdf`,`file:///storage/emulated/0/Documents/${this.fileName}.pdf`)
    },

    filterCidade (val, update) {
      update(() => {
        if (val === '') {
          this.filtroCidadeOptions = this.optionsCidadeLocal
        }
        else {
          const needle = val.toLowerCase()
          this.filtroCidadeOptions = this.optionsCidadeLocal.filter(
            v => v.toLowerCase().indexOf(needle) > -1
          )
        }
      })
    },

    validEmail(email){
      const re = /^(([^<>()[\]\\.,;:\s@"]+(\.[^<>()[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/;
      return re.test(email);
    },
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
      this.empresasObj.push({nomeEmpresa: '', cargo: '', descricao: '', dataInicio: '', dataFim: ''});
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
      this.cursosObjs.push({nomeCurso: '', tipoCurso: '', instituicao: '', situacaoCurso: '', dataCurso: ''});
      this.showEditCursoModal.view = true;
      this.showEditCursoModal.index = this.cursosObjs.length - 1;
    },

    checkPermission(){
      const permissions = cordova.plugins.permissions;
      const self = this;
      permissions.hasPermission(permissions.WRITE_EXTERNAL_STORAGE, function( status ){
        if ( status.hasPermission ) {
          self.permissaoWriteFile = true;
        }
        else {
          permissions.requestPermission(permissions.WRITE_EXTERNAL_STORAGE, success, error);
          function error() {
            self.$q.notify({
              message: 'Para salavar seu PDF é necessario aceitar a permissão !',
              caption: 'Essa permissão é usada apenas para escrita e leitura do arquivo PDF :)',
              icon: 'warning',
              position: 'top',
              color: 'negative'
            })
          }
          function success( status ) {
            if( !status.hasPermission ){
              error();
            }else {
              self.permissaoWriteFile = true;
            }
          }
        }
      });
    },

    editCurso(index){
      this.showEditCursoModal.view = true;
      this.showEditCursoModal.index = index;
    },

    nextStep() {
      if(this.step === 6){
        if(this.$q.platform.is.cordova){
          if(this.permissaoWriteFile === false){
            this.checkPermission();
          }else{
            this.geratePDF()
          }
        }else{
          this.geratePDF()
        }
      }
      this.step = this.step + 1;
    },

    selectCity(){
      this.cidadeLocal = ''
      this.optionsCidadeLocal = [];
      City.getCitiesOfState(countries.alpha3ToAlpha2(this.paisLocal.value), this.estadoLocal.value).forEach(cidade =>{
        this.optionsCidadeLocal.push(cidade.name)
      })
    },

    selectState(){
      this.estadoLocal = '';
      this.optionsEstadoLocal = [];
      State.getStatesOfCountry(countries.alpha3ToAlpha2(this.paisLocal.value)).forEach(estado =>{
        this.optionsEstadoLocal.push({label: estado.name, value: estado.isoCode})
      })
    },

    openPDF(){
      cordova.plugins.fileOpener2.open(
        `file:///storage/emulated/0/Documents/${this.fileName}.pdf`,
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

    iso2FlagEmoji(iso){
      if(!iso) return null;
      iso = countries.alpha3ToAlpha2(iso);
      return String.fromCodePoint(...[...iso.toUpperCase()].map(char => char.charCodeAt(0) + 127397))
    },

     geratePDF() {
      function capitalize(s) {
          return s[0].toUpperCase() + s.slice(1);
      }

      async function gerateNewPage() {
          await doc.addPage ('a4', 'p');
      }

      this.$watch('lastY', (newY, oldY)=> {
        if(this.lastY >= 275){
          console.log('createNewPage')
          this.lastY = 22;
          gerateNewPage();
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
      if(this.file){
        doc.text('_________________________________________________________________________________________________________________', 20, this.lastY, {maxWidth: 140})
      }else{
        doc.text('__________________________________________________________________________________________________________________________________________________', 20, this.lastY, {maxWidth: 170})
      }


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

      if(this.paisNacionalidade.label){
        doc.setFont('helvetica', 'bold')
        this.lastY = this.lastY + 5;
        doc.text('País de Nacionalidade: ', 22, this.lastY)
        doc.setFont('helvetica', 'normal')
        doc.text(this.paisNacionalidade.label, 69, this.lastY)
      }

      if(this.telefone){
        doc.setFont('helvetica', 'bold')
        this.lastY = this.lastY + 5;
        doc.text('Telefone: ', 22, this.lastY)
        doc.setFont('helvetica', 'normal')
        doc.text(this.telefone, 42, this.lastY)
      }

      doc.setFont('helvetica', 'bold')
      this.lastY = this.lastY + 5;
      doc.text('Celular: ', 22, this.lastY)
      doc.setFont('helvetica', 'normal')
      doc.text(this.celular, 40, this.lastY)

      doc.setFont('helvetica', 'bold')
      this.lastY = this.lastY + 5;
      doc.text('Endereço: ', 22, this.lastY)
      doc.setFont('helvetica', 'normal')
      doc.text(`${this.endereco} ${this.bairroLocal} ${this.cep} ${this.cidadeLocal} ${this.estadoLocal.label ? this.estadoLocal.label : ''} ${this.paisLocal.label ? this.paisLocal.label : ''}`, 44, this.lastY,{maxWidth: 170})

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
         this.cursosObjs.forEach((curso, index) =>{
          doc.setFont('helvetica', 'bold')
          this.lastY = this.lastY + 10;
          doc.text(curso.tipoCurso, 22, this.lastY);
          doc.setFont('helvetica', 'normal')
          this.lastY = this.lastY + 5;
          doc.text(`${curso.nomeCurso}, ${curso.instituicao}\n(${curso.dataCurso}) - ${curso.situacaoCurso}`, 22, this.lastY);
          this.lastY = this.lastY + 5;
        })
      }


      if(this.empresasObj.length > 0){
        doc.setFontSize (23)
        doc.setTextColor(0, 0, 0)
        this.lastY = this.lastY + 15;
        doc.text('Histórico profissional', 20,this.lastY)

        doc.setFontSize(12);

         this.empresasObj.forEach((empresa, index) =>{
          doc.setFont('helvetica', 'bold')
          this.lastY = this.lastY + 10;
          doc.text(empresa.nomeEmpresa, 22, this.lastY);
          doc.setFont('helvetica', 'normal')
          this.lastY = this.lastY + 5;
          doc.text(`${empresa.cargo}, ${empresa.descricao}\n${empresa.dataInicio} - ${empresa.dataFim === '' ? 'Até o momento' : empresa.dataFim}`, 22, this.lastY);
          this.lastY = this.lastY + 5;
        })
      }

      doc.setFontSize (23)
      doc.setTextColor(0, 0, 0)
      this.lastY = this.lastY + 15;
      doc.text('Objetivos', 20,this.lastY)

      doc.setFontSize(12);

      doc.setFont('helvetica', 'bold')
      this.lastY = this.lastY + 8;
      doc.text('Cargo desejado: ', 22, this.lastY)
      doc.setFont('helvetica', 'normal')
      doc.text(this.cargoDesejado, 57, this.lastY)

      if(this.areaInteresse){
        doc.setFont('helvetica', 'bold')
        this.lastY = this.lastY + 5;
        doc.text('Área de interesse: ', 22, this.lastY)
        doc.setFont('helvetica', 'normal')
        doc.text(this.areaInteresse, 60, this.lastY)
      }

      if(this.pretensaoSalarial > 0.00){
        doc.setFont('helvetica', 'bold')
        this.lastY = this.lastY + 5;
        doc.text('Pretensão salarial: ', 22, this.lastY)
        doc.setFont('helvetica', 'normal')
        doc.text('R$ ' + this.pretensaoSalarial, 62, this.lastY)
      }


      doc.setFontSize (23)
      doc.setTextColor(0, 0, 0)
      this.lastY = this.lastY + 15;
      doc.text('Informações complementares', 20,this.lastY)

      doc.setFontSize(12);

      doc.setFont('helvetica', 'bold')
      this.lastY = this.lastY + 8;
      doc.text('Aceita viajar pela empresa: ', 22, this.lastY)
      doc.setFont('helvetica', 'normal')
      doc.text(this.viajarEmpresa === 'nao' ? 'Não' : capitalize(this.viajarEmpresa), 78, this.lastY)

      doc.setFont('helvetica', 'bold')
      this.lastY = this.lastY + 5;
      doc.text('Considera trabalhar em outra cidade: ', 22, this.lastY)
      doc.setFont('helvetica', 'normal')
      doc.text(this.trabalharOutraCidadeEmpresa === 'nao' ? 'Não' :capitalize(this.trabalharOutraCidadeEmpresa), 98, this.lastY)

      if(this.informacoesComplementares){
        this.lastY = this.lastY + 8;
        doc.setFont('helvetica', 'normal')
        doc.text(this.informacoesComplementares, 22, this.lastY, {maxWidth: 170})
      }

      const pdfOutput = doc.output ("arraybuffer");

      if(!Platform.is.cordova){
        doc.save(`Curriculo ${this.nome}.pdf`);
      }else {
        const self = this;
        if(contador === 0){
          self.fileName = `Curriculo ${self.nome}`;
        }else{
          self.fileName = self.fileName + Date.now();
        }

        window.resolveLocalFileSystemURL ('file:///storage/emulated/0/Documents', successCallback, errorCallback)

        function successCallback(fs) {
          fs.getFile (self.fileName + '.pdf', {create: true, exclusive: false}, function (fileEntry, dataObj) {

            fileEntry.createWriter (function (fileWriter) {
              fileWriter.onwrite = function (e) {
                self.showFileDialog = true;
              };
              fileWriter.onerror = function (e) {
                contador = contador + 1;
                self.$q.notify({
                  message: 'Falha ao salvar seu PDF !',
                  caption: 'Verifique se o App tem permissões de Escrita e Leitura em seu sistema e tente novamente',
                  icon: 'warning',
                  position: 'top',
                  color: 'negative'
                })
              };
              fileWriter.write (pdfOutput);
            }, errorCallback);
          }, errorCallback);
        }

        function errorCallback(error) {
          self.$q.notify({
            message: 'Falha ao salvar seu PDF !',
            caption: 'Verifique se o App tem permissões de Escrita e Leitura em seu sistema',
            icon: 'warning',
            position: 'top',
            color: 'negative'
          })
        }
          doc = new jsPDF ("p", "mm", "a4");
      }
       doc = new jsPDF ("p", "mm", "a4");
    },
  },

  mounted() {
    if(this.$q.platform.is.cordova){
      this.checkPermission();
    }

    listaPaises.forEach(pais =>{
      this.optionsPaisNacionalidade.push({label: pais.label, value: pais.label, flag: pais.code, dialCode: pais.dialCode});
      this.optionsPaisLocal.push({label: pais.label, value: pais.code})
    })

    this.itensMenu.forEach((menuItem, index) =>{
        for(let i = 0; i < menuItem.fields.length; i++){
          this.$watch(menuItem.fields[i].field, (newQuestion) => {
            if(newQuestion){
              if(newQuestion.label){
                menuItem.fields[i].status = newQuestion.label.length > 0
              }else{
                menuItem.fields[i].status = newQuestion.length > 0;
              }
            }
          })
        }
    })
  },

  data(){
    const step = 1;
    const nome = '';
    const dataNascimento = '';
    const estadoCivil = '';
    const sexo = ''
    const optionsSexo = ['Masculino', 'Feminino'];
    const optionsEstadoCivil = ['Solteiro(a)', 'Casado(a)', 'Separado(a)', 'Divorciado(a)', 'Viúvo(a)'];
    const paisNacionalidade = {label: '', value: '', flag: '', dialCode: ''};
    const optionsPaisNacionalidade = [];
    const email = '';
    const telefone = '';
    const celular = '';
    const cep = '';
    const paisLocal = {label: '', value: ''};
    const optionsPaisLocal = [{label: '', value: ''}]
    const estadoLocal = {label: '', value: ''};
    const optionsEstadoLocal = [{label: '', value: ''}];
    const cidadeLocal = '';
    const optionsCidadeLocal = [];
    const bairroLocal = '';
    const endereco = '';
    const leftDrawerOpen = null;
    const toggleLeftDrawer = null;
    const showEditCursoModal = {view: false, index: null};
    const showEditEmpresaModal = {view: false, index: null};
    const nivelEscolaridade = '';
    const optionsNivelEscolaridade = ['Ensino Fundamental Incompleto', 'Ensino Fundamental Completo', 'Ensino Médio Incompleto', 'Ensino Médio Completo', 'Formação Superior Incompleta', 'Formação Superior Completa', 'Pós-graduação no nível Especialização', 'Pós-graduação no nível Mestrado', 'Pós-graduação no nível Doutorado'];
    let cursosObjs = [];
    let empresasObj = [];
    const cargoDesejado = '';
    const areaInteresse = '';
    const pretensaoSalarial = '';
    const informacoesComplementares = '';
    const controleDados = 0;
    const viajarEmpresa = 'nao';
    const trabalharOutraCidade = 'nao';
    const showFileDialog = false;
    const filtroCidadeOptions = optionsCidadeLocal;
    const permissaoWriteFile = false;
    const fileName = ''
    const itensMenu = [
      {icon: 'person', label: 'Dados Pessoais', fields: [{label: 'Nome', status: false, field: 'nome'}, {label:'Data de Nascimento', status: false, field: 'dataNascimento'}, {label: 'País de Nascionalidade', status: false, field: 'paisNacionalidade'}]},
      {icon: 'contact_page', label: 'Informações de Contato', fields: [{label: 'E-mail', status: false, field: 'email'}, {label: 'Telefone', status: false, field: 'telefone'}, {label: 'Celular', status: false, field: 'celular'}, {label: 'CEP', status: false, field: 'cep'}, {label: 'País', status: false, field: 'paisLocal'},{label: 'Estado', status: false, field: 'estadoLocal'},{label: 'Cidade', status: false, field: 'cidadeLocal'},{label: 'Bairro', status: false, field: 'bairroLocal'}, {label: 'Endereço', status: false, field: 'endereco'},]},
      {icon: 'school', label: 'Formação', fields: [{label: 'Nível de Escolaridade', status: false, field: 'nivelEscolaridade'}]},
      {icon: 'checklist', label: 'Objetivos', fields: [{label: 'Cargo Desejado', status: false, field: 'cargoDesejado'}, {label: 'Área de Interesse', status: false, field: 'areaInteresse'}, {label: 'Pretensão Salarial', status: false, field: 'pretensaoSalarial'}]},
      {icon: 'post_add', label: 'Informações Complementares', fields: [ {label: 'Informações Complementares', status: false, field: 'informacoesComplementares'}]}]
    return{
      step: step,
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
      showEditCursoModal: showEditCursoModal,
      showEditEmpresaModal: showEditEmpresaModal,
      leftDrawerOpen: leftDrawerOpen,
      toggleLeftDrawer: toggleLeftDrawer,
      cargoDesejado: cargoDesejado,
      areaInteresse: areaInteresse,
      pretensaoSalarial: pretensaoSalarial,
      informacoesComplementares: informacoesComplementares,
      controleDados: controleDados,
      viajarEmpresa: viajarEmpresa,
      trabalharOutraCidadeEmpresa: trabalharOutraCidade,
      itensMenu: itensMenu,
      showFileDialog: showFileDialog,
      filtroCidadeOptions: filtroCidadeOptions,
      permissaoWriteFile: permissaoWriteFile,
      fileName: fileName,
      total: 17,
      lastY: 0,
      moneyFormatForDirective: {
        decimal: ',',
        thousands: '.',
        disableNegative: true,
        precision: 2,
      }
    }
  }

})
</script>

<style scoped>

</style>
