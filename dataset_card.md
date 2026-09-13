# Dataset Card - Trabalho 1: Aquisição de Dados

## A.1 Identificação

**Nome da base:** Livros Adaptados para o Cinema

**Grupo / integrantes:**
- Anna Luisa Antony
- Elaine de Castro Freire
- Manuela Figueira Batista
- Raissa Clara Brasil

**Tema e pergunta motivadora:**

Uma base que cruza livros e suas adaptações cinematográficas, combinando avaliações de leitores
(Google Books API) com dados de recepção, produção e desempenho comercial dos filmes correspondentes
(Wikipédia, via scraping, + TMDB API).

*Pergunta motivadora:* O sucesso de um livro entre os leitores se traduz em sucesso do filme que ele
inspira - e o que explica os casos em que isso não acontece?

Desdobramentos que a base permite investigar:
- Livros com mais avaliações (`n_avaliacoes`) geram filmes com maior bilheteria ou popularidade?
- A nota média do livro (`nota_media`) tem relação com o desempenho comercial/crítico do filme
  (`media_votos`, `receita`)?
- O tempo entre a publicação do livro e o lançamento do filme (`anos_ate_adaptacao`) influencia esse
  "aproveitamento" do sucesso original?
- Certos gêneros literários convertem melhor sua base de leitores em público de cinema do que outros?
- Existem "ondas" de adaptações - décadas em que certos gêneros ou autores foram mais explorados pelo
  cinema?

**Data da coleta:** 11 de setembro de 2026

---

## A.2 Fontes e proveniência

**Fonte 1 - nome e URL:** Wikipédia (Pt) - Categoria: Livros adaptados para o cinema
`https://pt.wikipedia.org/wiki/Categoria:Livros_adaptados_para_o_cinema`

**Fonte 1 - método:** Web scraping (requests + BeautifulSoup), com paginação automática seguindo o
link "página seguinte" da categoria. Extraído: título da obra (`titulo_wikipedia_sujo`) e URL do
artigo (`url_artigo`). `robots.txt` verificado e permite o acesso (`can_fetch` = True). Pausa de 2s
entre requisições de página.

**Fonte 1 - licença/termos:** conteúdo textual da Wikipédia sob CC BY-SA 4.0, que permite uso e
redistribuição (inclusive acadêmica) mediante atribuição. Extraímos apenas dados factuais (títulos e
links), preservando a atribuição na coluna `url_artigo`.

**Fonte 2 - nome e URL:** Google Books API
`https://www.googleapis.com/books/v1/volumes`

**Fonte 2 - método:** API REST, uma requisição por título (`q=intitle:{titulo}`), reaproveitando os
424 títulos extraídos da Wikipédia. Resposta JSON bruta salva por título. Retry automático em caso de
erro 429 (cota excedida). Pausa de 2s entre requisições.

**Fonte 2 - licença/termos:** uso regido pelos Termos de Serviço das APIs do Google; permitido para
fins acadêmicos e de pesquisa, vedada a redistribuição dos dados brutos como produto concorrente.

**Fonte 3 - nome e URL:** TMDB (The Movie Database) API
`https://api.themoviedb.org/3/search/movie` e `https://api.themoviedb.org/3/movie/{id}`

**Fonte 3 - método:** API REST autenticada (Bearer token v4), duas chamadas por título - busca
(`/search/movie`) seguida de detalhes (`/movie/{id}`) do candidato mais relevante. Pausa de 2s entre
requisições.

**Fonte 3 - licença/termos:** TMDB exige atribuição ("This product uses the TMDB API but is not
endorsed or certified by TMDB") e proíbe uso que viole seus Termos de Serviço (ex.: revenda dos dados
brutos como produto concorrente). Uso acadêmico é permitido.

**Chave de integração:** `chave_titulo` - título normalizado (minúsculas, sem acentos, sem pontuação,
sem espaços duplicados), derivado de `titulo_wikipedia` e aplicado às três fontes. O join é feito com
`how="left"` a partir da Wikipédia (que funciona como "esqueleto" da base - todo livro da categoria),
enriquecida pelas outras duas fontes. Duplicatas de chave (ex.: reedições do mesmo livro) são reduzidas
à primeira ocorrência.

