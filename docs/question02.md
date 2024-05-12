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

# Questão 02

O conjunto das top 10 músicas e dos top 10 artistas varia muito se considerarmos apenas musicas lançadas no mesmo ano?

---

### Artistas

<div class="grid grid-cols-2">
  <div id="width-div" class="card grid-colspan-2">
    <h2 class="title">[Gráfico 1] Top10 Artistas</h2>
    <div style="margin-top: 15px;">
      ${ vl.render(barArtists(divWidth - 250, top10Artists)) }
    </div>
  </div>
</div>

```js
const yearArtistFilter = view(
  Inputs.select(musicYears, {
    label: "Ano lançamento",
  })
);
```

<div class="grid grid-cols-2">
  <div class="card grid-colspan-2">
    <h2 class="title">[Gráfico 2] Top10 Artistas contando apenas músicas lançadas em ${yearArtistFilter}</h2>
    <div style="margin-top: 15px;">
      ${ vl.render(barArtists(divWidth - 250, filteredTop10Artists, { highlight: filteredTop10ArtistsNamesHighlight, color: "#8050B2", highlightColor: "#27104E" })) }
    </div>
  </div>
</div>

### Músicas

<div class="grid grid-cols-2">
  <div class="card grid-colspan-2">
    <h2 class="title">[Gráfico 3] Top10 Músicas</h2>
    <div style="margin-top: 15px;">
      ${ vl.render(barMusics(divWidth - 250, top10Musics)) }
    </div>
  </div>
</div>

```js
const yearMusicFilter = view(
  Inputs.select(musicYears, {
    label: "Ano lançamento",
  })
);
```

<div class="grid grid-cols-2">
  <div class="card grid-colspan-2">
    <h2 class="title">[Gráfico 4] Top10 Musicas contando apenas músicas lançadas em ${yearMusicFilter}</h2>
    <div style="margin-top: 15px;">
      ${ vl.render(barMusics(divWidth - 250, filteredTop10Musics, { highlight: filteredTop10MusicsHighlightNames, color: "#8050B2", highlightColor: "#27104E" })) }
    </div>
  </div>
</div>

```js
// Load div width
const divWidth = Generators.width(document.querySelector("#width-div"));
```

```js
// load auxiliar methods
function sortByStreams(musicA, musicB) {
  const musicAStreams = parseInt(musicA["streams"]);
  const musicBStreams = parseInt(musicB["streams"]);

  if (musicAStreams > musicBStreams) return -1;
  else return 1;
}

/*
  config = {
    highlight: array<string>
    highlightColor: string
    color: string
  }
*/
function barArtists(
  divWidth,
  data,
  config = { highlight: [], color: "#27104E", highlightColor: "" }
) {
  return {
    spec: {
      width: divWidth,
      height: 350,
      data: {
        values: data,
      },
      mark: "bar",
      encoding: {
        y: {
          field: "name",
          type: "nominal",
          title: "Nome do artista",
          sort: {
            field: "streams",
            order: "descending",
          },
        },
        x: {
          field: "streams",
          type: "quantitative",
          title: "Total Streams",
          scale: {
            domain: [0, 17000000000],
          },
        },
        color: {
          value: config.color,
          condition: [
            {
              value: config.highlightColor,
              test: { field: "name", oneOf: config.highlight },
            },
          ],
        },
      },
    },
  };
}
function barMusics(
  divWidth,
  data,
  config = { highlight: [], color: "#27104E", highlightColor: "" }
) {
  return {
    spec: {
      width: divWidth,
      height: 350,
      data: {
        values: data,
      },
      mark: "bar",
      encoding: {
        y: {
          field: "track_name",
          type: "nominal",
          title: "Nome da musica",
          sort: {
            field: "streams",
            order: "descending",
          },
        },
        x: {
          field: "streams",
          type: "quantitative",
          title: "Total Streams",
          scale: {
            domain: [0, 4000000000],
          },
        },
        color: {
          value: config.color,
          condition: [
            {
              value: config.highlightColor,
              test: { field: "track_name", oneOf: config.highlight },
            },
          ],
        },
      },
    },
  };
}

function onlyUnique(value, index, array) {
  return array.indexOf(value) === index;
}
const musicYears = spotify
  .map((d) => parseInt(d["released_year"]))
  .sort()
  .reverse()
  .filter(onlyUnique)
  .map((y) => y.toString());
```

