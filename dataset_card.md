# Dataset Card - Trabalho 1: Aquisição de Dados

## A.1 Identificação

**Nome da base:** Livros e suas Adaptações para o Cinema

**Grupo / integrantes:**
- Anna Luisa Antony Afonso
- Elaine de Castro Freire
- Manuela Figueira Batista
- Raissa Clara Teixeira Brasil

**Tema e pergunta motivadora:**

Uma base que cruza obras literárias com suas adaptações cinematográficas, combinando a avaliação dos
leitores sobre o livro (Google Books API) com os dados de produção, recepção e desempenho comercial
do filme correspondente (TMDB API), a partir de pares livro→filme extraídos da Wikipédia (scraping).

*Pergunta motivadora:* O sucesso de um livro entre os leitores se traduz em sucesso do filme que ele
inspira - e o que explica os casos em que isso não acontece?

Desdobramentos que a base permite investigar:
- Livros com mais avaliações (`n_avaliacoes_livro`) geram filmes com maior bilheteria (`receita`) ou
  popularidade (`popularidade`)?
- A nota do livro (`nota_media_livro`) tem relação com a nota do filme (`media_votos_filme`)?
- Certos gêneros (`generos_filme`) convertem melhor sua base de leitores em público de cinema?


**Data da coleta final:** 18 de setembro de 2026

---

## A.2 Fontes e proveniência

**Fonte 1 - nome e URL:** Wikipédia (En) - listas de obras de ficção adaptadas para o cinema
`https://en.wikipedia.org/wiki/List_of_fiction_works_made_into_feature_films`
(sub-listas alfabéticas: 0–9/A–C, D–J, K–R, S–Z)

**Fonte 1 - método:** Web scraping das tabelas com `requests` + `pandas.read_html`. Cada linha das
tabelas traz, já pareados por editores da Wikipédia, o livro (`Título (ano), Autor`) e o filme
(`Título do filme (ano)`); esses campos foram separados por expressão regular. O HTML bruto de cada
página foi salvo. `robots.txt` verificado programaticamente (função `raspar`, `can_fetch = True`) antes
da coleta. Pausa de 2s entre requisições.

**Fonte 1 - licença/termos:** conteúdo da Wikipédia sob CC BY-SA 4.0, que permite uso e redistribuição
(inclusive acadêmica) mediante atribuição. Extraímos apenas dados factuais das listas (títulos, anos,
autores), não o texto dos artigos. Atribuição preservada na coluna `url_fonte`.

**Fonte 2 - nome e URL:** Google Books API
`https://www.googleapis.com/books/v1/volumes`

**Fonte 2 - método:** API REST, uma requisição por livro único (`q=intitle:{título} inauthor:{autor}`,
`langRestrict=en`), reaproveitando título e autor extraídos da Wikipédia. Buscar por título + autor
reduz muito o risco de casar com outro livro. Extraídos nota média, nº de avaliações, categorias,
idioma e editora. JSON bruto salvo por consulta. Retry automático em 429/503. Pausa de 2s.

**Fonte 2 - licença/termos:** uso regido pelos Termos de Serviço das APIs do Google; permitido para
fins acadêmicos, vedada a redistribuição dos dados brutos como produto concorrente.

**Fonte 3 - nome e URL:** TMDB (The Movie Database) API
`https://api.themoviedb.org/3/search/movie`, `/movie/{id}` e `/movie/{id}/keywords`

**Fonte 3 - método:** API REST autenticada (Bearer token v4). Para cada par: busca do filme por
**título + ano** (`primary_release_year`) com fallback sem ano quando necessário; detalhes do filme
(orçamento, receita, duração, gêneros, votos); e keywords, usadas para confirmar a adaptação via
`based on novel or book` (keyword 818). JSON bruto salvo por chamada. Retry em 429 e erros de conexão.
Pausa de 2s.

**Fonte 3 - licença/termos:** a TMDB exige a atribuição *"This product uses the TMDB API but is not
endorsed or certified by TMDB"* e proíbe uso que viole seus Termos de Serviço. Uso acadêmico permitido.

**Chave de integração:** a Wikipédia é o esqueleto (um par livro→filme por linha). Há **duas chaves**,
porque as fontes descrevem coisas diferentes:
- **Wikipédia × Google Books:** casadas pelo **livro** - `chave_livro` = título do livro + autor,
  normalizados (minúsculas, sem acentos, sem pontuação, espaços colapsados). O casamento tolera
  divergências de grafia via essa normalização.
