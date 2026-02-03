# GEMINI.md — Versão Final Híbrida para Error-MLM

```markdown
# 🏢 GEMINI.md — Regras do Condomínio

Este arquivo define as regras que toda IA (Gemini, GPT, Claude, Roo) deve seguir ao trabalhar neste projeto.

---

## ⚠️ IMPORTANTE (sempre)

1. **Se ficar em dúvida**, liste **até 3 opções** e recomende **1** (com motivo).
2. **Antes de escrever código**, apresente um **plano com 3 a 8 tópicos** (para eu aprovar e saber o tamanho da mudança e quantidade de tokens aproximados).
3. **Mudanças pequenas e revisáveis**: 1 tarefa = 1 PR (ou 1 commit pequeno).

---

## 📋 SOBRE O PROJETO

```
Tipo: Teclado inteligente Android (IME) com correção por ML
Diferencial: Corrige erros SEM aprender eles
Privacidade: 100% local, zero dados enviados
Stack: Rust (core) + Kotlin (Android) + TinyBERT (ONNX)
Target: Android API 26+ → Play Store
Ritmo: Protótipo → Beta → Produção
```

### Visão (North Star)
- Teclados tradicionais aprendem "vc", "tbm", "oq" como corretos
- Error-MLM faz o **caminho inverso**: sempre sugere a forma correta
- Produto vendável: múltiplos idiomas, UX intuitiva, privacidade total

---

## 🏗️ REGRA 1: Estrutura "Condomínio" (Seções)

Cada arquivo é um "prédio" com "apartamentos" (seções numeradas).

```rust
// ════════════════════════════════════════════════════════════
// SEÇÃO 1: IMPORTS
// ════════════════════════════════════════════════════════════
use std::collections::VecDeque;
use std::time::Instant;

// ════════════════════════════════════════════════════════════
// SEÇÃO 2: TIPOS E STRUCTS
// ════════════════════════════════════════════════════════════
// 📋 Descrição: Estruturas de dados principais
// ════════════════════════════════════════════════════════════
pub struct PauseDetector {
    intervals: VecDeque<u64>,
    max_history: usize,
}

pub enum Trigger {
    Pause,
    Punctuation,
    WordBoundary,
}

// ════════════════════════════════════════════════════════════
// SEÇÃO 3: CONSTRUTORES (new, default, from)
// ════════════════════════════════════════════════════════════
impl PauseDetector {
    pub fn new() -> Self {
        Self {
            intervals: VecDeque::with_capacity(20),
            max_history: 20,
        }
    }
}

// ════════════════════════════════════════════════════════════
// SEÇÃO 4: CONSULTAS (não modificam estado)
// ════════════════════════════════════════════════════════════
impl PauseDetector {
    pub fn average(&self) -> u64 {
        if self.intervals.is_empty() { return 200; }
        self.intervals.iter().sum::<u64>() / self.intervals.len() as u64
    }
    
    pub fn is_pause(&self, current_ms: u64) -> bool {
        current_ms > (self.average() * 3).max(500)
    }
}

// ════════════════════════════════════════════════════════════
// SEÇÃO 5: MODIFICADORES (alteram estado)
// ════════════════════════════════════════════════════════════
impl PauseDetector {
    pub fn record(&mut self, interval_ms: u64) {
        if self.intervals.len() >= self.max_history {
            self.intervals.pop_front();
        }
        self.intervals.push_back(interval_ms);
    }
}

// ════════════════════════════════════════════════════════════
// SEÇÃO 6: TESTES
// ════════════════════════════════════════════════════════════
#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_pause_detection() {
        let mut detector = PauseDetector::new();
        detector.record(100);
        detector.record(150);
        assert!(detector.is_pause(1000)); // 1s é pausa
        assert!(!detector.is_pause(200)); // 200ms não é
    }
}
```

### Por que isso importa?
- Seções podem ser editadas **sem afetar outras**
- Ordem das seções **não quebra o código**
- Facilita revisão no celular
- Erro reporta a SEÇÃO afetada

---

## 📏 REGRA 2: Tamanho dos Arquivos

```
LIMITE: 200-400 linhas por arquivo
MOTIVO: Leitura confortável no celular
AÇÃO: Se passar de 400 linhas → dividir em módulos
```

---

## 🚫 REGRA 3: Sem Estado Global

```rust
// ❌ PROIBIDO
static mut MODELO: Option<TinyBert> = None;