```js
// Top 10 artists
const artists = [];
for (const music of spotify) {
  if (!parseInt(music.streams)) continue;

  const artist = artists.find((a) => a.name === music["artist(s)_name"]);
  if (!artist)
    artists.push({
      name: music["artist(s)_name"],
      streams: parseInt(music.streams),
    });
  else artist.streams += parseInt(music.streams);
}

const top10Artists = artists.sort(sortByStreams).slice(0, 10);

function getOrderedArtistsFilterByReleaseDate() {
  const filteredMusics = spotify.filter(
    (music) => music["released_year"] == yearArtistFilter
  );

  const artists = [];
  for (const music of filteredMusics) {
    if (!parseInt(music.streams)) continue;

    const artist = artists.find((a) => a.name === music["artist(s)_name"]);
    if (!artist)
      artists.push({
        name: music["artist(s)_name"],
        streams: parseInt(music.streams),
      });
    else artist.streams += parseInt(music.streams);
  }

  return artists.sort(sortByStreams);
}

const filteredTop10Artists = getOrderedArtistsFilterByReleaseDate().slice(
  0,
  10
);
const filteredTop10ArtistsNamesHighlight = filteredTop10Artists
  .filter((artist) => {
    const top10ArtistisNames = top10Artists.map((a) => a.name);

    return top10ArtistisNames.includes(artist.name);
  })
  .map((a) => a.name);
```

```js
// top 10 musics

const top10Musics = spotify.sort(sortByStreams).slice(0, 10);
const filteredTop10Musics = spotify
  .filter((music) => music["released_year"] == yearMusicFilter)
  .sort(sortByStreams)
  .slice(0, 10);
const filteredTop10MusicsHighlightNames = filteredTop10Musics
  .filter((music) => {
    const top10MusicsNames = top10Musics.map((m) => m.track_name);

    return top10MusicsNames.includes(music.track_name);
  })
  .map((m) => m.track_name);
```

## Análise

<div style="width: 100%;">

Analisando os **gráficos 1 e 2**, que dizem respeito aos artistas mais famosos, podemos ver que se considerarmos apenas músicas lançadas em 2023, nenhum artista está no top10 geral. Agora se modificarmos para anos próximos, como 2022, podemos ver que artistas como "Bad Bunny", "Harry Styles", "Taylor Swift" e "The Weeknd" que marcam presença em ambos os top10. É possível fazer a mesma análise para qualquer um dos outros anos.

E com relação aos **gráficos 3 e 4**, eles nos mostram que esse comportamento ocorre de forma semelhante com relação as músicas. Se olharmos apenas as músicas lançadas em 2023, nenhuma delas figura no top10 geral das músicas. Apenas em 2021 é possível ver uma música que figura em ambos os top10 ("STAY (with Justin Bieber)"). Esse comportamento parece fazer sentido, dado que músicas mais antigas estão disponíveis por mais tempo na plataforma e com isso tiveram mais tempo para receber mais streamings.

</div>

## Design utilizados

[TODO: falta descrever melhor os marcadores e canais visuais, e essa questão de separabilidade e discriminabilidade. Foquei mais em escrever um racional geral do gráfico]

#### Gráficos 01 e 03:

<div style="width: 100%;">

Escolhi um gráfico de barras ordenado comum para mostrar os top10 artistas e as top10 músicas. Ele mostra de um jeito prático e direto a informação necessária para análise. Na escala, foi colocado o número de streamings, para ficar mais fácil comparar posteriormente os gráficos 01 e 03 com o 02 e 04.

</div>

#### Gráficos 02 e 04:

<div style="width: 100%;">

Foi escolhido um gráfico de barra, assim como os gráficos 01 e 03, porém com algumas diferenças:

- Escala: a escalada do gráfico 02 se manteve igual à do gráfico 01, assim como a do gráfico 04 se manteve igual ao gráfico 03. Isso foi proposital, para ser mais fácil comparar a quantidade de streamings entre os gráficos.
- Cor: as barras desses gráficos apresentam por default uma cor mais clara que as dos gráficos "pais". Porém, quando uma música ou um artista aparece em ambos os gráficos, ele assume uma cor idêntica ao gráfico "pai". Isso foi proposital para que fosse mais fácil entender quando as músicas/artistas se repetiam entre os gráficos.

</div>