---

## A.3 Dicionário de variáveis

| Variável | Tipo | Descrição | Unidade |
|---|---|---|---|
| `chave_titulo` | texto | Chave normalizada de integração entre as três fontes | - |
| `titulo_wikipedia` | texto | Título do livro, conforme Wikipédia | - |
| `url_artigo` | texto | URL do artigo da Wikipédia sobre a obra | - |
| `titulo_google_books` | texto | Título do livro conforme Google Books | - |
| `autor` | texto | Autor(es) do livro | - |
| `ano_publicacao_livro` | numérica discreta | Ano de publicação do livro | ano |
| `nota_media` | numérica contínua | Nota média do livro no Google Books | 0–5 |
| `n_avaliacoes` | numérica discreta | Número de avaliações do livro no Google Books | contagem |
| `categorias` | categórica | Categoria/gênero do livro | - |
| `idioma` | categórica | Idioma da edição do livro | código ISO |
| `editora` | categórica | Editora do livro | - |
| `tem_avaliacoes_google_books` | booleana | Se o livro tem nota registrada no Google Books | - |
| `titulo_tmdb` | texto | Título do filme conforme TMDB | - |
| `data_lancamento` | data/hora | Data de lançamento do filme | AAAA-MM-DD |
| `ano_lancamento_filme` | numérica discreta | Ano de lançamento do filme | ano |
| `idioma_original` | categórica | Idioma original do filme | código ISO |
| `generos` | categórica | Gênero(s) do filme | - |
| `duracao_min` | numérica contínua | Duração do filme | minutos |
| `orcamento` | numérica contínua | Orçamento de produção do filme | USD |
| `receita` | numérica contínua | Receita de bilheteria do filme | USD |
| `popularidade` | numérica contínua | Índice de popularidade/engajamento na TMDB | índice TMDB |
| `media_votos` | numérica contínua | Nota média do filme no TMDB | 0–10 |
| `contagem_votos` | numérica discreta | Número de votos do filme no TMDB | contagem |
| `tem_match_tmdb` | booleana | Se o título encontrou um filme correspondente na TMDB | - |
| `sim_titulo_gb` | numérica contínua (derivada) | Similaridade textual entre título da Wikipédia e o retornado pelo Google Books | 0 a 1 |
| `sim_titulo_tmdb` | numérica contínua (derivada) | Similaridade textual entre título da Wikipédia e o retornado pela TMDB | 0 a 1 |
| `ano_coerente` | booleana (derivada) | Se o ano do filme é igual ou posterior ao ano do livro, sinalizando inversões suspeitas | - |
| `match_confiavel` | booleana (derivada) | Sinaliza casamentos considerados confiáveis, combinando similaridade de título e coerência de ano | - |
| `anos_ate_adaptacao` | numérica discreta (derivada) | Anos entre publicação do livro e lançamento do filme | anos |
| `retorno_financeiro` | numérica contínua (derivada) | (receita menos orçamento) dividido por orçamento | proporção |

---

## A.4 Volume e granularidade

**Número de linhas / colunas:** 424 linhas (livros da categoria da Wikipédia) × 30 colunas na base
integrada. Nenhuma linha foi perdida na deduplicação de chave (424 brutas -> 424 após dedup, nas três
fontes).

**O que representa uma linha:** um livro pertencente à categoria "Livros adaptados para o cinema" da
Wikipédia em português, enriquecido com dados do livro (Google Books) e do filme correspondente (TMDB)
quando encontrados.

**Cobertura:** todas as obras listadas na categoria da Wikipédia no momento da coleta - não é uma
amostra aleatória, é a totalidade da categoria (um "censo" da categoria). Sem recorte temporal
definido: cobre obras de diferentes décadas, idiomas e nacionalidades.

---

## A.5 Limitações e decisões

**Dados descartados:** duplicatas completas (linha idêntica em todas as colunas) e duplicatas de
`chave_titulo` (mantida apenas a primeira ocorrência - trata reedições do mesmo livro como a mesma
obra).

