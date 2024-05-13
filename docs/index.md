---
toc: true
---

<style>
    body, div, p, li, ol { max-width: none; }
</style>

# Músicas mais populares - Spotify 2023

Análise de um [conjunto de dados do Spotify](https://www.kaggle.com/datasets/nelgiriyewithana/top-spotify-songs-2023) contendo informações relevantes sobre as músicas mais populares da plataforma em 2023. Queremos responder basicamente 3 questões:

1. **Existe alguma característica que faz uma música ter mais chance de se tornar popular?**
2. **O conjunto das top 10 músicas e dos top 10 artistas varia muito se considerarmos apenas musicas lançadas no mesmo ano?**
3. **Discuta as diferenças entre as plataformas (Spotify, Deezer, Apple Music e Shazam)?**
---
## Data set

```js
const spotify = await FileAttachment("./data/spotify-2023.csv").csv();

view(Inputs.table(spotify));
```
---
## Alunos:

### 1 - Augusto Cesar da F. dos Santos
### 2 - Lucas Alexandre S. dos Santos 