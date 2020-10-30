<template>
  <q-page class="flex flex-center">
    <div class="q-pa-md full-width">
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
            <div class="col-12">
              <q-input
                outlined
                v-model="weight"
                label="Pesos (separados por vírgula)"
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
      inputs: [],
      weight: "",
      weights: [],
      latex: "",
      calcs: [],
      resultado: ""
    };
  },
  created() {
    this.reloadInputs();
    this.$nextTick().then(() => {
      this.reRender();
    });
  },
  computed: {
  },
  watch: {
    input(input) {
      this.inputs = input.split(",");
      if (input.endsWith(",")) {
        this.inputs.splice(this.inputs.length - 1);
      }

      this.saveInputs();
      this.calc();
    },
    weight(input) {
      this.weights = input.split(",");
      if (input.endsWith(",")) {
        this.weights.splice(this.weights.length - 1);
      }
      this.saveInputs();
      this.calc();
    },
    calcs() {
      this.$nextTick().then(() => {
        this.reRender();
      });
    }
  },
  methods: {
    calc() {
      let media = this.media();
      let mediaPonderada = this.mediaPonderada();
      let mediana = this.mediana();
      let moda = this.moda();
      this.calcs = [
        {label: 'Média: '  + media.result, id: 'media', latex: media.latex},
        {label: 'Média Ponderada: '  + mediaPonderada.result, id: 'media-ponderada', latex: mediaPonderada.latex},
        {label: 'Médiana: '  + mediana.result, id: 'mediana', latex: mediana.latex},
        {label: 'Moda: '  + moda.result, id: 'moda', latex: moda.latex},
      ];
    },
    saveInputs() {
      window.localStorage.input = this.input;
      window.localStorage.weight = this.weight;
    },
    reloadInputs() {
      if (window.localStorage.input) {
        this.input = window.localStorage.input;
      }
      if (window.localStorage.weight) {
        this.weight = window.localStorage.weight;
      }
    },
    reRender() {
      if (window.MathJax) {
        window.MathJax.Hub.Queue(["Typeset", window.MathJax.Hub]);
      }
    },
    media() {
      var total = 0;
      let calculo = [];
      for (var index in this.inputs) {
        let valor = parseInt(this.inputs[index]);
        if (valor) {
          total += valor;
          calculo.push(this.inputs[index]);
        }
      };
      var tamanho = this.inputs.length;
      var resultado = total / tamanho;
      let latex =
        "$$\\text{Quantidade de valores (n): " + tamanho + ".}\\\\$$";
      
      if (tamanho == 0) {
        return {
          result: 0,
          latex: latex
        };
      }

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
      let total = 0;
      let totalPeso = 0;
      let solution = [];
      for (let index in this.inputs) {
        const input = parseInt(this.inputs[index]);
        const weight = ('undefined' !== typeof this.weights[index]) ? parseInt(this.weights[index]) : 1;
        total += input * weight;
        totalPeso += weight;
        solution.push(input + "*" + weight);
      };
      const resultado = total / totalPeso;
      const tamanho = this.inputs.length;
      let latex =
        "$$\\text{Quantidade de valores (n): " + tamanho + ".}\\\\$$";
      latex +=
        "$$\\underline{x} = \\frac{\\sum_{i=1}^{n} x_i p_i}{\\sum_{i=1}^{n} p_i}\\\\$$";
      latex +=
        "$$\\underline{x} = \\frac{" +
        solution.join("+", solution) +
        "}{" +
        tamanho +
        "}\\\\$$";
      latex += "$$\\underline{x} = " + resultado + "$$";

      return {
        result: resultado,
        latex: latex
      };
    },

    mediana: function() {
      let passos = [];
      var tamanho = this.inputs.length;
      passos.push("\\text{Quantidade de valores (n): " + tamanho + ".}\\\\");
      var posicao = tamanho / 2;
      let resultado = 0;

      if (tamanho % 2 == 0) {
        passos.push("\\text{" + tamanho + " é par.}\\\\");
        passos.push("\\frac{" + tamanho + "}{2} = " + posicao + ".\\\\");
        passos.push(
          "Md = \\frac{x(" + posicao + ") + x(" + (posicao + 1) + ")}{2}. \\\\"
        );
        posicao--;
        var menor = parseFloat(this.inputs[posicao]);
        var maior = parseFloat(this.inputs[posicao + 1]);
        passos.push("Md = \\frac{" + menor + " + " + maior + "}{2}. \\\\");
        resultado = (menor + maior) / 2;
        passos.push("Md = " + resultado);
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
        resultado = this.inputs[Math.floor(posicao)];
        passos.push(
          "\\text{A mediana é " +
            resultado +
            " (" +
            arredondamento +
            "º valor). }\\\\"
        );
      }

      return {
        result: resultado,
        latex: "$$" + passos.join("") + "$$"
      };
    },
    moda: function() {
      let totais = {};
      for (let index in this.inputs) {
        let entrada = this.inputs[index];
        let valor = parseFloat(entrada);
        totais[valor] = (totais[valor] || 0) + 1;
      }

      let passos = [];
      for (let index in totais) {
        passos.push(
          "\\text{" + index + " aparece " + totais[index] + " vezes.}\\\\"
        );
      }

      const maior = Object.values(totais).reduce(function(a, b) {
        return a > b ? a : b;
      });

      let resultado = [];
      for (let k in totais) {
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
      return {
        result: resultado.toString(),
        latex: "$$" + passos.join("") + "$$"
      }
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
      for (index in this.inputs) {
        let entrada = this.inputs[index];
        let descricao = entrada.descricao;
        totais[descricao] =
          (totais[descricao] || 0) + parseFloat(entrada.quantidade);
      }
      return totais;
    },
    total: function() {
      var total = 0;
      for (index in this.inputs) {
        let entrada = this.inputs[index];
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
      for (index in this.inputs) {
        let entrada = this.inputs[index];
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
