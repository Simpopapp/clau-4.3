# Roadmap — projeto 9: Integridade, medição honesta e fidelidade à prova real
> Planejamento: `.opencode/project9.md` · PRD: `.opencode/prd-project9.md` (criado na Fase 0) · Ordem de dependência, sem estimativa de prazo.
## Fase 0 — Planejar o PRD
- [ ] Criar `prd-project9.md` (500–1000 linhas) a partir de `project9.md`, confirmando cada evidência no código; decidir e registrar as "Decisões delegadas"
- [ ] Confirmar na fonte oficial da Pearson: 22 tasks, tempos por tipo e matriz tarefa×habilidade
- [ ] Gates: PRD cobre as Fases 1–9 com escopo + critérios de aceite
## Fase 1 — Integridade do progresso (storage v6)
- [ ] Snapshot SRS por questão independente do log; log truncável + agregados diários; migração v5→v6 semeia o snapshot **antes** de qualquer corte; datas sem passo fixo de 24 h
- [ ] Quarentena de estado ilegível; versão futura nunca sobrescreve; import com prévia + substituir|mesclar + cópia de segurança + desfazer; escrita coalescida, aviso preventivo de quota, `storage.persist()`, lembrete de backup
- [ ] Gates: probes A1/A2 viram testes verdes · round-trip de backup v5→v6 sem perda · vitest/tsc/lint/build
## Fase 2 — SRS correto
- [ ] Caixa só avança se a questão estava vencida (prática antecipada conta, não promove); lapso rebaixa gradualmente; `PASS_SCORE` calibrado por tipo; "dominada" reavaliada
- [ ] Fila do dia e plano usam o novo SRS; migração recalcula caixas do histórico sem perder tentativas
- [ ] Gates: simulações determinísticas (prática seguida, lapso, atraso) · probe B5 verde
## Fase 3 — Medição honesta (fala e escrita)
- [ ] Extras/repetições entram no score; subscores Conteúdo · Fluência (proxy) · Pronúncia ("não avaliada" ou proxy rotulado); aviso de que o reconhecedor "corrige" a fala; Ajustes sem "nenhum servidor"
- [ ] Detector local de frases-modelo/repetição em Essay/SWT/SST (sem IA); aviso do uso do microfone no 1º uso
- [ ] Gates: casos anotados de scoring · UI mobile 390×844 com subscores
## Fase 4 — Escala, alvo e prontidão
- [ ] Matriz tarefa×habilidade → pesos derivados; estimativa 10–90 por habilidade (rotulada) e alvo comparado na mesma escala
- [ ] Prontidão sem cobertura de banco no núcleo: desempenho recente ponderado + tendência + confiança (n mínimo); cobertura/variedade secundária; relatório semanal atualizado
- [ ] Gates: nenhuma tela mistura % com 10–90 · estado "confiança baixa" com poucos dados
## Fase 5 — Banco: proveniência, duplicatas, payload
- [ ] Proveniência (verificada/comunitária/gerada por IA/seed) com selo, filtro "só verificadas" e peso na prontidão
- [ ] Dedupe fuzzy em todos os tipos (28 pares de SWT) com `canonicalId` que funde progresso; remover campos redundantes (WFD ≈ 51%); selo "banco pequeno" (≤ 6 itens)
- [ ] Gates: zero quase-duplicatas · progresso migrado sem perda · bundles medidos antes/depois
## Fase 6 — Offline real
- [ ] Pré-cache gerado no build + "Baixar para offline" por seção/tipo com progresso; cache versionado e aviso de nova versão; descrição do manifest derivada do banco
- [ ] Gates: Playwright offline passa em tipo nunca visitado após o download · atualização do SW testada
## Fase 7 — Novos tipos (22/22)
- [ ] Fontes verificáveis para Summarize Group Discussion, Respond to a Situation, Describe Image (+ tipos com ≤ 6 itens); por tipo: integrar se aprovado, senão "em breve" com motivo
- [ ] Motores reaproveitando Retell Lecture/SST (SGD com áudio multi-voz via TTS; RTS 10 s de preparo + 40 s); Describe Image só com imagens acessíveis e licença clara
- [ ] Gates: gate de dados por fonte · vitest dos motores novos · catálogo com 22 tipos ou justificativa
## Fase 8 — Simulado fiel
- [ ] Blueprint oficial (contagens, ordem, tempos); rotular "mini-simulado" × "simulado completo"
- [ ] Resultado por habilidade (Fase 4), contra o alvo e contra simulados anteriores
- [ ] Gates: ordem/tempos conferidos com a fonte oficial · Playwright de um simulado de seção ponta a ponta
## Fase 9 — Acessibilidade e mobile
- [ ] Auditoria de acessibilidade (axe) e teclado nos motores; indicar a voz TTS realmente usada; revisão 390×844
- [ ] Gates: sem violações graves nas rotas principais · vitest/tsc/lint/build · preview 8080
