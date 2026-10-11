# Projeto 9 — PTE Master Hub: integridade do progresso, medição honesta e fidelidade à prova real

> Estado de partida (medido em 10/10/2026 sobre o zip `clau-4.3-main`): projetos 1–8 concluídos.
> vitest 258/258 (27 arquivos) · tsc limpo · lint 0 erros (8 warnings) · build OK · 2.536 questões / 19 task types.
> O app funciona. Este projeto não adiciona "mais do mesmo": corrige o que hoje **mostra números errados ao aluno**,
> **pode apagar o progresso dele**, e **não treina o que a prova real cobra**.

## Como ler as evidências
Cada achado traz **prova reproduzível** (script/teste/grep). Onde a evidência vem de fonte externa ou não foi
exercitada (mic real, a11y, mobile), isso está marcado — o agente deve confirmar no PRD antes de agir.

## Problemas observados (do mais grave ao menos grave)

### A. O progresso do aluno pode ser perdido ou corrompido (P0)
| # | Achado | Evidência |
|---|---|---|
| A1 | **O teto `MAX_ATTEMPTS=5000` apaga o SRS.** `computeStats` deriva caixa/`dueAt`/dominada **do log de tentativas**; ao cortar as mais antigas, a questão volta a "nunca tentada". | Probe: questão com 3 tentativas fortes (caixa 3) → após estourar o teto, `stats.get(id)` = `undefined`. |
| A2 | **Estado ilegível ou de versão desconhecida vira `emptyState()` e é sobrescrito na próxima gravação**, sem cópia. Uma versão futura (v6) aberta por um build antigo apaga tudo. | `storage.ts`: `load()` (catch → vazio) e `migrateLegacyState` (versão não listada → vazio) + `persist()`. |
| A3 | **Importar backup substitui tudo, sem prévia, sem confirmação, sem mesclar.** Importar um backup antigo destrói o progresso mais novo. | `settings.tsx:174` chama `importBackup` direto (sem `confirm`/`AlertDialog`); `importBackup` → `setState(migrated)`. |
| A4 | Persistência = `JSON.stringify` do estado inteiro a cada mutação (síncrono), sem aviso **preventivo** de quota; sem `navigator.storage.persist()`; sem lembrete de backup. No Safari/iOS o navegador pode limpar armazenamento de sites pouco usados. | `persist()`; `usePersistFailed` só avisa **depois** da falha. (Comportamento do Safari: fato conhecido de plataforma — confirmar no PRD.) |
| A5 | `computeStreak`/`dailyAverages` andam em passos fixos de 24 h; em fuso com horário de verão o dia pode ser pulado/duplicado. | `storage.ts` (`t -= DAY`). Severidade baixa. |

### B. A medição ensina números que não significam o que parecem (P0)
| # | Achado | Evidência |
|---|---|---|
| B1 | **Escalas misturadas.** Ajustes pede "meta de score no PTE (escala 10–90)", mas a Home compara com a **média em % (0–100)**: "média 72% / alvo 65". | `goal.ts → targetVerdict(recentAvg, targetScore)`; `index.tsx` linha do `verdict`; `settings.tsx` ("escala 10–90"). |
| B2 | **Prontidão mede o tamanho do banco, não a habilidade.** 40% do índice é cobertura (tentadas/total). Listening tem 1.168 itens, 934 só de WFD → a cobertura nunca sobe; a média mistura tipos de bancos muito desiguais. | `readiness.ts` (`WEIGHT_COVERAGE=40`); `stability = 1 − σ/100` quase sempre ≈ 0,85–0,95 (pouco discrimina). |
| B3 | **Pesos 30/30/20/20 sem fonte** e modelo "1 tarefa → 1 seção". Na prova, tarefas pontuam **mais de uma habilidade** (ex.: Summarize Group Discussion → Listening+Speaking; Summarize Written Text → Reading+Writing; Summarize Spoken Text → Listening+Writing). | `studyPlan.ts` (`EXAM_WEIGHTS`). Matriz acima vem de fonte secundária (PrepEx, "checada contra a Pearson em 23/06/2026") — **confirmar no Score Guide oficial**. |
| B4 | **Score de fala ignora o que sobra.** `extras` e `repetitions` são calculados e **não entram** no score; fluência e pronúncia não são medidas (a Web Speech API não expõe pronúncia; `confidence` não é lido). Reconhecedores tendem a "corrigir" para palavras prováveis → score inflado. | `scoring.ts → gradePassage` (`credit/total` só com hit/subst./moved); `useSpeechRecognition.ts` (só `transcript`). |
| B5 | **SRS pode ser inflado por prática seguida.** Toda tentativa ≥ 80 sobe a caixa, mesmo fora do prazo; qualquer falha derruba para 0; o mesmo limiar 80 vale para todos os tipos. | Probe: 5 acertos em 5 minutos → **caixa 5, próxima revisão em 30 dias**. |
| B6 | Texto de Ajustes diz "Nenhuma conta, nenhum servidor", mas no Chrome o reconhecimento de voz costuma rodar em servidor do navegador. | `settings.tsx:295`; (comportamento do Chrome: fato conhecido — confirmar no PRD). |

