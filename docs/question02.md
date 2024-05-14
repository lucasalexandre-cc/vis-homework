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


---

## Análises e designs

<div style="width: 100%;">

Os gráficos acima consideram como os conjuntos das top 10 músicas e dos top 10 artistas variam em suas posições nos diferentes anos. Em todos os gráficos, similar a questão anterior, foi utilizado o atributo streams, em virtude da correlação com os nomes dos artistas e os nomes das faixas musicais. Nas próximas linhas também serão analisados os designs, os marcadores e os canais visuais.

</div>

---

## Artistas e Músicas

<div style="width: 100%;">

**Gráficos 1 e 2:**

Ao analisar os **gráficos 1 e 2**, que dizem respeito aos artistas mais famosos, pode-se verificar que se considerarmos apenas músicas lançadas em 2023, nenhum artista está nos Top 10 geral. Agora, ao modificarmos para anos próximos, como, por exemplo, 2022, podemos verificar que artistas como "Bad Bunny", "Harry Styles", "Taylor Swift" e "The Weekend" permanecem presentes em ambos os Top 10. Além disso, é possível fazer a mesma análise para qualquer um dos outros anos.

**Gráficos 3 e 4:**

Quanto aos **gráficos 3 e 4**, eles nos mostram que esse comportamento ocorre de forma semelhante com relação às músicas. Se olharmos apenas as músicas lançadas em 2023, nenhuma delas é exibida no Top 10 geral das músicas. Vale notar que apenas em 2021 é possível ver uma música que figura em ambos os Top 10, que é a faixa ("STAY (with Justin Bieber)"). Esse comportamento parece fazer sentido, dado que músicas mais antigas estão disponíveis por mais tempo na plataforma e com isso, tiveram mais tempo para receber mais streaming.


--- 

</div>

## Design utilizados


<div style="width: 100%;">

De modo geral, todos os gráficos desta questão são do tipo gráfico de barras ordenados e em posição horizontal, em virtude da melhor visualização, tendo em vista a representação das top 10 músicas e dos top 10 artistas. Tais gráficos conseguem elucidar de forma prática, direta e imediata as informações correlacionadas na análise. Assim, no intuito de facilitar as comparações entre os gráfico 01 e 03 com os gráficos 02 e 04, na escala foi utilizado o número de streaming.    

No entanto, vale ressaltar que existem diferenças, por ocasião da visualização, na comparação entre os gráficos 02 e 04 e os gráficos 01 e 03. São elas:

- Escalas: Assumimos que os gráficos 01 e 03 foram origem para confecção dos gráficos descendentes 02 e 04, respectivamente. Assim, a escalada do gráfico 02 se manteve igual a do 01, bem como a do gráfico 04, mante-se igual a do 03. Isso foi proposital, para ser mais fácil comparar as quantidades de streaming entre eles.

- Cores: as barras dos gráficos descendentes, apresentam, por padrão, uma cor mais clara que as dos gráficos dos quais se originam. Porém, quando uma música ou um artista aparece em ambos os gráficos, ele assume uma cor idêntica à origem. Por exemplo, nos gráficos relacionados aos artistas, ao selecionar o ano de lançamento ("Ano lançamento") em "2022", pode-se perceber que os artistas "Bad Bunny", "Harry Styles" e "Taylor Swift" diferenciam-se dos demais, mudando sua tonalidade de cor. O mesmo ocorre no gráfico relacionado às músicas. Tal efeito no canal visual foi proposital, com intuito de destacar as músicas/artistas que se repetem.


</div>

---