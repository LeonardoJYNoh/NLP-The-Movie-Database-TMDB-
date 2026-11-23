# 🎬 PLN com LangChain, Gemini e TMDB

Projeto prático da disciplina **Processamento de Linguagem Natural (2026-Q3)** – UFABC
Prof. Alexandre Donizeti Alves

[![Abrir no Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/134g75ilHyJ9JCA2z5kOJaalnpmvUaroH?usp=sharing)

## 📌 Sobre o projeto

Este projeto aplica **quatro técnicas de Processamento de Linguagem Natural** usando um **Grande Modelo de Linguagem (LLM)** integrado a uma **API de filmes**, tudo organizado com o framework **LangChain** em um notebook do Google Colab.

A ideia central é a interação entre o LLM e a API:

```
texto do usuário → LLM interpreta (técnica de PLN) → API TMDB busca dados reais → LLM responde em português
```

Assim, o modelo não precisa "inventar" informações sobre filmes: ele interpreta o pedido, e os dados reais vêm do TMDB.

## 👥 Equipe

- Leonardo Jae Yong Noh
- Thalita Kelly Ferreira da Costa
- Victor Martim Nascimento

## 🧰 Tecnologias

| Componente | Escolha |
|---|---|
| Linguagem | Python (Google Colab) |
| Framework | [LangChain](https://docs.langchain.com) (`langchain`, `langchain-google-genai`) |
| LLM | Google Gemini (`gemini-3.5-flash-lite`) – [documentação](https://ai.google.dev/gemini-api/docs) |
| API | [The Movie Database (TMDB)](https://www.themoviedb.org/?language=pt-BR) – [documentação](https://developer.themoviedb.org/docs/getting-started) |
| Outras bibliotecas | `pandas`, `pydantic`, `requests` |

## 🧠 Técnicas de PLN

| # | Técnica | O que o usuário fornece | O que o TMDB traz | Resultado |
|---|---|---|---|---|
| 1 | **Reconhecimento de Entidades Nomeadas (NER)** | Texto livre citando atores e diretores | Filmografia (`/search/person`, `/person/{id}/movie_credits`) | Atores e diretores identificados e seus filmes mais populares |
| 2 | **Perguntas e Respostas (RAG)** | Perguntas sobre um filme | Elenco, direção, duração e sinopse (`/search/movie`, `/movie/{id}`) | Respostas baseadas **somente** nos dados recuperados |
| 3 | **Classificação de Textos** | Pedido livre ("quero algo tenso e sombrio...") | Filmes dos gêneros escolhidos (`/discover/movie`) | Gêneros, emoção desejada e recomendações explicadas |
| 4 | **Análise de Sentimentos + Tradução + Sumarização** | (nenhum) | Avaliações de espectadores em inglês (`/movie/{id}/reviews`) | Sentimento, resumo em português e veredito geral do filme |

### Como cada técnica funciona

**1. NER → filmografia**
O LLM extrai do texto os nomes de atores e diretores (saída estruturada com `Pydantic`). Para cada pessoa, o TMDB devolve os 5 filmes mais populares, como ator ou como diretor.

**2. Perguntas e Respostas (RAG)**
Primeiro recuperamos os dados do filme no TMDB (*retrieval*), depois o LLM responde usando apenas esse contexto (*generation*). Se a informação não estiver no contexto (ex.: bilheteria), o modelo deve dizer que não sabe.

**3. Classificação de textos**
O LLM classifica o pedido do usuário em 1 ou 2 gêneros oficiais do TMDB e identifica a emoção desejada. Com os gêneros, o TMDB devolve filmes populares e o LLM escreve uma frase explicando por que cada um combina com o pedido.

**4. Sentimentos + Tradução + Sumarização**
Para cada avaliação em inglês, o LLM classifica o sentimento (positivo, negativo ou neutro) e resume em uma frase traduzida para o português. Em seguida, uma segunda chain gera o veredito geral do filme a partir desses resumos.

## 🔗 Como o LangChain é usado

Cada etapa é uma *chain*, montada com o operador `|`:

```python
# Resposta em formato fixo (ex.: sentimento só pode ser positivo, negativo ou neutro)
chain = prompt | llm.with_structured_output(Esquema)

# Resposta em texto livre
chain = prompt | llm | StrOutputParser()
```

- `ChatPromptTemplate`: instruções para o modelo, sempre pedindo resposta em português.
- `with_structured_output` + `Pydantic`: garantem respostas em formato previsível.
- `chain.batch(...)`: aplica a chain a vários textos de uma vez.

## 🔑 Como obter as chaves de API

O projeto usa duas APIs. Ambas têm plano gratuito e exigem cadastro.

### 1) Gemini (Google AI Studio)

1. Acesse [aistudio.google.com/apikey](https://aistudio.google.com/apikey) e entre com uma conta Google.
2. Clique em **Create API key** (criar chave de API) e escolha um projeto (ou crie um novo).
3. Copie a chave gerada (começa com `AIza...`) e guarde em local seguro.

> A cota gratuita limita o número de chamadas por minuto e por dia. Se aparecer erro **429**, aguarde um pouco e execute novamente.

### 2) TMDB (The Movie Database)

1. Crie uma conta gratuita em [themoviedb.org](https://www.themoviedb.org/?language=pt-BR) e confirme o e-mail.
2. Acesse [themoviedb.org/settings/api](https://www.themoviedb.org/settings/api) (menu do perfil → **Configurações** → **API**).
3. Solicite uma chave de API (tipo **Developer**) e preencha o formulário com os dados do projeto (finalidade: uso educacional).
4. Na mesma página, copie o **Token de Acesso de Leitura da API** (*API Read Access Token*).

> ⚠️ Use o **token de leitura**, que é longo e começa com `eyJ...`. A **chave de API v3**, curta, não funciona neste projeto, porque o notebook autentica com `Authorization: Bearer <token>`. Se aparecer erro **401**, é esse o motivo.

Os nomes dos menus podem mudar com o tempo; em caso de dúvida, consulte a [documentação do TMDB](https://developer.themoviedb.org/docs/getting-started).

## 🚀 Como usar o notebook

### Passo 1: abrir no Colab

Escolha uma das opções:

- Clique no botão **Abrir no Colab** no topo deste README; ou
- Baixe o arquivo `Projeto_PLN_TMDB.ipynb`, acesse [colab.research.google.com](https://colab.research.google.com) e use **Arquivo → Fazer upload de notebook**.

### Passo 2: guardar as chaves nos Secrets do Colab

1. Na barra lateral esquerda do Colab, clique no ícone de **chave** (**Secrets**).
2. Clique em **Adicionar novo secret** e cadastre os dois:

   | Nome (exatamente assim) | Valor |
   |---|---|
   | `GEMINI` | chave do Gemini |
   | `TMDB` | token de leitura do TMDB |

3. Ative a opção **Acesso ao notebook** (*Notebook access*) de cada secret.

Se preferir não usar Secrets, não tem problema: ao executar, o notebook pede as chaves por digitação.

### Passo 3: executar

Use **Ambiente de execução → Executar tudo**, ou rode célula por célula com `Shift + Enter`, na ordem. O notebook é organizado assim:

| Seção do notebook | O que faz |
|---|---|
| **Biblioteca / Validações** | Instala os pacotes (`!pip install`), faz os imports, lê as chaves, cria o LLM (Gemini) e configura o acesso ao TMDB |
| **Como usamos o LangChain** | Imports do LangChain e do Pydantic |
| **Funções da API (TMDB)** | Define a função `tmdb(...)`, carrega os gêneros e define o filme em `FILME` |
| **1. NER** | Extrai atores e diretores de textos livres e mostra a filmografia de cada um |
| **2. Perguntas e Respostas (RAG)** | Busca os dados do filme e responde perguntas com base neles |
| **3. Classificação de textos** | Classifica pedidos em gêneros e recomenda filmes |
| **4. Sentimentos + Tradução + Sumarização** | Analisa avaliações em inglês e gera o veredito em português |
| **Conclusão** | Resumo das técnicas |

> A primeira célula leva cerca de 1 minuto (instalação). Execute as seções **na ordem**, pois as técnicas reutilizam funções e variáveis definidas antes.

### Passo 4: testar com os seus próprios dados

Depois de rodar, experimente trocar as entradas:

| Onde | O que alterar | Exemplo |
|---|---|---|
| Funções da API | variável `FILME` | `FILME = "Interestelar"` |
| Técnica 1 | lista `textos` | `"Quero ver os filmes do Quentin Tarantino com o Brad Pitt"` |
| Técnica 2 | lista `perguntas` | `"Em que ano o filme foi lançado?"` |
| Técnica 3 | chamadas `recomendar(...)` | `recomendar("Quero um filme de aventura para ver em família")` |

Depois de alterar um valor, execute novamente apenas a célula correspondente (e, se mudou o `FILME`, as células das técnicas 2 e 4).

### Problemas comuns

| Erro | Causa provável | Solução |
|---|---|---|
| `401 Unauthorized` (TMDB) | Foi usada a chave v3 em vez do token de leitura | Use o **Token de Acesso de Leitura** e atualize o secret `TMDB` |
| `429` / *quota exceeded* (Gemini) | Limite da cota gratuita | Aguarde alguns minutos e execute de novo |
| `ModuleNotFoundError` | A célula de instalação não foi executada | Execute a seção **Biblioteca / Validações** do início |
| `NameError` (variável não definida) | Seções executadas fora de ordem | Use **Executar tudo** ou rode as células de cima para baixo |
| `IndexError` na técnica 4 | O filme escolhido não tem avaliações no TMDB | Troque o valor de `FILME` por um filme mais popular |
| Secret não encontrado | Secret com nome diferente ou sem acesso ao notebook | Confira os nomes `GEMINI` e `TMDB` e ative **Acesso ao notebook** |

## 📁 Estrutura do repositório

```
├── Projeto_PLN_TMDB.ipynb   # notebook com todo o projeto
└── README.md                # este arquivo
```

## ⚠️ Limitações

- A busca por nome usa o **primeiro resultado** do TMDB; nomes ambíguos podem retornar a pessoa ou o filme errado.
- As avaliações do TMDB são escritas pelos usuários, em sua maioria em inglês, e algumas são longas (limitamos o tamanho para economizar *tokens*).
- A cota gratuita da API do Gemini pode limitar o número de chamadas por minuto.
- O LLM pode errar; por isso o RAG restringe as respostas ao contexto vindo da API.

## 📚 Referências

- [TMDB API – Getting Started](https://developer.themoviedb.org/docs/getting-started)
- [Google Gemini API](https://ai.google.dev/gemini-api/docs)
- [LangChain](https://docs.langchain.com)

---

*Este produto usa a API do TMDB, mas não é endossado nem certificado pelo TMDB.*
