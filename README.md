# 🤖 Trello AI Automation Agent

Este projeto implementa um agente inteligente em Python capaz de automatizar o fluxo de tarefas no Trello utilizando integração com API, lógica de workflow e Inteligência Artificial.

---

# 📌 Objetivo

Automatizar o gerenciamento de tarefas no Trello, permitindo que um agente:

* Leia tarefas (cards)
* Analise o conteúdo usando IA
* Identifique o status correto
* Mova automaticamente os cards entre listas

---

# 🧠 Arquitetura do Sistema

O agente foi dividido em módulos para facilitar manutenção e escalabilidade:

```
agent/
 ├── main.py              # Orquestrador
 ├── trello_client.py    # Integração com Trello
 ├── analyzer.py         # Análise com IA
 ├── workflow.py         # Regras de decisão
 ├── executor.py         # Execução das ações
 └── .env                # Variáveis de ambiente
```

---

# 🔧 Pré-requisitos

* Python 3.9+
* Conta no Trello
* Chave da API do Trello
* Token de acesso
* Chave da OpenAI

---

# 🔑 Configuração do Trello

1. Acesse: [https://trello.com/app-key](https://trello.com/app-key)
2. Gere sua API KEY
3. Gere seu TOKEN

---

# 📦 Instalação das dependências

```bash
pip install requests python-dotenv openai
```

---

# 🔐 Configuração do ambiente

Crie um arquivo `.env` na raiz do projeto:

```env
TRELLO_API_KEY=your_key
TRELLO_TOKEN=your_token
OPENAI_API_KEY=your_openai_key
```

---

# 🧩 Código do Projeto

## 📁 trello_client.py

Responsável pela comunicação com a API do Trello.

```python
import requests
import os
from dotenv import load_dotenv

load_dotenv()

class TrelloClient:
    def __init__(self):
        # Carrega credenciais do ambiente
        self.key = os.getenv("TRELLO_API_KEY")
        self.token = os.getenv("TRELLO_TOKEN")
        self.base_url = "https://api.trello.com/1"

    def get_cards(self, list_id):
        # Busca todos os cards de uma lista específica
        url = f"{self.base_url}/lists/{list_id}/cards"
        params = {"key": self.key, "token": self.token}
        return requests.get(url, params=params).json()

    def move_card(self, card_id, list_id):
        # Move um card para outra lista
        url = f"{self.base_url}/cards/{card_id}"
        params = {
            "idList": list_id,
            "key": self.key,
            "token": self.token
        }
        return requests.put(url, params=params).json()
```

---

## 📁 analyzer.py

Responsável por utilizar IA para classificar o status da tarefa.

```python
from openai import OpenAI
import os

client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))

class TaskAnalyzer:

    def analyze(self, card):
        # Extrai dados do card
        description = card.get("desc", "")
        name = card.get("name", "")

        # Prompt enviado para IA
        prompt = f"""
        Analise a tarefa abaixo e classifique o status:

        Tarefa: {name}
        Descrição: {description}

        Responda apenas com:
        - TODO
        - IN_PROGRESS
        - DONE
        """

        # Chamada à API da OpenAI
        response = client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[{"role": "user", "content": prompt}]
        )

        return response.choices[0].message.content.strip()
```

---

## 📁 workflow.py

Define as regras de negócio para movimentação dos cards.

```python
class WorkflowEngine:

    def decide(self, ai_status, current_list, mapping):
        """
        Decide se o card deve ser movido.

        mapping = {
            "TODO": "list_id_1",
            "IN_PROGRESS": "list_id_2",
            "DONE": "list_id_3"
        }
        """

        target_list = mapping.get(ai_status)

        # Só move se for diferente da lista atual
        if target_list and target_list != current_list:
            return target_list

        return None
```

---

## 📁 executor.py

Executa a ação de mover o card.

```python
class ActionExecutor:

    def __init__(self, trello_client):
        self.trello = trello_client

    def execute(self, card, target_list):
        print(f"Movendo card '{card['name']}' para nova lista...")
        self.trello.move_card(card["id"], target_list)
```

---

## 📁 main.py

Orquestrador principal do agente.

```python
from trello_client import TrelloClient
from analyzer import TaskAnalyzer
from workflow import WorkflowEngine
from executor import ActionExecutor

# IDs das listas do Trello
LIST_TODO = "id_lista_todo"
LIST_IN_PROGRESS = "id_lista_progresso"
LIST_DONE = "id_lista_done"

# Mapeamento de status
mapping = {
    "TODO": LIST_TODO,
    "IN_PROGRESS": LIST_IN_PROGRESS,
    "DONE": LIST_DONE
}

def main():
    # Inicialização dos módulos
    trello = TrelloClient()
    analyzer = TaskAnalyzer()
    workflow = WorkflowEngine()
    executor = ActionExecutor(trello)

    # Busca cards da lista inicial
    cards = trello.get_cards(LIST_TODO)

    for card in cards:
        print(f"Analisando: {card['name']}")

        # IA classifica o status
        ai_status = analyzer.analyze(card)
        print(f"Status IA: {ai_status}")

        # Decide ação
        target = workflow.decide(ai_status, card["idList"], mapping)

        # Executa ação
        if target:
            executor.execute(card, target)
        else:
            print("Nenhuma ação necessária.")

if __name__ == "__main__":
    main()
```

---

# ▶️ Como executar

```bash
python main.py
```

---

# 📊 Benefícios

* Automação de tarefas repetitivas
* Redução de erros humanos
* Aumento de produtividade
* Organização inteligente de fluxo Kanban

---

# 👨‍💻 Autor: Fred.On

Desenvolvido para automação de processos com IA e integração de ferramentas.
