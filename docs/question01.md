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
    <h2 class="title">[Gráfico 1] Distribuição de por características das músicas</h2>
    <div style="margin-top: 15px;">
      ${ vl.render(heatmap(divWidth - 250)) }
    </div>
  </div>
</div>

<div class="grid grid-cols-2">
  <div class="card grid-colspan-2">
    <h2 class="title">[Gráfico 2] A influência da instrumentabilidade nos streams</h2>
    <div style="margin-top: 15px;">
      ${ vl.render(bar(divWidth - 250)) }
    </div>
  </div>
  <div class="card grid-colspan-2">
    <h2 class="title">[Gráfico 3] Os batimentos por minuto (BPM's) das músicas</h2>
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

## Análises e designs

<div style="width: 100%;">

Nas próximas linhas serão elucidadas as análises pertinentes aos atributos dos dados, utilizados em cada um dos gráficos acima, que fazem com que uma música tenha mais chance de se tornar popular. Vale ressaltar que em todos os gráficos foi utilizado o atributo streams, em virtude da correlação com as características musicais, quais sejam "Dançabilidade", "Valência", "Energia", "Acústica", "Vivacidade", "Articulação vocal", "BPM" e "Instrumentalidade". Também serão analisados os designs, os marcadores e os canais visuais. 

---

**Gráfico 1 - Distribuição de streams por características das músicas**

Cumpre informar, em princípio, que a escolha do gráfico do tipo heatmap tem como motivo à correlação entres os atributos, padrões e tendências dos dados supramencionados. Por essa razão, é possível depreender informações com relação às características musicais e analisar como a popularidade de cada música é impactada. 

Segue abaixo uma análise sucinta de como cada atributo musical relaciona-se com a quantidade de stream:

1. **Acústica:** mais da metade dos streamings são de músicas com acústico na faixa de 0% a 20%.
2. **Articulação vocal:** 87% dos streamings são de músicas com articulação vocal na faixa de 0% a 20%.
3. **Dançabilidade**: a faixa de 40%-80% de "dançabilidade" parece ser a ideal considerando o número de streamings, chegando à 76% dos streamings.
4. **Energia**: apresenta um comportamento parecido com a "dançabilidade" e possui 72% dos streamings na faixa de 40%-80%.
5. **Valência**: aqui pode ser notado que os streamings se dispersam de forma bem semelhante nas faixas de valência das músicas. Apesar da faixa de 40%-60% ser a mais significativa, existe a proximidade com as faixas de 20% a 40% e de 60% a 80%.
6. **Vivacidade**: a grande maioria dos streamings são de músicas com "vivacidade" de 0% a 20% (71% dos streamings).

Com relação ao design do gráfico 1 foram utilizados marcadores do tipo retângulo e canais visuais que representam a porcentagem dos streamings com cores que variam sua tonalidade de acordo com valores mínimos e máximos, posicionados horizontalmente. 

Reitera-se que a escolha do heatmap, teve como motivo a possibilidade de analisar, em apenas um gráfico, o impacto de várias características na popularidade da música. Uma vez que os atributos em análise, neste caso, possuem a mesma escala de dados (todos vão de 0% a 100%), a opção do gráfico de heatmap tornou-se viável, o que possibilitou, de foma mais fácil, o agrupamento dos dados.

Vale salientar que em caso de dúvidas com relação ao valor exato de cada faixa da característica, ao passar o mouse sobre o quadrante, será  renderizado um tooltip com a informação de quantos porcentos de streamings aquele quadrante possui.

---

**Gráfico 2 - A influência da instrumentabilidade nos streams**

A opção do gráfico de barras, neste caso, permite uma visualização mais palatável, uma vez que os atributos em análise são a instrumentalidade e os streamings. Dessa forma, é possível analisar, unitariamente, a característica musical instrumentalidade e sua correlação com os streamings. Ademais, é possível verificar, de imediato, que mais de 90% dos streamings possui instrumentalidade 0 (zero). 

O outro motivo que ratifica a escolha do gráfico de barras é em virtude da grande maioria das músicas estarem distribuídas em apenas um valor unitário da instrumentabilidade. Comparando-o ao gráfico do tipo heatmap, utilizado acima, esta análise se dificultaria no que tange a identificar a distribuição dos dados, haja vista que praticamente todas as músicas estariam entre os intervalos de 0 (zero) a 20 (vinte). Assim, com um gráfico de barras simples, foi possível visualizar que a maioria das músicas está concentrada na instrumentabilidade 0 (zero).

Com relação ao design do gráfico 2 foi utilizado o marcador do tipo barra, com canal visual em posição horizontal, tendo em vista a melhor expressividade e efetividade na representação da comparação com os atributos em questão. 

---

**Gráfico 3 - Os batimentos por minuto (BPM's) das músicas**

A escolha do gráfico do tipo histograma, nessa comparação, considera a correlação do atributo musical BPM's e a quantidade de streammings. É possível perceber na visualização do gráfico, o destaque concentrado nas faixas de 80bpm a 140bpm em virtude da grande maioria dos streamings. Nessa lógica, no intuito de explicitar somente as quantidades relevantes de BPM, optou-se pela exposição das quantidades de BPM na faixa de 60 a 220. 

Outrossim, um possível viés desse gráfico são os valores utilizados para agrupar as músicas no histograma. Intervalos grandes podem omitir uma informação mais precisa com relação a qual faixa de batimentos torna as músicas mais populares, enquanto intervalos pequenos demais podem dificultar a análise dos padrões gerais do atributo.

Vale ressaltar que o gráfico de histograma tem objetivo semelhante ao gráfico heatmap, ou seja, ambos conseguem mostrar a distribuição das músicas a partir de uma escala de valores do atributo. Porém, os BPM's possuem uma escala diferente dos outros atributos, isto é, não está contida no intervalo de valores de 0 a 100. Por isso, foi usado um histograma separado apenas para analisá-lo.

Com relação ao design do gráfico 3 foi utilizado o marcador do tipo barra, com canal visual em posição vertical, que facilita a comparação numérica entre faixas de valores distintos.

---
