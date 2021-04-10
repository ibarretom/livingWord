<template>
  <v-container class='align-center flex-column' fill-height>
    <h1 class='titulo'> Escolha o Livro, Capítulo e Versículo</h1>
    <v-sheet   
      color="white"
      elevation="0"
      rounded
      class="home mx-auto my-5 container">
      <div class= 'd-flex flex wrap'>
          <v-select
          :items="livros"
          label="Livros"
          solo
          class='mx-2'
          v-model='livroSelecionado'
          @change="seleciona('livro')"
          ></v-select>

          <v-select
          :items="capitulos"
          label="capitulos"
          solo
          class='mx-2'
          v-model="capituloSelecionado"
          @change="seleciona('capitulo')"
          :disabled='!cap'
          ></v-select>

          <v-select
          :items="versicles"
          label="Versiculo"
          solo
          class='mx-1'
          @change="seleciona('versiculo')"
          v-model='versiculoSelecionado'
          :disabled='!vers'
          ></v-select>
      </div>
      <div v-if='mostraTexto'>
        <h2>{{livroSelecionado}} {{capituloSelecionado}}:{{versiculoSelecionado}}</h2>
        <article>{{texto}}</article> 
        <div class='d-flex justify-space-between'>
          <v-btn text @click="passaVersiculo('anterior')">anterior</v-btn>
          <v-btn text @click="passaVersiculo('proximo')">proximo</v-btn>
        </div>
      </div>
    </v-sheet >
  </v-container>
</template>

<script>
// @ is an alias to /src
import axios from 'axios'
export default {
  name: 'Home',
  data: () => ({
    objetoLivro: [],
    livros: [],
    capitulos: [],
    versicles: [],
    livroSelecionado: '',
    texto: '',
    capituloSelecionado:'',
    versiculoSelecionado:'',
    cap: false,
    vers: false,
    mostraTexto: false,
    abrev: ''
  }),
  components: {
  },
  methods:{
    /*Quando o usuário seleciona uma opção, chama a função slecionar 
    com o identificador da seleção. Pode ser: livro, capitulo ou versiculo*/ 
    async seleciona(opcao){
      const {objetoLivro, livroSelecionado, capituloSelecionado, versiculoSelecionado} = this

      /*caso tenha selecionado livro, eu busco no objetoLivro, qual a abreviação daquele livro
      pois no get eu preciso passar a abreviação do livro*/
      if(opcao === 'livro'){
        
        const busca = objetoLivro.find(livro => livro.name == livroSelecionado)
        this.abrev = busca.abbrev.pt
        this.capitulos = this.geraCapVers(busca.chapters)
        this.cap = true //mostra o próximo select

        /*Quando estiver com o capitulo selecionado, busco pela quantidade de versiculo para que seja selecionada.*/
      }else if(opcao === 'capitulo'){

        axios.get(`https://www.abibliadigital.com.br/api/verses/nvi/${this.abrev}/${capituloSelecionado}`)
        .then(resp => {
          this.versicles = this.geraCapVers(resp.data.chapter.verses)
          this.vers = true
        })
      
      /*Quado estiver com versiculo selecionado eu faço a requisição do texto*/
      }else if(opcao === 'versiculo'){
        axios.get(`https://www.abibliadigital.com.br/api/verses/nvi/${this.abrev}/${capituloSelecionado}/${versiculoSelecionado}`)
        .then(resp => {

          this.texto = resp.data.text
          this.mostraTexto = true
        })
      }
    },
    passaVersiculo(param){
      const {abrev, capituloSelecionado, versicles} = this

      if(this.versiculoSelecionado < versicles.length && this.versiculoSelecionado > 1){
        param === 'anterior'? this.versiculoSelecionado-- : this.versiculoSelecionado++
        axios.get(`https://www.abibliadigital.com.br/api/verses/nvi/${abrev}/${capituloSelecionado}/${this.versiculoSelecionado}`)
        .then(resp => {

          this.texto = resp.data.text
          this.mostraTexto = true

        })
        .catch(erro => {console.log(erro)})  
      }else if(this.versiculoSelecionado === 1){
        param === 'anterior'? 1 : this.versiculoSelecionado++
        axios.get(`https://www.abibliadigital.com.br/api/verses/nvi/${abrev}/${capituloSelecionado}/${this.versiculoSelecionado}`)
        .then(resp => {
          this.texto = resp.data.text
          this.mostraTexto = true

        })
      }

      
    },
    /*Essa função gera a quantidade de capítulos e versiculos, pois na api só vem a quantidade*/
    geraCapVers(quant){
      let arrayQuantidades = []
      for(let i = 1; i <= quant; i++){
        arrayQuantidades.push(i.toString())
      }
      return arrayQuantidades
    }
  },
  mounted() {
    //Quando estiver no processo de montagem eu pego todos os livros da bíblia
    axios.post('https://www.abibliadigital.com.br/api/users')
    .then()
    axios.get('https://www.abibliadigital.com.br/api/books').then(resp => {
      let arrayLivros = resp.data
      this.objetoLivro = arrayLivros
      arrayLivros.forEach(elemento => {
        this.livros.push(elemento.name)
      })
    })
  }
}
</script>
<style scoped>
@media (min-width: 1200px){
  .container{
   max-width: 40vw;
   max-height: 70vh;
  }  
}

.titulo{
  text-align: center;
}
</style>