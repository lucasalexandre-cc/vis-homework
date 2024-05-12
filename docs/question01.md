<style>
  canvas {
    width: 100%!important;
  }

  ol, p {
    max-width: unset;
  }
</style>

```js
// Load data
const spotify = (await FileAttachment("./data/spotify-2023.csv").csv()).filter(
  (music) => parseInt(music["streams"])
);

const allStreams = spotify.reduce((acc, m) => {
  return parseInt(m["streams"]) + acc;
}, 0);

// Load vega
import * as vega from "npm:vega";
import * as vegaLite from "npm:vega-lite";
import * as vegaLiteApi from "npm:vega-lite-api";

const vl = vegaLiteApi.register(vega, vegaLite);
```

# Questão 01

Existe alguma característica que faz uma música ter mais chance de se tornar popular?

---

<div class="grid grid-cols-2">
  <div id="width-div" class="card grid-colspan-2">
    <h2 class="title">[Gráfico 1] Distribuição de streams por características das músicas</h2>
    <div style="margin-top: 15px;">
      ${ vl.render(heatmap(divWidth - 250)) }
    </div>
  </div>
  <div class="card grid-colspan-2">
    <h2 class="title">[Gráfico 2] Como a instrumentabilidade influência os streams?</h2>
    <div style="margin-top: 15px;">
      ${ vl.render(bar(divWidth - 250)) }
    </div>
  </div>
  <div class="card grid-colspan-2">
    <h2 class="title">[Gráfico 3] Analisando os batimentos por minuto das musicas</h2>
    <div style="margin-top: 15px;">
      ${ vl.render(histogram(divWidth - 250)) }
    </div>
  </div>
</div>

```js
// Load div width
const divWidth = Generators.width(document.querySelector("#width-div"));
```

```js
// Heatmap Chart
function groupMusicCaracteriticsAndSumStreams(spotify) {
  const attributes = {
    "danceability_%": "Dançabilidade",
    "valence_%": "Valência",
    "energy_%": "Energia",
    "acousticness_%": "Acústica",
    "liveness_%": "Vivacidade",
    "speechiness_%": "Articulação vocal",
  };
  const keys = Object.keys(attributes);
  const data = [];

  for (const music of spotify) {
    for (const key of keys) {
      const subcategory = data.find(
        (element) =>
          element.category === key && element.subcategory === music[key]
      );

      if (subcategory) {
        subcategory.nStreams += parseInt(music["streams"]);
      } else {
        data.push({
          category: key,
          categoryInPortuguese: attributes[key],
          subcategory: music[key],
          nStreams: parseInt(music["streams"]),
        });
      }
    }
  }

  return data.map((subcategory) => ({
    ...subcategory,
    streamsPercentage: ((subcategory.nStreams / allStreams) * 100).toFixed(2),
  }));
}

function heatmap(divWidth) {
  return {
    spec: {
      width: divWidth,
      height: 350,
      data: {
        values: groupMusicCaracteriticsAndSumStreams(spotify),
      },
      mark: "rect",
      encoding: {
        y: {
          field: "categoryInPortuguese",
          type: "nominal",
          title: "Características",
        },
        x: {
          field: "subcategory",
          type: "ordinal",
          bin: {
            step: 20,
            extent: [0, 100],
          },
          title: "Valor das características em %",
        },
        color: {
          field: "streamsPercentage",
          type: "quantitative",
          title: "% total de streamings",
          aggregate: "sum",
          scale: {
            range: ["#DDACF5", "#64379F", "#27104E"],
          },
        },
        tooltip: [
          {
            field: "streamsPercentage",
            type: "quantitative",
            title: "Streams (%)",
            aggregate: "sum",
          },
        ],
      },
    },
  };
}
```

```js
// Bar Chart (instrumentability)
function groupInstrumentability(spotify) {
  const data = [
    { category: "0", value: 0 },
    { category: "1", value: 0 },
    { category: "2", value: 0 },
    { category: "Outros", value: 0 },
  ];

  for (const music of spotify) {
    if (parseInt(music["instrumentalness_%"]) <= 2) {
      data.find((d) => d.category === music["instrumentalness_%"]).value +=
        parseInt(music["streams"]);
    } else {
      data.find((d) => d.category === "Outros").value += parseInt(
        music["streams"]
      );
    }
  }

  return data.map((c) => ({
    ...c,
    streamPercentage: ((c.value / allStreams) * 100).toFixed(2),
  }));
}

function bar(divWidth) {
  return {
    spec: {
      width: divWidth,
      height: 350,
      data: {
        values: groupInstrumentability(spotify),
      },
      mark: "bar",
      encoding: {
        y: {
          field: "category",
          type: "nominal",
          title: "Instrumentalidade (%)",
        },
        x: {
          field: "streamPercentage",
          type: "quantitative",
          title: "Streamings (%)",
          aggregate: "sum",
        },
        color: {
          value: "#27104E",
        },
      },
    },
  };
}
```