// ❌ PROIBIDO
lazy_static! {
    static ref CONFIG: Mutex<Config> = Mutex::new(Config::default());
}

// ✅ PERMITIDO: Estado dentro de structs
pub struct Engine {
    tinybert: TinyBert,
    config: Config,
    buffer: InputBuffer,
}
```

---

## 📁 REGRA 4: Estrutura de Pastas

```
error-mlm/
├── 📄 README.md              ← Visão pública
├── 📄 GEMINI.md              ← Este arquivo (regras IA)
├── 📄 BLOCKS.md              ← Índice de seções/blocos
│
├── 📁 core/                  ← Rust (motor)
│   ├── 📄 Cargo.toml
│   ├── 📄 lib.rs             ← Entry point
│   ├── 📄 engine.rs          ← Motor principal
│   ├── 📄 detector.rs        ← Detecção de pausa
│   ├── 📄 corrector.rs       ← Lógica de correção
│   ├── 📄 tinybert.rs        ← Wrapper ONNX
│   ├── 📄 symspell.rs        ← Gerador de candidatos
│   └── 📄 undo.rs            ← Pilha de desfazer
│
├── 📁 android/               ← Kotlin (IME)
│   ├── 📄 IMEService.kt      ← Serviço principal
│   ├── 📄 PreviewPanel.kt    ← Painel de sugestões
│   ├── 📄 RustBridge.kt      ← JNI bridge
│   └── 📄 Settings.kt        ← Configurações
│
├── 📁 models/                ← ML e dados
│   ├── 📄 tinybert.onnx
│   └── 📄 dict_ptbr.txt
│
└── 📁 tests/                 ← Benchmarks
    ├── 📄 bench_tinybert.rs
    └── 📄 bench_symspell.rs
```

---

## 🎯 REGRA 5: Um Teste = Um Objetivo

```
CERTO:
bench_tinybert   → Medir latência do TinyBERT direto
bench_symspell   → Medir SymSpell + TinyBERT
test_pausa       → Validar detecção de pausa

ERRADO:
test_tudo        → Pausa + correção + undo + UI
```

Cada teste prova **UMA coisa**.

---

## 📝 REGRA 6: Padrão de Commits

```
FORMATO:
<tipo>: <descrição curta>

TIPOS:
feat    → Nova funcionalidade
fix     → Correção de bug
refac   → Refatoração
test    → Testes/benchmarks
docs    → Documentação
chore   → Configs, deps

EXEMPLOS:
feat: adicionar PauseDetector
fix: corrigir threshold de pausa
test: benchmark TinyBERT vs SymSpell
docs: atualizar GEMINI.md
```

---

## 🔄 REGRA 7: Padrão de Pull Requests

```markdown
## O que mudou
[Descrição clara]

## Por que mudou
[Motivação]

## Como testar
[Passos para verificar]

## Checklist
- [ ] Código segue estrutura de SEÇÕES
- [ ] Arquivo com menos de 400 linhas
- [ ] Sem estado global novo
- [ ] Testes passando
```

---

## ⚙️ REGRA 8: Requisitos Não-Funcionais

| Requisito | MVP | Produção |
|-----------|-----|----------|
| **Latência** | <100ms | <50ms |
| **Memória** | <150MB | <100MB |
| **Bateria/hora** | <5% | <2% |
| **APK** | <50MB | <30MB |
| **Cold start** | <1s | <500ms |
| **Acurácia PT-BR** | >80% | >95% |

---

## 🔌 REGRA 9: Conceitos do Teclado

### Pipeline de Correção
```
Texto → SymSpell (candidatos) → TinyBERT (scorer) → Sugestão
```

### TinyBERT é Scorer, não Corretor
```rust
// ❌ ERRADO: TinyBERT corrige direto
let corrigido = tinybert.correct(texto);