- **Wikipédia × TMDB:** casadas pelo **filme**, por **posição** (índice) - o `df_tmdb` foi construído
  linha a linha a partir do esqueleto, então a linha *i* de um corresponde à linha *i* do outro,
  casamento exato e sem risco de erro textual.

O tratamento de divergências de casamento está descrito em A.5.

---

## A.3 Dicionário de variáveis

*(Também exportado em `dados_tratados/dicionario_variaveis.csv`.)*

| Variável | Tipo | Descrição | Unidade |
|---|---|---|---|
| `titulo_livro` | texto | Título do livro (Wikipédia, limpo) | - |
| `autor` | categórica | Autor(es) do livro (Wikipédia) | - |
| `ano_publicacao_livro` | numérica discreta | Ano da 1ª edição do livro (Wikipédia) | ano |
| `titulo_filme` | texto | Título do filme (Wikipédia, limpo) | - |
| `ano_lancamento_filme` | numérica discreta | Ano de lançamento do filme (TMDB) | ano |
| `eh_serie` | booleana | Se a obra é uma série/coletânea agrupada na fonte | - |
| `url_fonte` | texto | URL da lista da Wikipédia (atribuição) | - |
| `nota_media_livro` | numérica contínua | Nota média do livro (Google Books) | 0–5 |
| `n_avaliacoes_livro` | numérica discreta | Nº de avaliações do livro (Google Books) | contagem |
| `categorias_livro` | categórica | Categorias do livro (Google Books) | - |
| `editora` | categórica | Editora do livro (Google Books) | - |
| `idioma_livro` | categórica | Idioma da edição do livro (Google Books) | ISO |
| `tem_avaliacoes_livro` | booleana | Se o livro tem avaliação no Google Books | - |
| `tmdb_id` | numérica discreta | Identificador do filme na TMDB | - |
| `data_lancamento_filme` | data/hora | Data de lançamento do filme (TMDB) | data |
| `idioma_filme` | categórica | Idioma original do filme (TMDB) | ISO |
| `generos_filme` | categórica | Gêneros do filme (TMDB) | - |
| `duracao_min` | numérica contínua | Duração do filme (0 → NaN) | minutos |
| `orcamento` | numérica contínua | Orçamento do filme (0 → NaN) | USD nominal |
| `receita` | numérica contínua | Receita/bilheteria do filme (0 → NaN) | USD nominal |
| `popularidade` | numérica contínua | Índice de popularidade (TMDB) | score TMDB |
| `media_votos_filme` | numérica contínua | Nota média do filme (TMDB) | 0–10 |
| `contagem_votos_filme` | numérica discreta | Nº de votos do filme (TMDB) | contagem |
| `tem_match_tmdb` | booleana | Se o filme foi encontrado (e validado) na TMDB | - |
| `eh_adaptacao_livro` | booleana | Se a TMDB confirma o filme como adaptação de livro (keyword 818) | - |
| `retorno_financeiro` | numérica contínua | (receita - orçamento) / orçamento, se orçamento válido | razão |

---

## A.4 Volume e granularidade

**Número de linhas / colunas:** 911 linhas × 26 colunas (base tratada).

**O que representa uma linha:** um par **livro → filme** - uma obra literária e uma adaptação
cinematográfica específica dela. Como um mesmo livro pode ter várias adaptações (ex.: Pinóquio), um
livro pode aparecer em mais de uma linha, cada uma com um filme distinto. A unidade de observação é,
portanto, o *par*, não o livro nem o filme isoladamente.

**Cobertura:** pares extraídos das listas de ficção adaptada da Wikipédia em inglês, com **recorte
temporal: filmes lançados a partir do ano 2000**. O scraping rendeu 3821 pares; o recorte para ≥2000
(911 pares) foi aplicado por dois motivos: viabilizar a coleta via API em tempo reprodutível (enriquecer
os 3821 exigiria ~15 mil chamadas) e porque filmes recentes têm cobertura muito melhor de dados na TMDB.
A base representa, assim, **adaptações recentes**, não adaptações em geral.

---

## A.5 Limitações e decisões

**Dados descartados:**
- **Recorte temporal:** dos 3821 pares raspados, mantivemos os 911 com filme lançado a partir de 2000.
- **Duplicatas de livro no Google Books:** consultamos livros únicos (522) e removemos chave de livro
  repetida antes do join. Na Wikipédia, livros repetidos (várias adaptações do mesmo livro) são
  legítimos e mantidos.
- 

