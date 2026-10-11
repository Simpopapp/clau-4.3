# Roadmap — projeto 9: Integridade, medição honesta e fidelidade à prova real
> Planejamento: `.opencode/project9.md` · PRD: `.opencode/prd-project9.md` (criado na Fase 0) · Ordem de dependência, sem estimativa de prazo.
## Fase 0 — Planejar o PRD
- [ ] Criar `prd-project9.md` (500–1000 linhas) a partir de `project9.md`, confirmando cada evidência no código; decidir as "Decisões delegadas" e registrá-las no PRD
- [ ] Confirmar na fonte oficial da Pearson (Score Guide): 22 tasks, tempos por tipo e matriz tarefa×habilidade; registrar fontes no PRD
- [ ] Gates: PRD cobre as Fases 1–10 com escopo + critérios de aceite; nenhum código alterado
## Fase 1 — Rede de segurança (sem mudar comportamento)
- [ ] `package.json`: scripts `test`/`typecheck`/`check`; um único lockfile reproduzível (`npm ci` ou `bun install --frozen-lockfile` passa); CI em `.github/workflows`
- [ ] Probes de A1/A2/B5 viram `it.fails`; testes de componente por motor; Playwright 390×844 (rotas + offline) commitado no repo
- [ ] Gates: CI verde em clone limpo · 258+ testes · nenhum comportamento alterado
## Fase 2 — Integridade do progresso (storage v6)
- [ ] Snapshot SRS por questão independente do log; log truncável + agregados diários; migração v5→v6 semeia o snapshot **antes** de qualquer corte; datas sem passo fixo de 24 h
- [ ] Quarentena de estado ilegível; versão futura nunca sobrescreve; import com prévia + substituir|mesclar + snapshot de segurança + desfazer; escrita coalescida, aviso preventivo de quota, `storage.persist()`, lembrete de backup
- [ ] Gates: `it.fails` de A1/A2 viram verdes · round-trip de backup v5→v6 sem perda · vitest/tsc/lint/build
## Fase 3 — SRS correto
- [ ] Caixa só avança se a questão estava vencida (prática antecipada conta, não promove); lapso rebaixa gradualmente; `PASS_SCORE` calibrado por tipo e documentado; "dominada" reavaliada
- [ ] Fila do dia e plano usam o novo SRS; migração recalcula caixas do histórico sem perder tentativas
- [ ] Gates: simulações determinísticas (cram, lapso, atraso) · `it.fails` de B5 verde
## Fase 4 — Medição honesta (fala e escrita)
- [ ] Extras/repetições entram no score; subscores Conteúdo · Fluência (proxy) · Pronúncia ("não avaliada" ou proxy rotulado); aviso de que o reconhecedor "corrige" a fala; Ajustes sem "nenhum servidor"
- [ ] Detector local de frases-modelo/repetição em Essay/SWT/SST (sem IA); aviso do uso do microfone no 1º uso
- [ ] Gates: golden tests de scoring com casos anotados · UI 390×844 com subscores
## Fase 5 — Escala, alvo e prontidão
- [ ] Matriz tarefa×habilidade (fonte da Fase 0) → pesos derivados; estimativa 10–90 por habilidade (rotulada) e alvo comparado na mesma escala
- [ ] Prontidão sem cobertura de banco no núcleo: desempenho recente ponderado + tendência + confiança (n mínimo); cobertura/variedade secundária; relatório semanal atualizado
- [ ] Gates: nenhuma tela mistura % com 10–90 · estado "confiança baixa" com poucos dados · testes do mapeamento
## Fase 6 — Banco: proveniência, duplicatas, payload
- [ ] Proveniência (verificada/comunitária/gerada por IA/seed) com selo, filtro "só verificadas" e peso na prontidão; relatório dos 647 itens de IA + 200 seed
- [ ] Dedupe fuzzy em todos os tipos (28 pares de SWT) com `canonicalId` que funde progresso; remover campos redundantes (WFD ≈ 51%); selo "banco pequeno" (≤ 6 itens)
- [ ] Gates: `bank-integrity` sem quase-duplicatas · progresso migrado sem perda · bundles medidos antes/depois
## Fase 7 — Offline real e PWA
- [ ] Precache gerado no build + "Baixar para offline" por seção/tipo com progresso; cache versionado por hash e aviso de nova versão; manifest/descrição derivados do banco
- [ ] `metrics-sync` cobre `public/manifest.webmanifest` e `sw.js`; E2E: tipo nunca visitado → baixar → offline
- [ ] Gates: Playwright offline passa · atualização do SW testada
## Fase 8 — Novos tipos (22/22)
- [ ] Fontes pelo protocolo do projeto 4 para Summarize Group Discussion, Respond to a Situation, Describe Image (+ tipos com ≤ 6 itens); por tipo: integrar se aprovado, senão "em breve" com motivo registrado
- [ ] Motores reaproveitando Retell Lecture/SST (SGD com áudio multi-voz via TTS; RTS com 10 s de preparo + 40 s); Describe Image só com imagens acessíveis e licença clara
- [ ] Gates: gate de dados por fonte · vitest dos motores novos · catálogo com 22 tipos ou justificativa registrada
## Fase 9 — Simulado fiel
- [ ] Extrair os 9 widgets do `ExamRunner` para arquivos próprios com testes; blueprint oficial (contagens, ordem, tempos); rotular "mini-simulado" × "simulado completo"
- [ ] Resultado por habilidade (Fase 5), contra o alvo e contra simulados anteriores
- [ ] Gates: ordem/tempos conferidos com a fonte oficial · Playwright de um simulado de seção ponta a ponta
## Fase 10 — A11y, higiene e fechamento
- [ ] axe automatizado + teclado nos motores; arquivar `proj*-work`, `pte-json` obsoleto e `__pycache__`; enxugar `AGENTS.md` (Vite 8, paths, progressive disclosure); README real
- [ ] Fechamento: gates globais, `AGENTS.md` (estado atual), aviso ao monitor "remix finalizado (+N …)", checagem periódica por novos arquivos/fases
- [ ] Gates: CI verde · axe sem violações graves · vitest/tsc/lint/build · preview 8080