### C. O app não treina a prova que existe hoje (P1)
| # | Achado | Evidência |
|---|---|---|
| C1 | **A prova tem 22 task types desde 07/08/2025; o app cobre 19.** Faltam **Summarize Group Discussion**, **Respond to a Situation** e **Describe Image** (este já "em breve"). `manifest.webmanifest` ainda diz "16 task types". | Fontes externas (Cialfo, Collegedunia, PrepEx — secundárias); `public/manifest.webmanifest`. `metrics-sync.test.ts` não cobre `public/`. |
| C2 | **Proveniência invisível.** 647 itens são `ai-generated-prediction` (Read Aloud 140/195, Repeat Sentence 199/586, WFD 308/934) + 200 `pteracai-seed`; a UI **nunca** mostra `source`. Contradiz a regra do projeto 4 ("nada gerado por IA"). | `grep` de `.source` em `components/` e `routes/` = vazio; contagem por `source` nos JSONs. |
| C3 | **Quase-duplicatas furaram o "zero duplicatas".** 28 passagens de SWT aparecem duas vezes (`pte-swt-*` × `goarno-swt-NN`, mesmo início de 200 caracteres; similaridade 0,88 na amostra). O progresso é por `qid`, então estatística e SRS se dividem. | Script de prefixo de 200 caracteres normalizados em `src/data/pte/swt.json`. |
| C4 | **Bancos rasos viram decoreba.** `lst_hcs` 3 · `lst_hiw` 3 · `lst_smw` 3 · `lst_mcq_multi` 3 · `lst_summary` 5 · `lst_mcq` 6. | Contagem real dos JSONs (o manifesto bate: 2.536). |
| C5 | **O "simulado" é um mini-simulado.** Ex.: Speaking = 8 questões (~7 min) vs. prova de ~2h15 e 22 tasks; tempos em `EXAM_SECONDS` sem fonte citada (SGD e RTS nem existem). | `exam.ts` (`SECTION_PLANS`, `EXAM_SECONDS`). Duração oficial: fontes secundárias. |
| C6 | Listening usa TTS do navegador; a UI não informa **qual voz/sotaque** foi realmente usada (fallback por prefixo). | `useSpeechSynthesis.ts → pickVoice`. Qualidade por dispositivo **não foi medida**. |

### D. Engenharia e operação (P1–P2)
| # | Achado | Evidência |
|---|---|---|
| D1 | **Offline só funciona para o que já foi visitado.** O `sw.js` pré-cacheia 5 rotas; chunks de dados entram no cache só quando abertos online. Sem aviso de "nova versão", `CACHE` manual (`v2`). | `public/sw.js`; chunks: `wfd` 522 KB, `summarize_spoken_text` 435 KB, `glossary` 373 KB, `index` 339 KB, `pte` 294 KB (brutos). |
| D2 | **Payload inflado.** WFD: `text = transcript = audio_text = answer` em 934/934 itens → ≈ **51% dos bytes são repetição**. | Script sobre `src/data/pte/wfd.json`. |
| D3 | **Sem CI, sem hooks, sem scripts `test`/`typecheck`.** Os gates vivem só no agente (e citam `tsgo`/`bunx`, não declarados). | `package.json` (scripts: dev/build/lint/format…); ausência de `.github/`. |
| D4 | **Dois lockfiles e um deles quebrado.** `npm ci` falha: `package-lock.json` não bate com `package.json` (ajv, `@lovable.dev/vite-tanstack-config`). | Execução real de `npm ci`. |
| D5 | **Cobertura de testes não chega nas telas.** 258 testes, mas só 1–2 arquivos renderizam componentes; os 11 motores de prática e o `ExamRunner` (1.059 linhas, 9 widgets, 47 hooks) não têm teste de UI. O E2E offline vive em `/tmp`, fora do repo. | `grep "render("` em `src/test`; `projeto 5` cita `/tmp/browser/test_offline.py`. |
| D6 | **Higiene do repositório e do contexto do agente.** `.opencode/` carrega ≈ 16 MB de artefatos de trabalho (`proj4-work` 8,4 MB, `proj6-work` 2,7 MB, `pte-json` 2,5 MB **obsoleto** — 2.032 q vs 2.536 —, `__pycache__`); `AGENTS.md` com 72 KB (diz Vite 7; o projeto usa 8.1.5; paths `/dev-server`); README é boilerplate. O próprio `metodologia-skills` recomenda progressive disclosure — não aplicado ao `AGENTS.md`. | `du -sh`; `AGENTS.md §6`; `package.json`. |
| D7 | Acessibilidade **não auditada** (49 ocorrências de `aria-`/`role=` fora de `components/ui`; sem teste automatizado). | `grep`; nenhum axe no repo. **Não exercitado** nesta análise. |