**Tratamento de divergências de casamento (eixo central de qualidade):**
- **48 filmes recuperados:** a primeira busca na TMDB falhou em 64 filmes porque o título vinha com
  trechos de tradução grudados (ex.: "White Fang (French: Croc-Blanc)"). Limpando o título e
  reconsultando, 48 foram recuperados, elevando o match de 93,0% para 98,2%.
- **16 matches invalidados:** a busca com fallback sem ano às vezes casou com uma adaptação **antiga**
  do mesmo livro (ex.: "The Picture of Dorian Gray" de 1945 em vez da versão recente). Quando o ano do
  filme na TMDB diverge em mais de 5 anos do indicado pela Wikipédia, tratamos como obra diferente e
  **anulamos** os dados da TMDB dessas linhas. A taxa de match válido final é **96,5%**.

**Lacunas conhecidas:**
- **Cobertura de avaliação de livro baixa:** apenas **16,7%** das linhas (152 de 911) têm
  `nota_media_livro`. É uma limitação estrutural do Google Books (mesmo buscando por título + autor,
  a maioria dos livros não tem nota agregada), não do método. Análises que dependam da nota do livro
  ficam restritas a esse subconjunto.
- **Cobertura financeira parcial:** `orcamento` e `receita` ausentes em 45% dos filmes (0 na TMDB
  tratado como "não divulgado" → NaN). `retorno_financeiro` calculável em 48% das linhas.
- **Confirmação por keyword parcial:** apenas **46,2%** dos filmes têm a keyword 818 na TMDB. Como
  ela é preenchida por voluntários, sua ausência não significa que o filme não seja adaptação; por
  isso a keyword foi usada como **reforço positivo** de qualidade, não como filtro eliminatório.
- **Obras em série agrupadas:** séries (Harry Potter, Twilight, Hunger Games, etc.) aparecem agrupadas em
  uma entrada na Wikipédia; nessas linhas (sinalizadas por `eh_serie`), a nota do livro pode
  representar a série ou o primeiro volume, não o volume exato de cada filme.
- **~3,5% dos filmes sem match** na TMDB (sinalizados por `tem_match_tmdb = False`).

**Decisões de limpeza relevantes:**
- Chaves de integração normalizadas (minúsculas, sem acento/pontuação) e títulos limpos de trechos de
  tradução (`(French: ...)`, `(German: ...)`).
- `orcamento`/`receita`/`duracao_min` iguais a 0 na TMDB tratados como "não informado" (NaN).
- **Outliers financeiros detectados (IQR) mas não capados:** um blockbuster real é dado relevante para
  a pergunta motivadora. Os maiores retornos remanescentes são casos reais de filmes de baixo orçamento
  com alta bilheteria (ex.: The Invisible Man, Winnie-the-Pooh: Blood and Honey).
- **`retorno_financeiro` com piso de orçamento:** exigimos orçamento ≥ US$ 100 mil para calcular o
  retorno, descartando orçamentos implausíveis (poucos milhares de dólares, prováveis erros de cadastro
  na TMDB) que geravam retornos absurdos.
- Categóricas textuais ausentes preenchidas com o rótulo "Não informado".
- Valores financeiros em USD **nominais**, sem correção por inflação ou câmbio.

---

## A.6 Considerações éticas

**Contém dados pessoais?** Não, no sentido da LGPD (Lei nº 13.709/2018). A base contém nomes de
autores de livros, mas referem-se a figuras públicas em capacidade profissional (autoria de obra
publicada), não a indivíduos identificados em contexto privado. Não há dados sensíveis, comportamentais
ou de identificação de pessoas físicas privadas - apenas notas e contagens agregadas por obra. Nenhuma
técnica de anonimização foi necessária.

**Restrições de uso/redistribuição:** uso acadêmico permitido nas três fontes. A redistribuição da base
tratada deve manter a atribuição à Wikipédia (CC BY-SA 4.0) e à TMDB (atribuição obrigatória exigida
pelos termos da API), e não deve ser usada para criar um produto concorrente ao Google Books ou à TMDB.

**`robots.txt` verificado?** Sim, para a Wikipédia (única fonte raspada via HTML) - a função `raspar`
confirmou `can_fetch = True` para as URLs das listas antes da coleta, registrando o resultado na
proveniência. As APIs (Google Books e TMDB) não são cobertas por `robots.txt`, sendo regidas por seus
próprios Termos de Serviço, verificados acima. Boas práticas de coleta aplicadas: pausa de 2s entre
requisições, User-Agent identificando o projeto, retry em erros de cota, e coleta apenas dos campos
necessários.
