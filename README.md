# Chat-inteligente
# 🤖 Chat Inteligente (SAC + Catálogo)

Este projeto é um **chat inteligente com dois agentes especializados**, exposto via **API RESTful**.  
Ele foi desenvolvido como parte de um desafio prático para vaga de **Cientista de Dados**.

---

## 🚀 Visão Geral

A aplicação é composta por **dois sub-agentes**:

1. **Agente SAC (RAG)**  
   - Responde perguntas sobre políticas da empresa (trocas, devoluções, status de pedido).  
   - Base de conhecimento em `dados-sac.md`.  
   - Implementado com **LangChain** + **FAISS** + **SentenceTransformers** + **Qwen2.5-1.5B Instruct**.

2. **Agente Catálogo**  
   - Responde consultas sobre produtos.  
   - Base de produtos em `dados-produtos.json`.  
   - Implementado com **LangChain Tools** para busca no catálogo.

📦 O sistema mantém o **histórico de conversas em SQLite** (`chat_history.db`) para simular diálogos contínuos.  


### **Passos para Instalação**

   1.  **Clone o repositório:**
    ```bash
    git clone <URL_DO_SEU_REPOSITORIO>
    cd <NOME_DO_DIRETORIO>
    ```

2.  **Crie e ative um ambiente virtual (recomendado):**
    ```bash
    python -m venv venv
    # No Windows
    venv\Scripts\activate
    # No macOS/Linux
    source venv/bin/activate
    ```

3.  **Instale as dependências:**
    ```bash
    pip install -r requirements.txt
    ```

4.  **Prepare os dados:**
    Certifique-se de que os arquivos `dados-sac.md` e `dados-produtos.json` estejam no diretório raiz do projeto.
### Como o modelo armazena e utiliza informações
🗄️ Armazenamento de histórico de conversas

O histórico é salvo em um banco SQLite local (chat_history.db).

Existe uma tabela chamada messages com os campos:

id (chave primária)

agent (tipo de agente: sac ou catalog)

user (usuário que enviou a mensagem)

message (conteúdo da mensagem)

timestamp (quando foi registrada)

Cada interação (do usuário ou do bot) é registrada com save_message(), permitindo reconstruir todo o histórico da conversa com a função get_history().

👉 Isso garante persistência das conversas, o que é essencial para auditar, recuperar e analisar interações.

### 📦 Estratégia por agentes

O projeto implementa dois agentes distintos:

Agente Catálogo (CatalogAgent)

Carrega os dados de produtos a partir de um arquivo JSON (dados-produtos.json).

Transforma o JSON em um DataFrame Pandas para buscas eficientes.

Permite consultar produtos pelo nome ou parte do título, retornando os resultados filtrados.

Agente SAC (Atendimento ao Cliente)

Baseado em RAG (Retrieval-Augmented Generation).

Lê os documentos de suporte (dados-sac.md).

Divide o texto em chunks (pedaços de ~500 caracteres, com sobreposição de 50).

Cria embeddings vetoriais usando sentence-transformers/all-MiniLM-L6-v2.

Indexa os vetores com FAISS, permitindo busca semântica.

Usa o modelo Qwen2.5-1.5B-Instruct (via HuggingFacePipeline) para gerar respostas contextualizadas.

👉 Essa estratégia separa a lógica de busca de produtos estruturados (catálogo) da consulta semântica a documentos (SAC).

### Uso de vetores e embeddings

O texto dos documentos de SAC é convertido em representações vetoriais (embeddings).

Esses vetores são armazenados no índice FAISS, que permite encontrar os trechos mais relevantes em tempo real.

Quando o usuário faz uma pergunta, o modelo:

Busca os vetores mais próximos no FAISS (documentos relacionados).

Envia esses trechos de contexto junto com a pergunta para o modelo Qwen.

Gera uma resposta final fundamentada no material encontrado.

👉 Isso é a essência do RAG: o modelo não precisa "memorizar tudo", mas busca nas bases vetoriais e responde de forma atualizada.

### 🤔 Por que essas estratégias?

SQLite para histórico → leve, simples e persistente; ótimo para protótipos e aplicações pequenas.

Pandas no catálogo → eficiente para buscas estruturadas em dados tabulares (produtos, preços, descrições).

FAISS + Embeddings no SAC → necessário para buscas semânticas em texto não estruturado.

Divisão em chunks → melhora a cobertura e precisão da busca, evitando contextos muito grandes ou muito pequenos.

RAG com LLM → garante respostas mais relevantes e fundamentadas, em vez de depender apenas de geração livre do modelo.

### **Passos para Execução**

Para executar a API localmente, utilize o `uvicorn`:
```bash
uvicorn main:app --reload---

## 🛠️ Tecnologias Utilizadas

- [Python 3.10+](https://www.python.org/)  
- [FastAPI](https://fastapi.tiangolo.com/) → API RESTful  
- [LangChain](https://www.langchain.com/) → framework de agentes  
- [SentenceTransformers](https://www.sbert.net/) → embeddings  
- [FAISS](https://github.com/facebookresearch/faiss) → busca vetorial  
- [Transformers (HuggingFace)](https://huggingface.co/transformers/) → LLM (Qwen2.5-1.5B Instruct)  
- [SQLite](https://www.sqlite.org/) → persistência do histórico  
- [ngrok](https://ngrok.com/) → exposição da API no Colab  

5  🛠️ Alguns questinamentos
Quis  mostrar que sei usar LLM open source (sem custo de API), pora  rodar um modelo menor da família Qwen no Colab, como o qwen2.5-1.5b, que roda bem em GPU T4 (Colab free).por questão de redução de custo, e que o modelo opera de forma satisfatória

## 🔧 Possíveis Melhorias no Projeto
### 1. Infraestrutura do Banco de Dados

Atualmente, o histórico é salvo em SQLite (bom para protótipo).

Em produção → poderia ser substituído por PostgreSQL ou MySQL, permitindo maior escalabilidade, consultas complexas e suporte a múltiplos usuários.

### 2. Melhoria no Agente de Catálogo

Hoje o agente de catálogo faz busca simples com contains (string match).

Poderia evoluir para:

Busca semântica usando embeddings (similar ao SAC).

Filtros por categoria/preço 

Ordenação por relevância.

### 3. Melhoria no Agente SAC (RAG)

O modelo usado é Qwen2.5-1.5B (bom para protótipo, mas limitado em contexto).

Melhorias possíveis:

Testar modelos maiores (Qwen 7B, Mistral 7B, Llama 3) para respostas mais precisas.

Implementar re-ranker para melhorar a qualidade da recuperação de contexto.

Guardar embeddings no chromaDB ou Pinecone (mais robusto que FAISS local).

### 4. Gerenciamento de Sessões

Hoje, todo histórico vai para um único banco SQLite.

Em ambiente real → necessário controlar ID de sessão/usuário.

Exemplo: dois clientes diferentes não podem compartilhar o mesmo histórico.

### 5. Testes Automatizados

Atualmente não há testes.

Melhorias:

Testes unitários para funções (pytest).