## Direção
1. **Primeiro não perder nem distorcer o que o aluno já tem** (A, B1, B5): é o único tipo de erro que o aluno não consegue perceber sozinho.
2. **Depois medir com honestidade** (B2–B4, B6): rotular como *estimativa de treino* o que é heurística; separar o que o app mede (conteúdo) do que não mede (pronúncia).
3. **Depois alinhar à prova real** (C): 22 tipos, banco rastreável e sem duplicatas, simulado fiel.
4. **Em paralelo à base, tornar entregável e seguro de evoluir** (D): CI, testes de tela, offline real, higiene.

## Ordem lógica (fases candidatas para PRD/roadmap)
0. **Planejar o PRD** + confirmar fontes oficiais da Pearson (formato 22 tasks, tempos, matriz tarefa×habilidade).
1. **Rede de segurança** (sem mudar comportamento): scripts, 1 lockfile, CI, testes de caracterização (os probes de A1/A2/B5 viram `it.fails`), testes de componente dos motores, Playwright 390×844 e offline commitados.
2. **Integridade do progresso (storage v6)** — A1–A5. *Vem antes de tudo que mexe em dados do aluno.*
3. **SRS correto** — B5. Depende do snapshot da Fase 2.
4. **Medição honesta** nos motores de fala/escrita — B4, B6 (+ detector local de frases-modelo/repetição, pois a Pearson passou a enviar para revisão humana respostas sinalizadas como "template" — fonte secundária; confirmar).
5. **Escala, alvo e prontidão** — B1–B3. Depende da matriz oficial (Fase 0) e dos subscores (Fase 4).
6. **Banco: proveniência, duplicatas, payload** — C2–C4, D2. A fusão de duplicatas migra `qid` no progresso → depois da Fase 2.
7. **Offline real e PWA** — D1 (+ manifest/descrição derivados). Depois da Fase 6 (payload final).
8. **Novos tipos (22/22)** — C1. Dependente de dados; reaproveita os motores de Retell Lecture/SST.
9. **Simulado fiel** — C5, D5 (extrair widgets do `ExamRunner` antes de alterar). Depende das Fases 5 e 8.
10. **A11y, mobile, higiene e fechamento** — D6, D7.

## Decisões delegadas ao PRD (o agente decide e registra; padrão recomendado entre parênteses)
- Onde guardar o log: manter `localStorage` com snapshot + agregados, ou IndexedDB nativo para o log (**padrão: snapshot em localStorage; IndexedDB só se o tamanho medido exigir**; zero deps).
- Como comparar alvo vs. desempenho (**padrão: converter a média interna em *estimativa* na escala 10–90 por habilidade e comparar nessa escala; sempre rotulada "estimativa de treino", nunca "previsão oficial"**).
- Itens `ai-generated-prediction`/`seed`: (**padrão: não apagar; rotular, filtrar e reduzir peso na prontidão; opção "só fontes verificadas" nos Ajustes**).
- Pronúncia: (**padrão: exibir "não avaliada" em vez de nota inventada; proxy por `confidence` só se rotulado como proxy**).
- Describe Image: (**padrão: só integrar com imagens acessíveis e licença clara; senão manter "em breve" com motivo registrado**).

## Restrições permanentes (herdadas)
- Sem auth, sem nuvem, 100% local; backup JSON versionado continua o envelope (v5 → v6 com migração sem perda).
- TTS/STT pela Web Speech API; **zero dependências de runtime novas**; dev-deps só para testes/CI (Playwright, axe).
- Pack "studie-plus" continua 100% off por padrão; áudio fala sempre o inglês original (regra do projeto 7).
- Dados novos: só fonte verificável, sem login, sem conteúdo gerado por IA, com dedupe contra o banco **e** entre fontes (regras do projeto 4).
- Nada de números fixos em UI/manifest quando existir constante derivada (regra do projeto 5).

## Escopo negativo
- Não construir correção por IA/LLM nem usar AI Gateway. Não prometer nota oficial do PTE. Não apagar questões sem registro. Não adicionar contas, sincronização em nuvem ou pagamento.

## Critérios de aceite globais
- Nenhum cenário de A1–A3 e B5 reproduz (os probes viram testes verdes).
- Nenhuma tela mistura % com escala 10–90; toda estimativa vem rotulada e com nível de confiança.
- O catálogo mostra 22 tipos **ou** registra, por tipo, por que ainda não (fonte reprovada pelo gate).
- Offline: tipo **nunca visitado** funciona após "Baixar para offline".
- CI verde em clone limpo (`typecheck + lint + vitest + build`); testes de tela para todos os motores; axe sem violações graves nas rotas principais.
- Ao concluir: avisar o monitor ("remix finalizado (+N da etapa do roadmap)") e, se todos os `project{N}` estiverem concluídos, planejar o próximo.

## Limites desta análise (para o PRD não tratar como fato)
Mic real, a11y e layout mobile **não foram exercitados** aqui; o formato de 22 tasks e a matriz tarefa×habilidade vêm de fontes secundárias; comportamento de Safari/Chrome (limpeza de armazenamento, STT em servidor) é conhecimento de plataforma a confirmar.
