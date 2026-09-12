# Livros Adaptados para o Cinema

Trabalho 1 (Aquisição de Dados) da disciplina Ciência de Dados - UFAM/IComp.

## Sobre o projeto

Base de dados própria que cruza livros e suas adaptações cinematográficas, combinando avaliações de
leitores (Google Books API) com dados de recepção, produção e desempenho comercial dos filmes
correspondentes (Wikipédia, via scraping, + TMDB API).

**Pergunta motivadora:** o sucesso de um livro entre os leitores se traduz em sucesso do filme que ele
inspira — e o que explica os casos em que isso não acontece?

## Integrantes

- Anna Luisa Antony
- Elaine de Castro Freire
- Manuela Figueira Batista
- Raissa Clara Brasil

## Estrutura do repositório

```
├── notebook_aquisicao.ipynb     # Notebook de coleta, integração e limpeza (reprodutível)
├── dados_brutos/                 # Dados crus de cada fonte, exatamente como coletados
│   ├── wikipedia_html/
│   ├── google_books_json/
│   ├── tmdb_json/
│   └── proveniencia.json         # Registro de onde/quando/como cada dado foi coletado
├── dados_tratados/                # Base final integrada e limpa
│   ├── base_livros_filmes_tratada.csv
│   └── base_livros_filmes_tratada.parquet
└── dataset_card.md                # Ficha descritiva completa da base (fontes, variáveis, limitações, ética)
```

## Fontes de dados

| Fonte | Método | O que fornece |
|---|---|---|
| Wikipédia (Categoria: Livros adaptados para o cinema) | Web scraping (BeautifulSoup) | Títulos das obras e links dos artigos |
| Google Books API | API REST | Autor, ano de publicação, nota média, avaliações, gênero |
| TMDB API | API REST | Dados do filme: lançamento, orçamento, receita, popularidade, nota |

As três fontes são integradas por uma chave de título normalizada (`chave_titulo`). Detalhes completos
sobre a metodologia, dicionário de variáveis, decisões de limpeza e considerações éticas (LGPD,
licenças, `robots.txt`) estão em [`dataset_card.md`](./dataset_card.md).

## Como reproduzir a coleta

1. Abra `notebook_aquisicao.ipynb` no Google Colab.
2. Execute todas as células em sequência (Ambiente de execução → Executar tudo).
3. Quando solicitado, informe:
   - a chave da Google Books API (opcional. funciona sem, com limite de requisições menor);
   - o token de leitura (v4, Bearer) da TMDB API (obrigatório).
4. A coleta completa das três fontes leva entre 25 e 40 minutos, dependendo da estabilidade das APIs.

## Licenciamento e uso ético dos dados

- **Wikipédia:** conteúdo sob CC BY-SA 4.0, com atribuição preservada na coluna `url_artigo`.
- **Google Books API:** uso conforme os Termos de Serviço das APIs do Google, para fins acadêmicos.
- **TMDB API:** uso com atribuição obrigatória ("This product uses the TMDB API but is not endorsed
  or certified by TMDB"), conforme seus Termos de Serviço.
- Não há dados pessoais sensíveis na base (ver seção de LGPD em `dataset_card.md`).
