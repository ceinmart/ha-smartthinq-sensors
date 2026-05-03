# AGENTS.md — Instruções para Codex

## Objetivo do trabalho

Este repositório é um fork da integração Home Assistant `ollo69/ha-smartthinq-sensors`.

O objetivo é implementar melhorias incrementais para aparelhos LG ThinQ AC/HVAC, usando como referência técnica o arquivo `CODEX.md`.

Não implemente todas as fases de uma vez. Trabalhe em etapas pequenas, testáveis e revisáveis.

## Ordem obrigatória de leitura

Antes de alterar código, leia:

1. `CODEX.md`
2. `custom_components/smartthinq_sensors/wideq/const.py`
3. `custom_components/smartthinq_sensors/wideq/devices/ac.py`
4. `custom_components/smartthinq_sensors/switch.py`
5. `custom_components/smartthinq_sensors/sensor.py`
6. `custom_components/smartthinq_sensors/climate.py`

## Regras de implementação

- Preserve o comportamento atual da integração.
- Não remova entidades existentes.
- Não renomeie entidades existentes sem necessidade explícita.
- Novas entidades devem ser criadas somente quando o recurso for detectado pelo modelo/dispositivo.
- Evite hardcode para modelos específicos brasileiros.
- Use detecção dinâmica baseada em `model_info`, `available_features` e/ou status do dispositivo.
- Não trate a mera existência de um campo em `Value.airState.*` ou no `device_status` como prova suficiente de suporte real; muitos model JSONs da LG trazem campos genéricos para recursos que o aparelho não possui fisicamente.
- Prefira marcadores explícitos em `support.*` do `model_info` para criar entidades habilitadas por padrão. Exemplos confirmados:
  - Auto Dry: `support.racMode` contém `@AUTODRY`.
  - UVnano: `support.pacModeExt` contém `@UV_NANO`.
- Se só existir o campo de estado/comando, mas não houver marcador de suporte confiável, deixe a entidade desabilitada por padrão ou mantenha como pendência documentada.
- Quando houver dúvida sobre suporte real, deixe a entidade desabilitada por padrão ou documente como pendência.
- Não inclua dados sensíveis de diagnósticos reais em commits, testes ou documentação.

## Dados sensíveis proibidos

Nunca commite:

- tokens
- `deviceId`
- `userNo`
- `ssid`
- URLs assinadas da LG
- `modelJsonUri`
- `appModuleUri`
- `langPackProductTypeUri`
- `langPackModelUri`
- snapshots completos contendo identificadores pessoais ou de conta

## Estilo de código

- Siga o padrão atual do projeto.
- Use nomes de entidades em inglês.
- Use `AirConditionerFeatures` para novas features de AC.
- Use `ThinQSwitchEntityDescription` para switches.
- Use `SensorEntityDescription`, `SelectEntityDescription` ou `NumberEntityDescription` apenas quando o tipo real do recurso justificar.
- Mantenha compatibilidade com aparelhos que não possuam o recurso.
- Prefira mudanças pequenas por commit.

## Fases de trabalho

### Fase 1 — documentação

Primeiro implemente apenas documentação, sem alteração de comportamento.

Criar ou atualizar:

```text
docs/ac_modeljson_features.md
```

Esse documento deve conter:

- Campo LG.
- Valores conhecidos.
- Função provável.
- Correspondência com manual LG.
- Entidade Home Assistant sugerida.
- Confiança.
- Status de implementação.

### Fase 2 — Auto Dry e UVnano

Após revisão da Fase 1, implementar apenas:

- `Auto dry`
- `UVnano`

Não implemente `Power Save`, `Energy Control`, `Deep Sleep`, `Jet select` ou modos obscuros nesta fase.

### Fases seguintes

Seguir a ordem definida no `CODEX.md`.

## Ambiente Python

Este projeto não usa `.venv`.

No Windows, o executável `ruff` pode não estar disponível no PATH, mesmo com o pacote instalado via pip.

Sempre use:

```powershell
python -m ruff check custom_components/smartthinq_sensors
```

## Comandos de validação

Execute os comandos disponíveis no projeto. Se algum comando não existir, informe isso no resumo.

Tente, nesta ordem:

```bash
python -m compileall custom_components/smartthinq_sensors
python -m pytest
ruff check custom_components/smartthinq_sensors
```

Se houver `tox`, `nox`, `pre-commit` ou configuração equivalente no repositório, use o mecanismo recomendado pelo projeto.

## Cópia para teste no Home Assistant

Ao finalizar alterações em arquivos da integração, exiba comandos PowerShell
para copiar apenas os arquivos alterados para o Home Assistant de teste.

Destino fixo da integração no Home Assistant:

```powershell
\\box\dados\ha\custom_components\smartthinq_sensors
```

## Resumo obrigatório ao final de cada tarefa

Ao finalizar, informe:

- fase executada;
- arquivos alterados;
- comportamento alterado;
- comandos de validação executados;
- erros ou limitações encontradas;
- próximos passos sugeridos.

## Restrições importantes

- Não faça refatorações amplas fora do escopo da fase atual.
- Não altere autenticação, login, token ou fluxo de configuração.
- Não altere polling ou coordinator sem necessidade.
- Não substitua a arquitetura atual por ThinQ Connect oficial nesta etapa.
- Não implemente recursos obscuros como `iceValley`, `flowShower` ou `flowForest` sem validação explícita.
