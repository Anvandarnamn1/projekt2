<script>
	import { onMount } from "svelte";

const WORKER = 'https://icy-limit-9e2a.antonwestin08.workers.dev/';
let news = $state([])
  let loading = $state(true)
  let count = $state(2)
  let langs = $state(['eng'])

  function toggleLang(lang) {
      if (langs.includes(lang)) {
          if (langs.length > 1) langs = langs.filter(l => l !== lang)
      } else {
          langs = [...langs, lang]
      }
  }
  async function get_news() {
  // Step 1: get concept URI
  const conceptRes = await fetch(`${WORKER}?path=/api/v1/suggestConceptsFast`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ prefix: topic, lang: 'eng',count: 1})
    // no apiKey here — the worker injects it from the secret
  });
  const concepts = await conceptRes.json();
  const conceptUri = concepts[0]?.uri;


  // Step 2: fetch articles
  const res = await fetch(`${WORKER}?path=/api/v1/article/getArticles`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ conceptUri, articlesCount: count, resultType: 'articles', keyword: topic,  lang: langs.length === 2 ? undefined : langs[0],
    keywordLoc: 'title',
})
  });
  const data = await res.json();
  news = data.articles?.results ?? [];
  loading = false
}

let topic = $state('marcus')
onMount(()=> get_news())

</script>

<main>


<div1><h1>Latest</h1>
  <input
    type="text"
    bind:value={topic}
    placeholder="sökord"
  />
<div class="section-label">Antal artiklar</div>
<div class="btn-group">
    {#each [2, 5, 10, 20] as n}
        <button 
            class="toggle-btn {count === n ? 'active' : ''}"
            onclick={() => { count = n }}>
            {n}
        </button>
    {/each}
</div>

<div class="section-label">Språk</div>
<div class="btn-group">
    {#each [['eng','Engelska'], ['swe','Svenska']] as [code, label]}
        <button 
            class="toggle-btn {langs.includes(code) ? 'active' : ''}"
            onclick={() => toggleLang(code)}>
            {label}
        </button>
    {/each}
  <button onclick={() => get_news()}>Sök</button>
</div>
</div1>
<div2>
  <div3>
  {#if loading}
  <p>laddar...</p>
{:else}
  {#each news as article}
  <article>
  <a href={article.url} target="_blank" class="article-card">
    {#if article.image}
      <img src={article.image} alt={article.title} />
    {/if}
    <div class="article-info">
      <h2>{article.title}</h2>
      <p>{article.body}</p>
      <span>{article.source?.title ?? ''}</span>
    </div>
  </a>
</article>
  {/each}
{/if}
</div3></div2>

</main>

<style>
main{
    display:flex;
    flex-wrap: nowrap;
    width:100vw;
    height:100vh;
    align-items: flex-start;
}
div1{
    display:flex;
    flex-direction:column;
    width:15%;
    height:100%;
    background-color: grey;
    border-radius: 20px;
    justify-content: flex-start;
    align-items: center;
    padding-top:10px;
    gap:8px;
    overflow:hidden;
}
div2{
    display:flex;
    flex-direction:column;
    width:82%;
    background-color: lightgrey;
    border-radius:20px;
    margin-left:2px;
    padding:10px;
}
div3{
  padding:10px;
  border: 1px solid black ;
  border-radius:20px;
  display: flex;
  flex-wrap: wrap;
}

article{
  flex-basis: 400px;
  flex-grow: 1;
  margin:1%;
}
input{
  width:85%;
  padding:8px;
  border-radius:10px;
  border:1px solid black;
  font-size:15px;
  box-sizing:border-box;
}
button{
  padding:8px 14px;
  border-radius:10px;
  background-color:white;
  border: 1px solid black;
  cursor:pointer;
  width:85%;
  font-size:15;
  margin-top:10px;
}
.article-card{
  display:flex;
  flex-wrap:wrap;
  text-decoration:none;
  color:black;
  background:white;
  border-radius:12px;
  overflow:hidden;
  margin-bottom:12px;
  border:1px solid #ddd;
  transition:box-shadow 0.2s;
}

.article-card:hover{
  box-shadow:0 4px 12px rgba(0,0,0,0.15);
}

.article-card img {
  width:100%;
  height:180px;
  object-fit:cover;
}

.article-info{
  padding:10px;
}

.article-info h2{
  font-size:16px;
  margin:0 0 6px 0;
}

.article-info p{
  font-size:13px;
  color:#444;
  margin:0 0 6px 0;
  display:-webkit-box;
  -webkit-line-clamp:3;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.article-info span {
  font-size:11px;
  color:#888;
}
.section-label {
    color: white;
    font-size: 12px;
    font-weight: bold;
    margin-top: 10px;
    align-self: flex-start;
    padding-left: 8px;
}

.btn-group {
    display: flex;
    flex-wrap: wrap;
    gap: 6px;
    justify-content: center;
    width: 90%;
}

.toggle-btn {
    padding: 6px 10px;
    border-radius: 10px;
    border: 1px solid black;
    background-color: white;
    cursor: pointer;
    font-size: 12px;
    flex: 1;
}

.toggle-btn.active {
    background-color: black;
    color: white;
}
</style>