```js
// Histogram Chart (BPM)
function groubByBpm(spotify) {
  const data = [];
  for (const music of spotify) {
    const category = data.find((element) => element.category === music["bpm"]);

    if (category) {
      category.nStreams += parseInt(music["streams"]);
    } else {
      data.push({
        category: music["bpm"],
        nStreams: parseInt(music["streams"]),
      });
    }
  }

  return data.map((category) => ({
    ...category,
    streamsPercentage: ((category.nStreams / allStreams) * 100).toFixed(2),
  }));
}

function histogram(divWidth) {
  return {
    spec: {
      width: divWidth,
      height: 350,
      data: {
        values: groubByBpm(spotify),
      },
      mark: "bar",
      encoding: {
        x: {
          field: "category",
          type: "quantitative",
          bin: true,
          title: "BPM",
        },
        y: {
          field: "streamsPercentage",
          type: "quantitative",
          aggregate: "sum",
          title: "Total Streams (%)",
        },
        color: {
          value: "#27104E",
        },
      },
    },
  };
}
```

## Análise

<div style="width: 100%;">

Analisando o **Gráfico 1**, podemos perceber alguns padrões com relação as características das músicas e como isso impacta sua popularidade.

1. **Acústica:** mais da metade dos streamings são de músicas com acústico na faixa de 0%-20%.
2. **Articulação vocal:** 87% dos streamings são de músicas com articulação vocal na faixa de 0%-20%.
3. **Dançabilidade**: a faixa de 40%-80% de dansabilidade parece ser a ideal levando em conta o número de streamings nessa faixa, chegando à 76% dos streamings.
4. **Energia**: apresenta um comportamento parecido com a "dançabilidade", com 72% dos streamings na faixa de 40%-80%.
5. **Valência**: aqui já vemos que os streamings se disperçam de forma bem semelhante nas faixas de valência das músicas. Apesar da faixa de 40%-60% ser a mais significativa, está bem próxima das faixas 20%-40% e 60%-80%.
6. **Vivacidade**: a grande maioria dos streamings são de músicas com vivacidade de 0%-20% (71% dos streamings).

Em caso de dúvidas com relação ao valor exato de cada faixa da característica, basta passar o mouse sobre o quadrante que ele renderizará um tooltip com a informação de quantos % de streamings aquele quadrante possui.

Já com o **Gráfico 2**, podemos analisar unitariamente a instrumentalidade da música e sua relação com os streamings. Mais de 90% do streaming possui instrumentalidade 0.

E com relação ao **Gráfico 3**, onde olhamos para BPM's (batimentos por minuto), vemos que as faixas de 80bpm-140bpm concentram a grande maioria dos streamings.

</div>

## Design utilizados

[TODO: falta descrever melhor os marcadores e canais visuais, e essa questão de separabilidade e discriminabilidade. Foquei mais em escrever um racional geral do gráfico]

#### Gráfico 01:

<div style="width: 100%;">

A escolha do heatmap foi para ser possível analisar em apenas um gráfico o impacto de várias características na popularidade da música. O heatpmap só foi possível de ser usado porque esses atributos possuem a mesma escala de dados (todos vão de 0% a 100%), tornando mais fácil de agrupa-los.

O viés que o heatmap pode causar é que ele não deixa claro o valor exato dos streamings para as faixas de atributos. Para mitigar esse efeito, foi implementado um hover no gráfico, onde se passado o mouse é mostrado o valor exato da célula.

Outro possível viés é que estou analisando quantidades de streamings de forma geral. Existe uma chance de algumas músicas outliers impactarem fortemente esses atributos, e enviezarem a análise do comportamento geral.

</div>

#### Gráfico 02:

<div style="width: 100%;">

A escolha do gráfico de barras foi tomada porque a grande maioria das músicas estão distribuídas em apenas 1 valor unitário da instrumentabilide. Se fosse adicionado ao heatmap, seria difícil identificar a distribuição dos dados, porque praticamente todas as músicas estariam entre os intervalos 0-20. Com um gráfico de barras simples, foi possível visualizar melhor que a maior parte das músicas está concentrado na instrumentabilidade 0.

</div>

#### Gráfico 03:

<div style="width: 100%;">

No gráfico responsável por analisar os BPM's, foi escolhido um histograma. O objetivo dele é similar ao heatmap (analisar a distruibuição das músicas a partir de uma escala de valores do atributo). Porém, os BPM'm possuem uma escala diferente dos outros atributos (não está contida no intervalo de 0 a 100). Por isso, foi usado um histograma separado apenas para analisa-lo.

Um possível viés desse gráfico são os valores usados para agrupar as músicas no histograma. Intervalos grandes podem omitir uma informação mais precisa com relação a qual faixa de batimentos tornam as músicas mais populares, enquanto intervalos pequenos demais podem dificultar de analisar padrões gerais do atributo.

</div>
