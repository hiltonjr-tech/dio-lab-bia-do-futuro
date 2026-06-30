# Código da Aplicação

Esta pasta contém o código do seu agente financeiro.

Set do Ollama

# 1. Instalar o ollama(ollama.com)
# 2. Baixar um modelo leve
# 3. ollama pull gemma4

## Estrutura Sugerida
ollama run gemma4 "Hello Hilton Jr"


```
src/
├── app.py              # Aplicação principal (Streamlit/Gradio)
├── agente.py           # Lógica do agente
├── config.py           # Configurações (API keys, etc.)
└── requirements.txt    # Dependências
```

## Exemplo de requirements.txt

```

streamlit
openai
python-dotenv
```

## Como Rodar

Todo código fonte está no arquivo app.py

```bash
# Instalar dependências
pip install -r requirements.txt

# Rodar a aplicação
streamlit run app.py

import json
import pandas as pd
import requests
import streamlit as st

OLLAMA_URL = "http://localhost:11434/api/generate"
MODELO = "llama3"

perfil = json.load(open('./data/perfil_investidor.json'))
transacoes = pd.read_csv('./data/transacoes.csv')
historico = pd.read_csv('./data/historico.csv')
produtos = json.load(open('./data/produtos_financeiros.json'))

contexto = f"""
CLIENTE: {perfil['nome']}, {perfil['idade']} anos, perfil {perfil['perfil_investidor.json']}
OBJETIVO: {perfil['objetivo_principal']}
PATRIMÔNIO: {perfil['patrimonio_total']} | RESERVA: R$ {perfil['reserva_emergencia_atual']}

TRANSACOES RECENTES:
{transacoes.to_string(index=False)}

ATENDIMENTOS ANTERIORES:
{historico.to_string(index=False)}

PRODUTOS FINANCEIROS DISPONÍVEIS:
{json.dumps(produtos, indent=2, ensure_ascii=False)}
"""
SYSTEM_PROMPT = """ Você é Hilton Jr, um consultor financeiro amigável, didático e simpático.

OBJETIVOS:
Ensinar conceitos de finanças pessoais de formas simples, usando os dados dos clientes como exemplos práticos.

REGRAS:
1. NUNCA recomende investimento específico - apenas explique como funcionam
2. Use os dados fornecidos para dar exemplos personalizados
3. Se não souber algo, admita e ofereça alternativas
4. Linguagens simples, como se explicassem para um amigo
5. Se não souber algo, admita "Não tenho essa informação, mas posso explicar..."
6. Sempre pergunte se o cliente entendeu
       
"""


def perguntar(msg):
    prompt = f"""
{SYSTEM_PROMPT}

CONTEXTO DO CLIENTE:
{contexto}

Pergunta do cliente: {msg}
"""

    r = requests.post(
        OLLAMA_URL,
        json={
            "model": MODELO,
            "prompt": prompt,
            "stream": False
        }
    )

    return r.json()["response"]

st.title("Hilton Jr, Seu Consultor Financeiro ")

if pergunta := st.text_input("Sua dúvida sobre finanças:"):
    st.chat_message("user").write(pergunta)
    with st.spinner("..."):
        st.chat_message("assistant").write(perguntar(pergunta))
        
                    
   
```
