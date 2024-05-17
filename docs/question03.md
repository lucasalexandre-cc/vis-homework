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

// Load vega
import * as vega from "npm:vega";
import * as vegaLite from "npm:vega-lite";
import * as vegaLiteApi from "npm:vega-lite-api";

const vl = vegaLiteApi.register(vega, vegaLite);
```

# Questão 03

Discuta as diferenças entre as plataformas (Spotify, Deezer, Apple Music e Shazam)?

---

### Análise geral dos dados

<div class="grid grid-cols-2">
  <div id="width-div" class="card grid-colspan-1">
    <h2 class="title">[Gráfico 1.1] Distribuição das playlists</h2>  
    <div style="margin-top: 15px;">
      ${ vl.render(barChart(divWidth - 250, playlists, "Total Playlists")) }
    </div>
  </div>
  <div class="card grid-colspan-1">
    <h2 class="title">[Gráfico 1.2] Distribuição dos charts</h2>  
    <div style="margin-top: 15px;">
      ${ vl.render(barChart(divWidth - 250, charts, "Total Charts")) }
    </div>
  </div>
</div>

### Comparando as playlists e charts da plataforma nas principais músicas

```js
const type = view(
  Inputs.select(["playlists", "charts"], {
    label: "Qual dado analisar?",
  })
);
```

```js
const gender = type === "playlists" ? "das" : "dos";
```

<div class="grid grid-cols-2">
  <div class="card grid-colspan-2">
    <h2 class="title">[Gráfico 2] Distribuição ${gender} ${type} nas top5 músicas</h2>  
    <div style="margin-top: 15px;">
      ${ vl.render(groupBarChart(divWidth - 250, dataToShow, xLabel)) }
    </div>
  </div>
</div>

```js
// Load div width
const divWidth = Generators.width(document.querySelector("#width-div"));
```

```js
// Bar playlists and charts

const playlists = [
  { name: "Spotify", value: 0 },
  { name: "Apple", value: 0 },
  { name: "Deezer", value: 0 },
  { name: "Shazam", value: 0 },
];
const charts = [
  { name: "Spotify", value: 0 },
  { name: "Apple", value: 0 },
  { name: "Deezer", value: 0 },
  { name: "Shazam", value: 0 },
];

for (const music of spotify) {
  playlists[0].value += parseInt(music.in_spotify_playlists) || 0;
  charts[0].value += parseInt(music.in_spotify_charts) || 0;

  playlists[1].value += parseInt(music.in_apple_playlists) || 0;
  charts[1].value += parseInt(music.in_apple_charts) || 0;

  playlists[2].value += parseInt(music.in_deezer_playlists) || 0;
  charts[2].value += parseInt(music.in_deezer_charts) || 0;

  playlists[3].value += parseInt(music.in_shazam_playlists) || 0;
  charts[3].value += parseInt(music.in_shazam_charts) || 0;
}

function barChart(divWidth, data, xLabel) {
  return {
    spec: {
      width: divWidth,
      height: 350,
      data: {
        values: data,
      },
      mark: { type: "bar", size: 50 },
      encoding: {
        y: {
          field: "name",
          type: "nominal",
          title: "Plataforma",
          sort: {
            field: "value",
            order: "descending",
          },
        },
        x: {
          field: "value",
          type: "quantitative",
          title: xLabel,
        },
        color: {
          field: "name",
          type: "nominal",
          scale: {
            domain: ["Spotify", "Apple", "Deezer", "Shazam"],
            range: ["#1ED760", "#FF6C83", "#9D36F7", "#0E9DFE"],
          },
          legend: null,
        },
      },
    },
  };
}
```

```js
// Bar compare musics

function sortByStreams(musicA, musicB) {
  const musicAStreams = parseInt(musicA["streams"]);
  const musicBStreams = parseInt(musicB["streams"]);

  if (musicAStreams > musicBStreams) return -1;
  else return 1;
}

