<template>
  <q-page class="flex flex-center">
    <div class="q-pa-md">
      <div class="row">
        <div class="col-12">
          <p>Valores</p>
          <div class="row">
            <div class="col-12">
              <q-input
                outlined
                v-model="input"
                label="Valores (separados por vírgula)"
              />
            </div>
          </div>
          <div class="row">
            <div class="col-12">
              <q-separator />
              <q-list bordered class="rounded-borders">
                <q-expansion-item
                  v-for="calc in calcs"
                  expand-separator
                  icon="calculate"
                  :key="calc.id"
                  :label="calc.label"
                >
                  <q-card>
                    <q-card-section :key="calc.latex">
                      {{ calc.latex }}
                    </q-card-section>
                  </q-card>
                </q-expansion-item>
              </q-list>
            </div>
          </div>
        </div>
      </div>
    </div>
  </q-page>
</template>

<script>
export default {
  name: "PageIndex",
  data: function() {
    return {
      input: "",
      entradas: [],
      latex: "",
      resultado: ""
    };
  },
  created() {
    this.reloadEntradas();
  },
  computed: {
    calcs() {
      let media = this.media()
      return [
        {label: 'Média: '  + media.result, id: 'media', latex: media.latex}
      ]
    }
  },
  watch: {
    input(input) {
      this.entradas = input.trimRight(",").split(",");
      this.saveEntradas();
      this.latex = "";
      console.log("Cleaning latex");
      this.resultado = this.media();
    },
    calcs() {
      this.$nextTick().then(() => {
        this.reRender();
      });
    }
  },
  methods: {
    saveEntradas() {
      window.localStorage.input = this.input;
    },
    reloadEntradas() {
      if (window.localStorage.input) {
        this.input = window.localStorage.input;
      }
    },
    reRender() {
      if (window.MathJax) {
        window.MathJax.Hub.Queue(["Typeset", window.MathJax.Hub]);
      }
    },
    media() {
      var total = 0;
      var entradas = this.entradas;
      let calculo = [];
      entradas.forEach(function(entrada) {
        let valor = parseInt(entrada);
        if (valor) {
          total += valor;
          calculo.push(entrada);
        }
      });
      var tamanho = this.entradas.length;
      if (this.input.endsWith(",")) {
        tamanho--;
      }
      var resultado = total / tamanho;
      let latex =
        "$$\\text{Quantidade de valores (n): " + tamanho + ".}\\\\$$";
      latex += "$$\\underline{x} = \\frac{\\sum_{i=1}^{n} x_i}{n}\\\\$$";
      latex +=
        "$$\\underline{x} = \\frac{" +
        calculo.join("+", calculo) +
        "}{" +
        tamanho +
        "}\\\\$$";
      latex += "$$\\underline{x} = " + resultado + "$$";
      return {
        result: resultado,
        latex: latex
        };
    },
    mediaPonderada: function() {
      var total = 0;
      var totalPeso = 0;
      let calculo = [];
      for (index in this.entradas) {
        let entrada = this.entradas[index];
        total += parseInt(entrada.valor) * parseInt(entrada.peso);
        totalPeso += parseInt(entrada.peso);
        calculo.push(entrada.valor + "*" + entrada.peso);
      }
      const resultado = total / totalPeso;
      var tamanho = this.entradas.length;
      this.latex +=
        "$$\\text{Quantidade de valores (n): " + tamanho + ".}\\\\$$";
      this.latex +=
        "$$\\underline{x} = \\frac{\\sum_{i=1}^{n} x_i p_i}{\\sum_{i=1}^{n} p_i}\\\\$$";
      this.latex +=
        "$$\\underline{x} = \\frac{" +
        calculo.join("+", calculo) +
        "}{" +
        this.entradas.length +
        "}\\\\$$";
      this.latex += "$$\\underline{x} = " + resultado + "$$";

      this.resultado = resultado;
    },
    mediana: function() {
      let passos = [];
      var tamanho = this.entradas.length;
      passos.push("\\text{Quantidade de valores (n): " + tamanho + ".}\\\\");
      var posicao = tamanho / 2;
      if (tamanho % 2 == 0) {
        passos.push("\\text{" + tamanho + " é par.}\\\\");
        passos.push("\\frac{" + tamanho + "}{2} = " + posicao + ".\\\\");
        passos.push(
          "Md = \\frac{x(" + posicao + ") + x(" + (posicao + 1) + ")}{2}. \\\\"
        );
        posicao--;
        var menor = parseFloat(this.entradas[posicao].valor);
        var maior = parseFloat(this.entradas[posicao + 1].valor);
        passos.push("Md = \\frac{" + menor + " + " + maior + "}{2}. \\\\");
        const resultado = (menor + maior) / 2;
        passos.push("Md = " + resultado);
        this.resultado = resultado;
        this.latex = "$$" + passos.join("") + "$$";
      } else {
        passos.push("\\text{" + tamanho + " é impar.}\\\\");
        passos.push("\\frac{" + tamanho + "}{2} = " + posicao + ". \\\\");
        const arredondamento = Math.ceil(posicao);
        passos.push(
          "\\text{Arredondando " +
            posicao +
            " para cima temos " +
            arredondamento +
            ".}\\\\"
        );

        let resultado = this.entradas[Math.floor(posicao)].valor;
        passos.push(
          "\\text{A mediana é " +
            resultado +
            " (" +
            arredondamento +
            "º valor). }\\\\"
        );

        this.resultado = resultado;
        this.latex = "$$" + passos.join("") + "$$";
      }
    },
    moda: function() {
      var totais = {};
      for (index in this.entradas) {
        let entrada = this.entradas[index];
        let valor = parseFloat(entrada.valor);
        totais[valor] = (totais[valor] || 0) + 1;
      }

      let passos = [];
      for (index in totais) {
        passos.push(
          "\\text{" + index + " aparece " + totais[index] + " vezes.}\\\\"
        );
      }

      const maior = Object.values(totais).reduce(function(a, b) {
        return a > b ? a : b;
      });

      let resultado = [];
      for (k in totais) {
        if (totais[k] == maior) {
          resultado.push(k);
        }
      }
      passos.push(
        "\\text{" +
          resultado.toString() +
          " aparece mais vezes (" +
          maior +
          ").}"
      );
      this.resultado = resultado.toString();
      this.latex = "$$" + passos.join("") + "$$";
    },
    variancia: function() {
      this.resultado = this.variancia;
    },
    desvio: function() {
      this.resultado = Math.sqrt(this.variancia);
      this.latex += "$$\\text {E por último o Desvio Padrão}(S)$$";
      this.latex += "$$ S=\\sqrt{S^2}\\\\$$";
      this.latex +=
        "$$S=\\sqrt{" + this.variancia + "}=" + this.resultado + "$$";
    },
    frequenciaAbsoluta: function() {
      var totais = {};
      for (index in this.entradas) {
        let entrada = this.entradas[index];
        let descricao = entrada.descricao;
        totais[descricao] =
          (totais[descricao] || 0) + parseFloat(entrada.quantidade);
      }
      return totais;
    },
    total: function() {
      var total = 0;
      for (index in this.entradas) {
        let entrada = this.entradas[index];
        total += parseFloat(entrada.quantidade);
      }
      return total;
    },
    frequenciaRelativa() {
      var totais = {};
      var total = this.total;
      this.latex = "$$ f_i= \\frac {N_i}{N}\\\\$$";
      this.latex +=
        "$$\\text {Onde } N_i \\text { é a frequência absoluta e N é o número total de elementos}\\\\$$";
      for (index in this.entradas) {
        let entrada = this.entradas[index];
        let descricao = entrada.descricao;
        let quantidade = parseFloat(entrada.quantidade);
        if (quantidade) {
          totais[descricao] = this.absoluta[descricao] / total;
          this.latex +=
            "$$f_\\text{" +
            descricao +
            "}= \\frac {" +
            quantidade +
            "}{" +
            total +
            "}\\\\ $$";
        }
      }
      return totais;
    }
  }
};
</script>
