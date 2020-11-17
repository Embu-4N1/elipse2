<template>
  <q-page class="flex flex-center">
    <div class="q-pa-md full-width">
      <!--div class="row">
        <h2>Enunciado</h2>
        <q-editor v-model="statement" min-height="5rem" class="col-12" />
      </div-->
      <div class="row">
        <div class="col-12">
          <h2>Valores</h2>
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
                label="Pesos / Quantidade (separados por vírgula)"
              />
            </div>
          </div>
          <div class="row">
            <h2>Resultados</h2>
            <q-list bordered class="rounded-borders col-12">
              <q-expansion-item label="Medidas de posição e disperção">
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
              </q-expansion-item>
              <q-expansion-item label="Distribuição de frequência">
                <q-markup-table>
                  <thead>
                    <tr>
                      <th class="text-left">Descrição</th>
                      <th class="text-left">Absoluta</th>
                      <th class="text-left">Relativa</th>
                    </tr>
                  </thead>
                  <tr v-for="(value, index) in frequency" :key="index">
                    <td class="text-right">{{ index }}</td>
                    <td class="text-right">{{ value.absolute }}</td>
                    <td class="text-right">{{ value.relative }}</td>
                  </tr>
                </q-markup-table>
              </q-expansion-item>
            </q-list>
          </div>
        </div>
      </div>
    </div>
  </q-page>
</template>
<style scoped>
h2 {
  font-size: 21px;
  font-weight: bold;
}
</style>
<script>
import { mapState } from "vuex";