const top5Musics = spotify.sort(sortByStreams).slice(0, 5);
const musicsPlaylists = [];
const musicsCharts = [];
for (const music of top5Musics) {
  musicsPlaylists.push({
    category: music.track_name,
    group: "Spotify",
    value: parseInt(music.in_spotify_playlists) || 0,
  });
  musicsCharts.push({
    category: music.track_name,
    group: "Spotify",
    value: parseInt(music.in_spotify_charts) || 0,
  });

  musicsPlaylists.push({
    category: music.track_name,
    group: "Apple",
    value: parseInt(music.in_apple_playlists) || 0,
  });
  musicsCharts.push({
    category: music.track_name,
    group: "Apple",
    value: parseInt(music.in_apple_charts) || 0,
  });

  musicsPlaylists.push({
    category: music.track_name,
    group: "Deezer",
    value: parseInt(music.in_deezer_playlists) || 0,
  });
  musicsCharts.push({
    category: music.track_name,
    group: "Deezer",
    value: parseInt(music.in_deezer_charts) || 0,
  });

  musicsPlaylists.push({
    category: music.track_name,
    group: "Shazam",
    value: parseInt(music.in_shazam_playlists) || 0,
  });
  musicsCharts.push({
    category: music.track_name,
    group: "Shazam",
    value: parseInt(music.in_shazam_charts) || 0,
  });
}

const dataToShow = type === "playlists" ? musicsPlaylists : musicsCharts;
const xLabel = type === "playlists" ? "Playlists" : "Charts";

function groupBarChart(divWidth, data, xLabel) {
  return {
    spec: {
      width: divWidth,
      height: 300,
      data: {
        values: data,
      },
      mark: "bar",
      encoding: {
        y: {
          field: "category",
          type: "nominal",
          title: "Música",
          scale: {
            paddingInner: 0.4,
            paddingOuter: 0.1,
          },
          axis: {
            labelLimit: 100,
          },
        },
        x: {
          field: "value",
          type: "quantitative",
          title: xLabel,
        },
        yOffset: { field: "group" },
        color: {
          field: "group",
          type: "nominal",
          scale: {
            domain: ["Spotify", "Apple", "Deezer", "Shazam"],
            range: ["#1ED760", "#FF6C83", "#9D36F7", "#0E9DFE"],
          },
        },
      },
    },
  };
}
```

---

## Análises e designs

<div style="width: 100%;">

Nesta questão, serão discutidas as diferenças entre as quantidades de playlistas das plataformas, assim como distribuições das paradas musicais (charts). Os atributos analisados foram as maiores quantidades de músicas e artistas inseridos nas playlists e nas paradas musicais de cada aplicativo.

---

**Gráficos 1 e 1.2**

Em princípio, por estarem sendo tratados atributos quantitativos ordenáveis, a escolha do gráfico de barras horizontais adequou-se melhor a distribuição dos dados analisados. Além disso, nesse contexto, a fim de facilitar a comparação visual dos tamanhos de cada playlist, optamos pela diferenciação em cores, uma vez que estão sendo analisadas plataformas diferentes. Dessa forma, a utilização das colorações distintas ressalta aos olhos e destaca que as playlists do Spotify têm preferência massiva em relação as dos demais aplicativos. Por outro lado, a utilização das playlists no Deezer e a na Apple Music têm quantidades similares. O Shazam por ser um aplicativo com apelo distinto às demais plataformas, tem quantidade irrelevante em playlists.

Quanto ao gráfico de distribuição das paradas musicais, foi utilizado o mesmo padrão para codificação visual e para marcadores. Novamente, a distinção por cores ressalta as diferenças entre os dados analisados. Nesse caso, portanto, é perceptível um maior destaque das plataformas Apple e Shazam, em detrimento dos aplicativos com maiores quantidades de playlists.

A conclusão, portanto, é que a plataforma Spotify possui maior quantidade de playlist usufruídas pelos seus assinantes, seguido do Deezer e Apple Music. Enquanto os aplicativos Deezer e Shazam têm as maiores paradas musicais.

---

**Gráficos 2**

No gráfico 2, foram selecionadas as top 5 músicas, tendo em vista suas quantidades de streams e é feita a comparação de como estas faixas músicas não somente aparecem nas paradas das plataformas, como também nas playlists.

Os mesmos critérios visuais como o gráfico de barras horizontal ordenado e a distinção de cores utilizados em relação aos atributos de diferentes plataformas foi utilizado.

A principal perceção é que a plataforma Spotify é a única que contém todas faixas musicais em suas playlists.

---