**Lacunas conhecidas:**
- Taxa de match com a TMDB: 85,8% (364 de 424 títulos). Taxa de cobertura de avaliação no Google
  Books: apenas 19,6% (83 de 424), o que limita bastante análises futuras que dependam de
  `nota_media`.
- O `merge` por título sozinho não garante que o livro e o filme casados sejam de fato a mesma obra
  (risco de homônimos). Para tratar isso, foram criadas colunas de similaridade textual (`sim_titulo_gb`,
  `sim_titulo_tmdb`) e de coerência temporal (`ano_coerente`), combinadas na flag `match_confiavel`.
  Ao aplicar esse critério, 189 das 424 linhas apresentaram `anos_ate_adaptacao` fora de uma faixa
  plausível (filme antes do livro, ou intervalo maior que 120 anos), e apenas 21 dessas 189 foram
  consideradas `match_confiavel = True`. As linhas suspeitas não foram removidas, apenas sinalizadas,
  para preservar o volume da base; análises futuras devem filtrar por `match_confiavel` quando a
  precisão do casamento for crítica.
- Uma causa identificada para parte dessas incoerências é que a Google Books API tende a retornar,
  como primeiro resultado, a edição mais popular ou mais recente do livro (reedições, traduções),
  não necessariamente a primeira edição publicada. Isso distorce `ano_publicacao_livro` para cima em
  parte das linhas e, por consequência, distorce `anos_ate_adaptacao`, que depende desse campo. Esse
  comportamento da API não foi contornado nesta fase (exigiria reprocessar a lista completa de
  candidatos já salva em `dados_brutos/google_books_json/`, escolhendo a edição de menor
  `publishedDate` em vez do primeiro resultado), ficando registrado como melhoria possível para as
  fases futuras do projeto.
- 9 linhas apresentaram duração de filme (`duracao_min`) fora da faixa plausível (1 a 600 minutos),
  também não removidas, apenas não corrigidas.
- Nem todo livro tem correspondência na TMDB (sinalizado por `tem_match_tmdb = False`) - títulos muito
  antigos/obscuros ou com nome muito divergente entre fontes podem não ser encontrados.
- `orcamento`/`receita` iguais a 0 na TMDB foram tratados como "não informado" (NaN), pois a ausência
  desse dado é comum e não significa valor real igual a zero.

**Decisões de limpeza relevantes:**
- Chave de integração normalizada (minúsculas, sem acento/pontuação) para tolerar pequenas divergências
  de grafia entre as fontes.
- Anos extraídos de campos de data em formatos mistos (ex.: `"1999"`, `"1999-05-01"`) para permitir
  comparações numéricas.
- Outliers financeiros (bilheteria/orçamento) foram **detectados** pelo método IQR, mas **não
  removidos/capados** - um blockbuster real é informação relevante para a pergunta motivadora, não um
  erro de coleta.
- Categóricas textuais ausentes preenchidas com o rótulo explícito "Não informado", em vez de
  descartar a linha.

---

## A.6 Considerações éticas

**Contém dados pessoais?** Não, no sentido da LGPD (Lei nº 13.709/2018). A base contém nomes de
autores de livros, mas esses nomes referem-se a figuras públicas atuando em capacidade profissional
(autoria de obra publicada), não a indivíduos identificados incidentalmente em contexto privado. Não
há dados sensíveis, comportamentais ou de identificação de pessoas físicas privadas. Nenhuma técnica
de anonimização foi necessária além do escopo natural do projeto (não foram coletados dados de
usuários, avaliadores individuais ou compradores - apenas notas agregadas por obra).

**Restrições de uso/redistribuição:** uso acadêmico permitido nas três fontes. Redistribuição da base
tratada deve manter a atribuição à Wikipédia (CC BY-SA 4.0) e à TMDB (atribuição obrigatória exigida
pelos termos da API); não deve ser usada para criar um produto concorrente ao Google Books ou à TMDB.

**`robots.txt` verificado?** Sim, para a Wikipédia (única fonte raspada via HTML) - `can_fetch`
retornou `True` para a URL da categoria coletada. As APIs (Google Books e TMDB) não são cobertas por
`robots.txt`, pois são interfaces de programação com termos de uso próprios, verificados separadamente
acima.
