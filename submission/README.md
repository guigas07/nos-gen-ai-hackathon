# The *-Files — Anonymização de Documentos Médicos

Equipa: Larp_Coders
Membros: Guilherme Gomes
Desafio: NOS / UNTRACE — JunctionX Lisbon 2026

## O que faz

Este projeto anonimiza um documento, mascarando toda a informação que permite
identificar uma pessoa concreta, enquanto preserva o resto do documento
— estrutura, formatação, conteúdo clínico e social — exatamente como
estava no original.

O output (`submission/submission.txt`) mantém as mesmas linhas, a mesma
pontuação e os mesmos espaçamentos do documento original; a única
diferença é que cada palavra sensível é substituída por um `*`.

## Abordagem

1. Extração: `extract_text_from_pdf` usa PyMuPDF para converter o PDF
   em texto simples, preservando acentos, pontuação e quebras de linha.
   A única alteração nesta fase é a remoção de linhas vazias, conforme
   pedido no enunciado.
2. Prompt: as instruções de mascaramento (ver `submission/prompt.txt`
   para o prompt exato enviado, já com o documento incluído) definem,
   por categoria, o que conta como informação sensível e o que deve
   permanecer visível.
3. Chamada ao modelo: usamos a API do Gemini (`<nome do modelo>`,
   temperatura 0) para processar o documento.
4. Mascaramento: com o prompt que o gemini recebe é feito a anonimização
    do documento, substituindo as informações sensíveis por *.
5. Validação: `validate_output` faz uma verificação automática:
   compara o número de palavras por linha entre o original e o
   mascarado, confirma que as 4 linhas de cabeçalho conhecidas (título,
   data, referência, "Informações do Paciente:") permanecem inalteradas,
   e procura padrões de emails/telefones/NIFs que possam ter escapado
   por engano.

## Política de mascaramento

Mascaramos apenas o que pode ligar o documento a uma pessoa concreta:
nomes (paciente, família, amigos, médico), datas de nascimento,
identificadores governamentais (NIF, Cartão de Cidadão, Segurança
Social), contactos (telefone, email, morada), identificadores
financeiros (cartão de crédito, apólice, rendimento), identificadores
biométricos/de sistema, número de cédula profissional (CRM), idade
quando referida junto ao nome de uma pessoa, etnia/raça e religião de
uma pessoa específica, e o nome da instituição empregadora quando torna
a ocupação demasiado específica.

Deixamos visível o conteúdo clínico e social que não identifica a
pessoa por si só: diagnósticos comuns, sintomas, medicação, sinais
vitais, exames, estado civil, profissão genérica, hábitos, e os
descritores de relação entre parênteses (ex: "(irmão)"). O título do
documento, o nome da instituição no cabeçalho, a data de admissão e o
número de referência também ficam inalterados, por identificarem o
registo/instituição e não uma pessoa — este critério foi calibrado a
partir do próprio exemplo de output correto dado no enunciado do
desafio.

## Setup

Pré-requisitos: Python 3.10+, uma chave de API do Gemini
([aistudio.google.com](https://aistudio.google.com/)).

```bash
git clone https://github.com/<o-vosso-fork>/nos-gen-ai-hackathon.git
cd nos-gen-ai-hackathon
python3 -m venv .venv
source .venv/bin/activate 
pip install -r requirements.txt 
```

Criar um ficheiro `.env` na raiz do repositório (nunca o commitem):

```env
GEMINI_API_KEY=a_vossa_chave_aqui
```

## Como correr

```bash
python submission/anonymizer.py
```

Isto gera/atualiza `submission/submission.txt` (documento mascarado) e
`submission/prompt.txt` (prompt exato enviado ao modelo), e imprime no
terminal os avisos de validação, se os houver.

## Estrutura

```text
submission/
  anonymizer.py       # pipeline completo: extração, prompt, chamada à API, validação
  prompt_template.txt # instruções de mascaramento (sem o documento)
  prompt.txt           # gerado — instruções + documento, exatamente como enviado ao Gemini
  submission.txt        # gerado — o documento anonimizado
  README.md
```

## Equipa

Guilherme Gomes e Martim Jaime — divisão de trabalho: extração/chamada ao Gemini e
afinação do prompt de mascaramento, partilhados entre os dois ao longo
de várias iterações (ver histórico de commits/merges no branch
`team-Larp_Coders`).