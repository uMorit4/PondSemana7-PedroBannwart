## (Q1) Efeito de `num_examples` no vocabulário (D2L NMT)

| `num_examples` | Max ID observado (X/Y) | Limite inferior p/ tamanho do vocab* | ID de `<pad>` (pelas amostras) | `<unk>` presente na amostra? | valid lengths (X) | valid lengths (Y) | Observações principais |
|---:|:---:|:---:|:---:|:---:|:---:|:---:|:--|
| 100  | 10 / **33** | **≥ 34** | **4** | **Sim** (`0` no Y) | [4, 3] | [4, 3] | Vocab pequeno → mais `<unk>`; IDs menores; padding = 4. |
| 600  | 81 / **172** | **≥ 173** | **5** | **Não** | [5, 4] | [4, 3] | Vocab cresce; cobertura melhor (sem `<unk>` na amostra). |
| 2000 | 312 / **576** | **≥ 577** | **6** | **Não** | [4, 5] | [4, 7] | Vocab bem maior; `<pad>` muda de ID; ampla cobertura lexical. |

\* “Limite inferior” = (maior ID observado + 1), pois o tamanho real pode ser maior do que aparece na amostra.

**Takeaways rápidos:**  
- Aumentar `num_examples` **aumenta o vocabulário** e **reduz `<unk>`**.  
- **IDs não são estáveis** (inclusive dos especiais), então **não hardcode**; sempre leia `padding_idx`/`ignore_index` do vocab retornado.  
- `valid lengths` variam pelos pares amostrados (não por “tamanho do vocab” diretamente).

---

## (Q2) Tokenização em idiomas sem espaços (chinês/japonês): word-level é uma boa ideia?

**Resposta curta:** Geralmente **não**. Os seus testes mostram que **subwords** (SentencePiece/BPE/WordPiece) são a melhor escolha, com **char-level** como alternativa robusta.

### Evidências dos seus experimentos (simulando “sem espaços” no lado fonte)

- **WORD (src sem espaços)**  
  - `src_vocab size: 335` | `tgt_vocab size: 545`  
  - `valid X: [2, 2]` → a sentença fonte colapsa em **1 token gigante + <eos>**; quase tudo vira `<pad>`.  
  - Ex.: `X = [[130, 3, 1, 1, 1, 1, 1, 1], ...]`  
  - **Leitura:** Word-level **quebra** sem delimitadores; perda de granularidade e má generalização.

- **CHAR (src sem espaços)**  
  - `src_vocab size: 39` | `tgt_vocab size: 49`  
  - `valid X: [7, 8]` → sequências **mais longas**, mas **zero OOV**.  
  - Ex.: `X = [[13, 6, 27, 7, 15, 4, 3, 1], ...]`  
  - **Leitura:** Funciona, porém custa mais modelar dependências longas a partir de unidades muito pequenas.

- **SPM / Subword (src sem espaços)**  
  - `src_vocab size: 332` | `tgt_vocab size: 442`  
  - `valid X: [4, 4]` → **comprimento intermediário**, mais informativo que word-colapsado e menor que char.  
  - Ex.: `X = [[108, 16, 4, 3, 1, 1, 1, 1], ...]`  
  - **Leitura:** Divide em morfemas/partes frequentes, **baixa OOV** e sequências manejáveis. Melhor trade-off.

### Conclusão prática
- **Word-level** sem espaços → **não recomendado** (colapso em um token; `valid X` muito baixos).  
- **Char-level** → **robusto**, porém com sequências **longas**.  
- **Subwords (SentencePiece/BPE/WordPiece)** → **recomendado**: vocabulário moderado, **sequências intermediárias** e boa cobertura. É o padrão de fato para CJK.