// ✅ CERTO: SymSpell gera, TinyBERT escolhe
let candidatos = symspell.lookup(palavra, 3);
let melhor = tinybert.score(&candidatos, contexto);
```

### UI dentro do IME (não overlay)
```kotlin
// ❌ ERRADO: Overlay sobre apps
windowManager.addView(overlayView, params)

// ✅ CERTO: Painel dentro do teclado
previewPanel.visibility = View.VISIBLE
```

### Pilha de Undo (não lista de strings)
```rust
// ❌ ERRADO
let historico: Vec<String>;

// ✅ CERTO
struct UndoOp {
    original: String,
    corrigido: String,
    range: TextRange,
    tipo: OpType,
}
let undo_stack: VecDeque<UndoOp>;
```

---

## 🤖 INSTRUÇÕES PARA A IA

Quando trabalhar neste projeto:

1. **SEMPRE** usar estrutura de SEÇÕES nos arquivos
2. **NUNCA** criar estado global (static mut, lazy_static)
3. **SEMPRE** explicar o que o código faz em comentários
4. **NUNCA** modificar múltiplas seções sem pedir
5. **SEMPRE** manter arquivos abaixo de 400 linhas
6. **SEMPRE** incluir testes básicos (SEÇÃO 6)
7. **PERGUNTAR** se algo não estiver claro nas regras
8. **REPORTAR** erros com nome da SEÇÃO afetada

---

## 🚀 COMANDOS PARA SOLICITAR AJUDA

### Corrigir seção com erro:
```
@fix SEÇÃO: 4 (Consultas)
Arquivo: core/detector.rs
Erro: função is_pause retorna sempre false
```

### Adicionar nova seção:
```
@add SEÇÃO: nova
Arquivo: core/corrector.rs
Descrição: Correção de palavra única
```

### Mover código entre arquivos:
```
@move SEÇÃO: 5 (Modificadores)
De: core/engine.rs
Para: core/buffer.rs
```

### Comparar abordagens:
```
@compare
Opção A: TinyBERT direto
Opção B: SymSpell + TinyBERT
Critério: latência + acurácia
```

---

## 🎯 ROADMAP

### Fase 1: Protótipo Core ← ATUAL
- [ ] Setup Rust + estrutura
- [ ] PauseDetector + InputBuffer
- [ ] Benchmark TinyBERT vs SymSpell

### Fase 2: Android IME
- [ ] IME básico (teclado funcional)
- [ ] JNI bridge Kotlin↔Rust
- [ ] PreviewPanel

### Fase 3: Integração
- [ ] Pipeline completo
- [ ] UndoStack
- [ ] 3 modos de correção

### Fase 4: Polish
- [ ] Onboarding (3 telas)
- [ ] Configurações
- [ ] Multi-idioma (PT + EN)

---

## 📊 HISTÓRICO DE DECISÕES

| Data | Decisão | Motivo |
|------|---------|--------|
| [hoje] | UI: Painel no IME, não overlay | Play Store rejeita overlay |
| [hoje] | TinyBERT como scorer | Pipeline mais eficiente |
| [hoje] | Heurística para trigger | Neural é overkill |
| [hoje] | Pilha de operações para undo | Mais robusto que strings |
| [hoje] | Seções numeradas | Mais claro que blocos nomeados |

---

## 📚 REFERÊNCIAS

- **TinyBERT**: Modelo leve para MLM (~15MB quantizado)
- **SymSpell**: Corretor por distância de edição (~1ms)
- **ONNX Runtime**: Inferência mobile otimizada
- **Android IME**: InputMethodService API
- **JNI**: Java Native Interface para Rust

---

*Última atualização: [DATA]*
*Versão: 1.0*
```

---
