# Livros Adaptados para o Cinema

Trabalho 1 (Aquisição de Dados) da disciplina Ciência de Dados, UFAM/IComp.

## Sobre o projeto

Base de dados própria que cruza livros e suas adaptações cinematográficas, combinando pares
livro-filme extraídos diretamente da Wikipédia com avaliações de leitores (Google Books API) e
dados de recepção, produção e desempenho comercial dos filmes (TMDB API).

**Pergunta motivadora:** o sucesso de um livro entre os leitores se traduz em sucesso do filme que
ele inspira, e o que explica os casos em que isso não acontece?

## Integrantes

- Anna Luisa Antony
- Elaine de Castro Freire
- Manuela Figueira Batista
- Raissa Clara Brasil

## Estrutura do repositório

```
├── notebook_aquisicao.ipynb      # Notebook de coleta, integração e limpeza (reprodutível)
├── dados_brutos.zip               # Dados crus de cada fonte, exatamente como coletados
│   ├── wikipedia_html/            # HTML bruto das 4 páginas de lista da Wikipédia
│   ├── wikipedia_listas_pares.csv # Pares livro-filme extraídos, sem limpeza
│   ├── google_books_json/         # Resposta bruta de cada consulta ao Google Books
│   ├── tmdb_json/                 # Resposta bruta de cada consulta à TMDB
│   ├── tmdb_resultados.csv        # Consolidado bruto da TMDB
│   └── proveniencia.json          # Registro de onde/quando/como cada dado foi coletado
├── dados_tratados.zip              # Base final integrada e limpa
│   ├── base_livros_filmes_tratada.csv
│   ├── base_livros_filmes_tratada.parquet
│   └── dicionario_variaveis.csv
└── dataset_card.md                 # Ficha descritiva completa da base
```

## Fontes de dados

| Fonte | Método | O que fornece |
|---|---|---|
| Wikipédia (List of fiction works made into feature films, 4 páginas alfabéticas) | Web scraping (`pandas.read_html` sobre o HTML salvo) | Pares livro-filme prontos: título e ano do livro, título e ano do filme |
| Google Books API | API REST | Autor, nota média, número de avaliações, categoria, editora do livro |
| TMDB API | API REST, buscado por título e ano específicos de cada par | Data de lançamento, orçamento, receita, popularidade, nota do filme |

A Wikipédia já fornece o par livro-filme na mesma linha da tabela, o que elimina a necessidade de
adivinhar a correspondência entre livro e filme só pelo título. A TMDB é consultada com o título e
o ano exatos indicados pela Wikípedia para cada par, e um segundo filtro invalida os poucos casos em
que a TMDB retornou uma adaptação de ano muito diferente do esperado (por exemplo, uma versão antiga
do mesmo livro em vez do remake procurado). Com isso, 96,5% dos pares mantêm um match considerado
válido. Detalhes completos sobre a metodologia, dicionário de variáveis, decisões de limpeza e
considerações éticas (LGPD, licenças, `robots.txt`) estão em
[`dataset_card.md`](./dataset_card.md).

## Volume da base

A Wikípedia lista 3.821 pares livro-filme ao todo. Para manter o volume de chamadas às APIs viável
e priorizar filmes com mais dados disponíveis na TMDB, a base foi recortada para filmes lançados a
partir do ano 2000, resultando em **911 linhas e 26 colunas** na base tratada final.

## Como reproduzir a coleta

1. Abra o notebook no Google Colab.
2. Execute todas as células em sequência (Ambiente de execução → Executar tudo).
3. Quando solicitado, informe:
   - a chave da Google Books API (opcional, funciona sem, com limite de requisições menor);
   - o token de leitura (v4, Bearer) da TMDB API (obrigatório).
4. A coleta completa das três fontes leva algumas dezenas de minutos, dependendo da estabilidade
   das APIs.

## Licenciamento e uso ético dos dados

- **Wikipédia:** conteúdo sob CC BY-SA 4.0, com atribuição preservada na coluna `url_fonte`.
- **Google Books API:** uso conforme os Termos de Serviço das APIs do Google, para fins acadêmicos.
- **TMDB API:** uso com atribuição obrigatória ("This product uses the TMDB API but is not endorsed
  or certified by TMDB"), conforme seus Termos de Serviço.
- Não há dados pessoais sensíveis na base (ver seção de LGPD em `dataset_card.md`). O registro de
  proveniência tem qualquer chave de API usada durante a coleta mascarada antes da publicação.
