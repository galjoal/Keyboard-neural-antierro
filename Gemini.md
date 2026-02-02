  
---

```md
# 🏢 GEMINI.md — Regras do Condomínio (Motor + Protótipos)

Este arquivo define regras que toda IA (Gemini, GPT, Claude, Roo, etc.) deve seguir ao trabalhar neste repositório.

---

## ✅ IMPORTANTE (sempre)

1) **Se ficar em dúvida**, liste **até 3 opções** e recomende **1** (com motivo).  
2) **Antes de escrever código**, apresente um **plano com 3 a 8 tópicos** (para eu aprovar e saber o tamanho da mudança).  
3) **Mudanças pequenas e revisáveis**: 1 tarefa = 1 PR (ou 1 commit pequeno, se eu estiver trabalhando direto).

---

## 🎯 Visão (North Star) do projeto

- Este projeto nasce como **motor/estudo**, mas o jogo final é **estratégico–tático–imersivo**:
  - mundo **altamente interativo**;
  - escolhas do jogador **alteram rotas/oportunidades** (estilo “rotas” + consequências);
  - ambiente com profundidade e interações físicas/sistêmicas (quebrar/arrastar/soltar/queimar) influencia a tática.
- O **ambiente é quase um personagem**: tiles/blocos têm **HP**, material e reações.
- Diretriz central: **estruturas simples e baratas** (CPU/RAM), mas com **emergência** e diversidade.

> Regra: o motor deve permitir isso no futuro, mas **protótipos e core** devem continuar simples e incrementais.

---

## 📋 Sobre o projeto (escopo atual)

- Tipo: **base de motor** para jogo tático por turnos (estudo).
- Estilo: **2.5D** (lógica 3D/camadas, render inicial 2D).
- Linguagem: **Rust**.
- Target inicial: **WASM (browser)** → Android (futuro).
- Ritmo: sem pressa, evolução gradual por microprotótipos.

---

## ⚙️ Requisitos não-funcionais (desde o início)

- **Baixo custo de CPU/RAM** como prioridade.
- Evitar:
  - alocações desnecessárias em loops “por frame”;
  - varrer o mapa inteiro todo quadro sem motivo;
  - estruturas extremamente complexas antes do tempo.
- Preferir:
  - dados compactos e claros;
  - atualizações por eventos/dirty flags quando fizer sentido;
  - determinismo (bom para turnos, replay, debug).

> Não é “otimização prematura”: é **não escolher arquiteturas caras** sem necessidade.

---

## 🏗️ Regra 1: Estrutura “Condomínio” (padrão de arquivo)

Cada arquivo é um “prédio” com “apartamentos” (seções).  
A ordem das seções **não pode** ser necessária para o código funcionar.

Modelo recomendado:

```rust
// ================================================
// SEÇÃO 1: IMPORTS
// ================================================

// ================================================
// SEÇÃO 2: TIPOS / DADOS
// ================================================

// ================================================
// SEÇÃO 3: CONSTRUTORES / DEFAULTS
// ================================================

// ================================================
// SEÇÃO 4: CONSULTAS (não modificam estado)
// ================================================

// ================================================
// SEÇÃO 5: MODIFICADORES (alteram estado)
// ================================================

// ================================================
// SEÇÃO 6: FUNÇÕES PURAS / REGRAS (sem efeitos colaterais)
// ================================================

// ================================================
// SEÇÃO 7: TESTES
// ================================================
```

Observações:
- Comentários **só** onde for realmente útil (explicar o não-óbvio).
- Para isolamento extra, pode usar módulos internos: `mod data`, `mod rules`, `mod tests`.

---

## 📏 Regra 2: Tamanho dos arquivos

- Alvo: **200–400 linhas** por arquivo (mobile-friendly).
- Passou de 400 → **dividir** em módulos menores.

---

## 🚫 Regra 3: Sem estado global bagunçado

Proibido:
- `static mut`
- singletons escondidos
- `lazy_static`/globais mutáveis como “atalho de arquitetura”

Permitido:
- estado dentro de `struct` explícita (ex.: `GameState`, `World`, `Map`),
- recursos passados por referência.

---

## 📁 Regra 4: Estrutura de pastas (padrão)

```
motor-tatico/
├── GEMINI.md
├── README.md
├── docs/
│   └── decisions/
├── prototipos/
│   ├── proto_01_xxx/
│   ├── proto_02_xxx/
│   └── ...
└── core/
    └── src/
        ├── lib.rs
        ├── componentes/
        ├── sistemas/
        ├── recursos/
        └── util/
```

Não fazer reorganizações grandes sem pedir confirmação.

---

## 🎯 Regra 5: Um protótipo = um objetivo

Certo:
- `proto_01_ponto` → desenhar algo na tela
- `proto_02_input` → mover com input
- `proto_03_grid` → grid clicável
- `proto_04_projecao` → 3D→2D simples

Errado:
- `proto_01_tudo` → grid + câmera + combate + inventário

Cada protótipo deve ter um `README.md` com:
- o que demonstra
- como rodar
- o que observar

---

## 🧱 Regra 6: Camadas (Z) e interatividade do mundo

- **Camadas são dados**: o motor deve suportar N camadas por cenário.
- Evitar “hardcode” de quantidade fixa de camadas.
- Tiles/blocos podem ter **HP** e propriedades de interação (material, resistência, etc.).

Nota importante:
- Ter constantes utilitárias (ex.: “superfície = 0”) é OK,
  desde que **o mapa não dependa de um número fixo de camadas**.

---

## 🧪 Regra 7: Testes (quando aplicável)

- Em `core/`, sempre que der, incluir testes básicos para regras puras.
- Em protótipos, testes são opcionais, mas:
  - documentar “como verificar visualmente”
  - manter o código simples

---

## 📝 Regra 8: Padrão de commits

Formato:
`<tipo>: <descrição curta>`

Tipos:
- `proto` → novo protótipo
- `feat` → nova funcionalidade
- `fix` → correção de bug
- `refac` → refatoração sem mudar comportamento
- `docs` → documentação
- `test` → testes

Exemplos:
- `proto: criar proto_01 tela básica`
- `feat: adicionar componente Posicao`
- `fix: corrigir cálculo de projeção`
- `docs: explicar como rodar WASM`

---

## 🔄 Regra 9: Pull Requests

Todo PR deve ter:

```md
## O que mudou
- ...

## Por que mudou
- ...

## Como testar
1. ...
2. ...

## Checklist
- [ ] Segue estrutura de seções (Condomínio)
- [ ] Arquivos <= 400 linhas (ou justificativa)
- [ ] Sem estado global novo
- [ ] Testes passando (se aplicável)
```

---

## 🔐 Segurança

- Nunca colocar tokens/segredos no repo.
- Não sugerir comandos destrutivos sem confirmação explícita.
- Não abrir portas/serviços expostos por padrão.

---

## 📚 Referências rápidas

- Projeção 3D→2D (base): `x' = x/z`, `y' = y/z`
- Target: WASM (browser)
```

---

Se você quiser, eu também posso:
- ajustar esse `GEMINI.md` para refletir *exatamente* a estrutura real do seu repo (pastas/nomes atuais);
- e escrever um `README.md` mínimo do projeto com “como rodar o Proto 01”.

Qual vai ser o **Proto 01** que você quer primeiro: “tela básica” (quadrado na tela) ou “grid simples”?
