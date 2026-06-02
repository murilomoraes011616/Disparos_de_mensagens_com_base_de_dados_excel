# AUTOMAÇÃO DE DISPAROS - DOCUMENTAÇÃO TÉCNICA COMENTADA

## Visão Geral

Este workflow automatiza o envio de mensagens personalizadas via WhatsApp para pacientes cadastrados em uma planilha Google Sheets.

O processo utiliza:
- Google Sheets
- Google Gemini
- Evolution API
- JavaScript
- n8n

## 1. Schedule Triggers

### Schedule Trigger (13h)
Responsável por iniciar automaticamente o workflow às 13h.

### Schedule Trigger1 (10h)
Responsável por iniciar automaticamente o workflow às 10h.

### Schedule Trigger2 (11h)
Responsável por iniciar automaticamente o workflow às 11h.

Objetivo: distribuir os disparos ao longo do dia para evitar excesso de mensagens.

---

## 2. Consulta da Planilha

### Get row(s) in sheet

Consulta a planilha Google Sheets filtrando contatos onde o campo `foi_enviado` está vazio.

Objetivo: evitar disparos duplicados.

---

## 3. Limit

Limita a execução para apenas 20 contatos por rodada.

Benefícios:
- Menor carga na API
- Melhor controle operacional
- Evita grandes volumes simultâneos

---

## 4. Loop Over Items

Processa os contatos individualmente.

Objetivo:
Garantir tratamento individual para cada paciente.

---

## 5. IF

Condição:
`foi_enviado` está vazio.

Resultado:
- Verdadeiro → envia para IA
- Falso → ignora contato

---

## 6. Nó "faz nada"

Utilizado para encerrar o processamento de registros que não precisam ser tratados.

---

## 7. AI Agent

Responsável pela geração da mensagem personalizada.

Modelo utilizado:
Google Gemini 2.0 Flash

Objetivos:
- Personalização
- Humanização
- Aumentar taxa de resposta

---

## 8. Google Gemini Chat Model

Configuração:
- Modelo: gemini-2.0-flash
- Temperature: 0.8

---

## 9. Structured Output Parser

Força a IA a responder no formato:

```json
{
  "name": "nome",
  "numero": "telefone",
  "text": "mensagem"
}
```

---

## 10. Edit Fields

Reorganiza a saída da IA criando:

- output.name
- output.numero
- output.text

---

## 11. Split Out

Transforma os parágrafos da mensagem em itens individuais.

Objetivo:
Enviar mensagens de forma mais natural.

---

## 12. Loop de Mensagens

Controla o envio de cada parte da mensagem individualmente.

---

## 13. Evolution API

Responsável pelo envio da mensagem via WhatsApp.

Dados enviados:
- Número
- Mensagem

---

## 14. JavaScript - Intervalo Curto

Gera uma pausa aleatória entre 10 e 60 segundos.

Objetivo:
Evitar disparos instantâneos.

---

## 15. Wait7

Pausa o fluxo pelo tempo sorteado e retorna ao loop.

---

## 16. JavaScript - Intervalo Longo

Gera uma pausa aleatória entre 40 e 100 segundos.

Objetivo:
Espaçar o envio entre contatos.

---

## 17. Wait1

Aguarda o tempo sorteado antes da atualização da planilha.

---

## 18. Update Row in Sheet

Atualiza:
- nome
- numero
- mensagem
- data_hora
- foi_enviado

Define:
`foi_enviado = ENVIADO`

Objetivo:
Criar histórico e impedir reenvios.

---

## Fluxo Geral

Schedule
↓
Google Sheets
↓
Limit (20)
↓
Loop
↓
IF
↓
Gemini
↓
Parser
↓
Preparação dos Dados
↓
Split
↓
Evolution API
↓
Wait Aleatório
↓
Atualiza Planilha
↓
Próximo Contato

Quando não existirem mais registros pendentes, o workflow encerra e aguarda o próximo horário programado.
