# Especificação: Weather App

## Estado da spec

- **Versão:** 1.1
- **Baseline:** busca de cidade e clima atual
- **Última decisão:** acrescentar previsão diária de 7 dias; previsão horária permanece fora de escopo

## Escopo

Aplicação web estática para buscar cidades, consultar o clima atual e a
previsão diária dos próximos 7 dias usando a Open-Meteo, sem autenticação.

## Fora de escopo

- Previsão horária.
- Geolocalização automática.
- Histórico persistido e cidades favoritas.
- Notificações e múltiplos idiomas.

## Funcionalidades e Critérios de Aceite

### F1: Busca de cidade

- **CA1.1:** DADO que o campo está vazio, ENTÃO o botão "Buscar" permanece desabilitado.
- **CA1.2:** DADO um nome válido, QUANDO buscar, ENTÃO até cinco localizações são apresentadas para seleção.
- **CA1.3:** DADO que nenhuma localização foi encontrada, ENTÃO a mensagem "Nenhuma cidade encontrada." é apresentada.
- **CA1.4:** DADO uma falha de geocodificação, ENTÃO uma mensagem de erro é apresentada.

### F2: Clima atual

- **CA2.1:** DADO uma localização selecionada, ENTÃO a temperatura atual em Celsius é apresentada.
- **CA2.2:** DADO uma localização selecionada, ENTÃO a sensação térmica é apresentada.
- **CA2.3:** DADO uma localização selecionada, ENTÃO a condição climática possui descrição e representação visual.
- **CA2.4:** DADO uma localização selecionada, ENTÃO vento em km/h e umidade em percentual são apresentados.
- **CA2.5:** DADO uma consulta em andamento, ENTÃO um estado de carregamento é apresentado.
- **CA2.6:** DADO uma falha na consulta do clima, ENTÃO uma mensagem de erro é apresentada.

### F3: Conversão de temperatura

- **CA3.1:** A conversão segue $F = (C \times 9/5) + 32$.
- **CA3.2:** 0°C corresponde a 32°F.
- **CA3.3:** 100°C corresponde a 212°F.
- **CA3.4:** -40°C corresponde a -40°F.

### F4: Condições WMO

- **CA4.1:** O código WMO 0 representa "Céu limpo".
- **CA4.2:** O código WMO 95 representa "Tempestade".
- **CA4.3:** Um código desconhecido representa "Condição desconhecida".

### F5: Previsão diária de 7 dias

- **CA5.1:** DADO uma localização selecionada, QUANDO o clima for carregado, ENTÃO exatamente 7 dias de previsão são apresentados.
- **CA5.2:** DADO um dia da previsão, QUANDO ele for apresentado, ENTÃO suas temperaturas máxima e mínima são exibidas em Celsius.
- **CA5.3:** DADO um dia da previsão, QUANDO ele for apresentado, ENTÃO sua condição climática WMO possui descrição e representação visual.

## Delta F5: previsão diária de 7 dias

### Análise de impacto

| Superfície | Impacto mínimo |
|---|---|
| `WeatherData` | Acrescentar `daily` com arrays de data, máxima, mínima e código WMO |
| Open-Meteo Forecast | Solicitar `daily=temperature_2m_max,temperature_2m_min,weather_code`, `timezone=auto` e `forecast_days=7` |
| `WeatherCard` | Preservar a região de clima atual e acrescentar uma região acessível com sete entradas diárias |
| Testes | Usar fixtures com valores distintos e provar serviço, componente e jornada E2E |

Os arrays de `daily` são relacionados pelo mesmo índice: `time[i]`,
`temperature_2m_max[i]`, `temperature_2m_min[i]` e `weather_code[i]`
representam o mesmo dia.

### Decisões do delta

| Decisão | Escolha | Alternativa descartada | Motivo |
|---|---|---|---|
| Modelo Diário | Estender `WeatherData` com os arrays retornados pela API | Criar uma segunda árvore de estado | Mantém clima atual e previsão na mesma resposta |
| Período | `forecast_days=7` | Cortar um retorno maior na UI | O contrato é aplicado na fronteira externa |
| Campos | Solicitar apenas máxima, mínima e `weather_code` | Solicitar todos os campos diários | Evita dados sem requisito |
| Apresentação | Estender `WeatherCard` | Criar outro fluxo de seleção | Preserva a jornada existente |

### Estratégia de testes do delta

| Critério | Serviço | Componente | E2E |
|---|---|---|---|
| CA5.1 | Prova `forecast_days=7` e sete datas retornadas | Prova exatamente sete entradas | Prova sete entradas após busca e seleção |
| CA5.2 | Prova os arrays `temperature_2m_max` e `temperature_2m_min` | Prova máxima e mínima associadas a cada dia | Prova máxima e mínima na jornada |
| CA5.3 | Prova o array `weather_code` | Prova descrição e representação WMO por dia | Prova a condição na jornada |

A cobertura de F2 permanece nos testes existentes de serviço, componente e
E2E para detectar regressão do clima atual.

## Contrato observável

- Campo de busca: `searchbox` com nome acessível "Nome da cidade".
- Ação de busca: botão "Buscar", desabilitado quando o campo está vazio.
- Resultado: botão com nome acessível "Selecionar {cidade}, {país}".
- Clima atual: região com nome acessível "Clima atual para {cidade}".
- Previsão diária: região com nome acessível "Previsão de 7 dias para {cidade}", contendo exatamente sete entradas.
- Erros: elementos com `role="alert"`.

## Histórico

| Versão | Mudança | Critérios |
|---|---|---|
| 1.0 | Baseline de busca e clima atual | CA1.1–CA4.3 |
| 1.1 | Previsão diária dos próximos 7 dias | CA5.1–CA5.3 |