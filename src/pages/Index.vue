<template>
  <q-page class="flex flex-center">
    <q-input outlined v-model="input" label="Outlined" />
    <br />Resultado: {{ resultado }}
  </q-page>
</template>

<script>
export default {
  name: "PageIndex",
  data: function() {
    return {
      input: "",
      entradas: [],
      resultado: ""
    };
  },
  watch: {
    input(input) {
      console.log("Input:", input);
      this.entradas = input.trimRight(',').split(",");
      this.resultado = this.media();
    }
  },
  methods: {
    media() {
      var total = 0;
      var entradas = this.entradas;
      let calculo = [];
      console.log(entradas);
      entradas.forEach(function(entrada){
        let valor = parseInt(entrada);
        if (valor) {
          total += valor;
          calculo.push(entrada.valor);
        }
      });
      console.log(total);
      var tamanho = this.entradas.length;
      if (this.input.endsWith(',')) {
        tamanho--;
      }
      var resultado = total / tamanho;
      this.latex +=
        "$$\\text{Quantidade de valores (n): " + tamanho + ".}\\\\$$";
      this.latex += "$$\\underline{x} = \\frac{\\sum_{i=1}^{n} x_i}{n}\\\\$$";
      this.latex +=
        "$$\\underline{x} = \\frac{" +
        calculo.join("+", calculo) +
        "}{" +
        tamanho +
        "}\\\\$$";
      this.latex += "$$\\underline{x} = " + resultado + "$$";
      return resultado;
    }
  }
};
</script>