export default {
  name: "PageIndex",
  data: function() {
    return {
      statement: "",
      input: "",
      inputs: [],
      weight: "",
      weights: [],
      latex: "",
      calcs: [],
      frequency: {},
      history: []
    };
  },

  created() {
    if (this.exampleId) {
      this.loadExample(this.exampleId);
    } else if (this.paramHistory) {
      this.loadHistory(this.paramHistory);
    } else {
      this.reloadInputs();
    }
    this.$nextTick().then(() => {
      this.reRender();
    });
  },

  beforeRouteUpdate(to, from, next) {
    this.loadExample(to.params.example);
    next();
  },

  computed: {
    exampleId() {
      // let id = parseInt(Number.isNaN(this.$route.params.example));
      // console.log(this.$route.params.example, id);
      // if (Number.isNaN(id)) {
      //   return false;
      // }
      return this.$route.params.example;
    },
    paramHistory() {
      return this.$route.params.history;
    }
  },
  watch: {
    $route: "updateExample",
    input(input) {
      let inputs = input.split(",");
      if (input.endsWith(",")) {
        inputs.splice(inputs.length - 1);
      }
      if (JSON.stringify(inputs) != JSON.stringify(this.inputs)) {
        this.inputs = inputs;
        this.saveInputs();
        this.calc();
      }
    },

    inputs() {
      this.saveHistory();
    },

    weights(){
      this.saveHistory();
    },

    weight(input) {
      this.weights = input.split(",");
      if (input.endsWith(",")) {
        this.weights.splice(this.weights.length - 1);
      }
      this.saveInputs();
      this.calc();
    },

    statement(input) {
      this.saveInputs();
    },

    calcs() {
      this.$nextTick().then(() => {
        this.reRender();
      });
    }
  },
  methods: {
    updateExample() {
      this.loadExample(this.exampleId);
    },

    async loadExample(id) {
      const response = await fetch(process.env.API + "/api/example/" + id);
      if (response.ok) {
        let example = await response.json();
        this.input = example.input;
        this.weight = example.weight;
        this.statement = example.statement;
      }
    },

    loadHistory(history) {
        let example = JSON.parse(decodeURIComponent(history));
        this.input = example.input;
        this.weight = example.weight;
        this.statement = example.statement;

    },

    calc() {
      let media = this.media();
      let mediaPonderada = this.mediaPonderada();
      let mediana = this.mediana();
      let moda = this.moda();
      let variancia = this.variancia();
      let desvio = this.desvio();
      this.calcs = [
        { label: "Média: " + media.result, id: "media", latex: media.latex },
        {
          label: "Média Ponderada: " + mediaPonderada.result,
          id: "media-ponderada",
          latex: mediaPonderada.latex
        },
        {
          label: "Mediana: " + mediana.result,
          id: "mediana",
          latex: mediana.latex
        },
        { label: "Moda: " + moda.result, id: "moda", latex: moda.latex },
        {
          label: "Variância: " + variancia.result,
          id: "variancia",
          latex: variancia.latex
        },
        {
          label: "Desvio Padrão: " + desvio.result,
          id: "desvio",
          latex: desvio.latex
        }
      ];
      this.calcFrequency();
    },

    saveInputs() {
      window.localStorage.input = this.input;
      window.localStorage.weight = this.weight;
      window.localStorage.statement = this.statement;
    },

    saveHistory() {
      let data = {
        input: this.input,
        weight: this.weight,
        statement: this.statement
      };
      let history = window.localStorage.history ? JSON.parse(window.localStorage.history) : [];
      this.history = history.filter(value => JSON.stringify(value) != JSON.stringify(data));

      this.history.push(data);
      window.localStorage.history = JSON.stringify(this.history);
    },
    reloadInputs() {
      if (window.localStorage.input) {
        this.input = window.localStorage.input;
      }
      if (window.localStorage.weight) {
        this.weight = window.localStorage.weight;
      }
      if (window.localStorage.statement) {
        this.statement = window.localStorage.statement;
      }
      if (window.localStorage.history) {
        this.history = JSON.parse(window.localStorage.history);
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
      }
      var tamanho = this.inputs.length;
      var resultado = total / tamanho;
      let latex = "$$\\text{Quantidade de valores (n): " + tamanho + ".}\\\\$$";

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

    getWeight: function(index) {
      return ("undefined" !== typeof this.weights[index] && this.weights[index] !== "")
      ? parseFloat(this.weights[index])
      : 1;

    },

    mediaPonderada: function() {
      let total = 0;
      let totalPeso = 0;
      let solution = [];
      for (let index in this.inputs) {
        const input = parseInt(this.inputs[index]);
        const weight = this.getWeight(index);
        total += input * weight;
        totalPeso += weight;
        solution.push(input + "*" + weight);
      }
      const resultado = total / totalPeso;
      const tamanho = this.inputs.length;
      let latex = "$$\\text{Quantidade de valores (n): " + tamanho + ".}\\\\$$";
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
      };
    },

    variancia: function() {
      let total = 0;
      let latex = "$$\\text {Calculando a média}(\\underline{x})$$";
      let media = this.media();
      latex += media.latex;
      let calculo = [];
      let results = []

      for (let index in this.inputs) {
        let entrada = this.inputs[index];
        let valor = parseFloat(entrada);
        let result = Math.pow(valor - media.result, 2);
        total = total + result;
        let under = parseInt(index) + 1;
        calculo.push(`\\text{Para }x_${under}=${valor}, (${valor} - ${media.result})^2 = ${result} \\\\`);
        results.push(result);
      }
      latex += "$$\\text{Resolvendo }(x_i-\\underline{x})^2: $$";
      latex += `$$${calculo.join("")}$$`;

      let resultado = total / (this.inputs.length - 1);
      latex += "$$\\text{A variância da amostra }(S^2)$$";
      latex +=
        "$$S^2 = \\frac{\\sum_{i=1}^{n} (x_i-\\underline{x})^2}{n-1}\\\\$$";
      latex +=
        "$$S^2 = \\frac{" +
        results.join("+") +
        "}{" +
        this.inputs.length +
        "-1}\\\\$$";
      latex += "$$S^2 = " + resultado + "$$";
      return {
        result: resultado,
        latex: latex
      };
    },

    desvio: function() {
      let variancia = this.variancia();
      let resultado = Math.sqrt(variancia.result);
      let latex = variancia.latex;
      latex += "$$\\text {E por último o Desvio Padrão}(S)$$";
      latex += "$$ S=\\sqrt{S^2}\\\\$$";
      latex += "$$S=\\sqrt{" + variancia.result + "}=" + resultado + "$$";
      return {
        result: resultado,
        latex: latex
      };
    },

    calcFrequency() {
      this.frequency = {};
      this.frequenciaAbsoluta();
      this.frequenciaRelativa();
    },

    frequenciaAbsoluta: function() {
      for (let index in this.inputs) {
        let entrada = this.inputs[index];
        if ("undefined" === typeof this.frequency[entrada]) {
          this.frequency[entrada] = {
            relative: 0,
            absolute: 0
          };
        }
        this.frequency[entrada].absolute = this.frequency[entrada].absolute + this.getWeight(index);
      }
    },

    total: function() {
      var total = 0;
      for (let index in this.inputs) {
        total += this.getWeight(index);
      }
      return total;
    },

    frequenciaRelativa() {
      let total = this.total();
      let latex = "$$ f_i= \\frac {N_i}{N}\\\\$$";
      latex +=
        "$$\\text {Onde } N_i \\text { é a frequência absoluta e N é o número total de elementos}\\\\$$";
      for (let index in this.inputs) {
        let entrada = this.inputs[index];
        let quantidade = this.getWeight(index);
        if (quantidade) {
          this.frequency[entrada].relative =
            this.frequency[entrada].absolute / total;
          this.latex +=
            "$$f_\\text{" +
            entrada +
            "}= \\frac {" +
            quantidade +
            "}{" +
            total +
            "}\\\\ $$";
        }
      }
    }
  }
};
</script>
