# Timeline da Investigação

> Todos os horários abaixo são os efetivamente registrados durante o atendimento. Quando um horário exato não estava disponível nos documentos-fonte, isso é indicado explicitamente na tabela, em vez de ser presumido.
>
> Legenda de entidades anonimizadas: `Colaborador-01` (dono da conta comprometida) · `Responsável-01` / `Responsável-02` (responsáveis pela empresa vítima) · `Provedor-A` (provedor de e-mail) · `Contato-Provedor-A` (suporte técnico do provedor) · `Cliente-01`, `Cliente-02` (contatos externos que receberam o phishing) · `DESKTOP-COLAB01` (máquina do colaborador).

## 25/08 — Identificação do disparo e investigação inicial

| Data/Hora | Evento | Ação/Investigação | Evidência/Resultado |
| --- | --- | --- | --- |
| 25/08 09:58 | `Colaborador-01` envia e-mail "Comprovante fiscal do seu pedido" (atendimento nº 5188546, R$ 1.718,04, botão "Consultar documento") para `Cliente-01` | — | Primeiro disparo confirmado do padrão de phishing |
| 25/08 13:32 | `Cliente-01` responde informando não conseguir acessar o site do botão | Evidência preservada e revisada | Confirma que o link do botão estava ativo na mensagem original |
| 25/08 13:37 | O próprio `Colaborador-01` responde se retratando ("desconsiderar, estamos analisando com nosso TI") | — | Indício de que o disparo já havia sido percebido internamente antes da investigação formal |
| 25/08 13:42–15:00 | — | Investigação inicial dos e-mails suspeitos; comparação com e-mail legítimo de referência (baseline) | Confirmado padrão de golpe ("Consultar documento" → link malicioso) |
| 25/08 14:38 | — | Relatório inicial entregue a `Responsável-01` | Hipótese comunicada: possível comprometimento de conta |
| 25/08 15:00 | — | Documentação da demanda registrada; aguardando retorno de `Responsável-01` | Investigação seguia dependente de próximas evidências (não parada) |
| 25/08 15:56 | `Responsável-01` confirma acionamento formal do `Provedor-A` | Solicitados logs SMTP, IPs de origem, confirmação de uso de credenciais, lista de destinatários | Aguardando retorno do provedor |
| 25/08 16:09 | — | Recomendado a `Responsável-01` avisar proativamente os clientes atingidos | — |
| 25/08 16:16 | — | Texto padrão de aviso aos clientes elaborado | — |
| 25/08 16:19 | E-mails suspeitos desaparecem da pasta "Enviados" de `Colaborador-01` | — | Indício de ação de quarentena automática do `Provedor-A` após o acionamento |
| 25/08 16:20–17:57 | — | Retomada da análise: revisão de conversa com `Colaborador-01` (banner "remetente externo"); OSINT (Shodan) sobre infraestrutura do `Provedor-A`; OSINT (AbuseIPDB) sobre IP de origem do disparo | Duas hipóteses levantadas sobre o banner "remetente externo" (config. SPF/DKIM/DMARC vs. origem de rede atípica) — nenhuma confirmada nesta etapa |
| — | Recebido bounce (falha de entrega) para `Cliente-02` | Análise dos cabeçalhos de transporte do bounce | Confirmado padrão do mesmo phishing enviado a múltiplos destinatários (não caso isolado); cabeçalhos revelam HELO `DESKTOP-COLAB01` a partir de IP residencial nacional, autenticado como a conta comprometida; identificado domínio malicioso `maventra.club` |
| 25/08 18:00 | — | Documentação do dia encerrada | Próxima etapa: elaboração de resumo para `Responsável-01` |
| 25/08 20:35 | Retorno formal do `Provedor-A` (via `Contato-Provedor-A`) com logs de envio | Análise dos logs de autenticação/envio de 24 e 25/08 | Provedor informa que a senha da conta foi trocada *(ver nota de inconsistência ao final)*; envios ocorreram em horário comercial, levando o provedor a apontar hipótese de malware local como mais provável que roubo externo de credenciais |

## 26/08 — Investigação do endpoint e confirmação do malware

| Data/Hora | Evento | Ação/Investigação | Evidência/Resultado |
| --- | --- | --- | --- |
| 26/08 09:12 | `Colaborador-01` disponibiliza acesso remoto à máquina | — | — |
| 26/08 09:24 | — | Início de varredura (msert) e checagem de itens de inicialização (Autoruns); retorno adicional recebido do `Provedor-A` | — |
| 26/08 09:30–09:46 | — | Análise do Autoruns em busca de indícios de infostealer | Identificado arquivo suspeito em pasta não convencional de `AppData\Local` |
| 26/08 10:13 | — | Consulta de reputação do hash do arquivo suspeito; colaborador confirma nunca ter usado Python | Hash corresponde a um componente legítimo do interpretador Python (não malicioso por si só), mas caminho de instalação é atípico; identificado script auxiliar em Base64 executado por esse binário |
| 26/08 10:20 | — | Identificados arquivos gerados automaticamente na pasta suspeita (nomeados com base no e-mail da conta); identificado processo oculto em pasta disfarçada como diretório da Microsoft | Confirma persistência ativa e possível coleta de dados de contatos locais |
| 26/08 10:30 | — | msert identifica 15 arquivos suspeitos, análise em andamento | — |
| 26/08 10:36 | — | msert conclui varredura sem detectar vírus | Indício de que a varredura padrão, isoladamente, não detectou o componente (detecção veio da análise manual) |
| 26/08 10:48 | — | Processo "vigia" (pasta disfarçada como Microsoft) finalizado manualmente | — |
| 26/08 10:49 | Pasta maliciosa é recriada automaticamente | — | Confirma existência de segundo mecanismo de persistência |
| 26/08 10:51–10:53 | — | Identificado e finalizado (taskkill) o processo principal do componente malicioso (PID identificado) | Após esse momento, pasta maliciosa deixa de ser repopulada |
| 26/08 11:18 | — | Monitoramento da máquina para confirmar ausência de nova persistência | Nenhuma nova criação de artefatos observada |
| 26/08 (após 11:18) | — | Reinicialização da máquina; confirmação de ausência de novos artefatos; arquivos maliciosos movidos para quarentena local | — |
| 26/08 11:38 | — | Início de FullScan (msert) para validação adicional; pastas monitoradas confirmadas sem recriação | — |
| 26/08 11:46–12:26 | — | Revisão dos logs do `Provedor-A`; organização da lista de destinatários (502 únicos / 488 externos) | Lista pronta para uso no aviso oficial aos clientes |
| 26/08 12:27 | — | `Responsável-01` e `Colaborador-01` comunicados sobre status (máquina, malware confirmado/removido, lista de destinatários) | Investigação aguardando: conclusão do FullScan e definição de próximos passos |
| 26/08 15:15–15:29 | — | Reconexão para verificação final: revisão do log do msert (sem novos arquivos infectados), nova checagem de Autoruns e de processos ativos | Ambiente considerado restaurado à normalidade |
| 26/08 15:30–16:50 | — | Levantamento de quais clientes já haviam sido avisados manualmente por `Colaborador-01`; ajuste e importação da lista restante para envio do aviso oficial | Lista final pronta para disparo |
| 26/08 17:00–17:04 | — | Atualização enviada a `Responsável-01` | Dia técnico encerrado |
| 26/08 17:50 | `Responsável-02` relata que o `Provedor-A` bloqueou o envio de mais de 600 e-mails adicionais disparados pela conta, e que a conta autenticou a partir de 3 IPs diferentes | — | Reforça a escala do incidente |
| 26/08 18:00–18:13 | — | Contato telefônico com `Contato-Provedor-A`: confirmação de que não houve mais disparos à tarde; confirmação de impossibilidade de habilitar MFA para o tipo de serviço contratado | `Contato-Provedor-A` informa que a senha da conta **ainda não havia sido trocada** *(ver nota de inconsistência)* |
| 26/08 18:14 | — | Combinado retorno telefônico no dia seguinte para reconfirmação | `Responsável-02` questiona possibilidade de propagação para outra rede — esclarecido verbalmente |

## 27/08 — Validação pós-contenção

| Data/Hora | Evento | Ação/Investigação | Evidência/Resultado |
| --- | --- | --- | --- |
| 27/08 10:48 | `Colaborador-01` relata lentidão no recebimento de e-mails | Investigação da causa da lentidão | Identificada intermitência/latência de entrega, não relacionada a novo comprometimento |
| 27/08 10:48–11:05 | — | Forçado recebimento manual de mensagens pendentes; ausência de novos bounces observada | Indício adicional de normalização do ambiente |
| 27/08 14:00–14:05 | — | Contato telefônico com `Contato-Provedor-A`; consulta aos logs | Confirmado que não houve mais disparos suspeitos; hipótese de disparo local (sem vazamento de credenciais) reforçada |
| 27/08 14:10 | — | `Responsável-01`, `Responsável-02` e `Colaborador-01` atualizados sobre a situação final | Caso considerado normalizado |

### Progressão resumida

`Identificação do disparo (25/08 09:58–13:37)` → `Hipótese de spoofing vs. conta autenticada (bounce, 25/08)` → `Coleta de evidências (OSINT + logs do provedor, 25/08 tarde–noite)` → `Correlação (horário comercial + IPs nacionais → hipótese de malware local, 25/08 20:35)` → `Descoberta (componente malicioso com dupla persistência no endpoint, 26/08 09:46–10:53)` → `Contenção (remoção, quarentena, reset de senha solicitado, 26/08)` → `Validação (novas varreduras + confirmação do provedor, 26–27/08)`

### Nota de inconsistência entre fontes

Os documentos-fonte registram duas informações conflitantes sobre a troca de senha da conta comprometida: em 25/08 (20:35) o provedor teria informado que a senha já havia sido alterada; em 26/08 (18:13) o mesmo provedor informou que a senha **ainda não havia sido trocada**. Essa divergência não foi resolvida na documentação original e está registrada aqui como está, sem tentativa de arbitrar qual das duas informações está correta.

### Nota sobre verificações de segurança recorrentes

O endpoint envolvido foi submetido a mais de uma varredura de segurança ao longo da investigação (varredura inicial com detecção parcial, varredura manual complementar via Autoruns/Gerenciador de Tarefas, e FullScan de validação). A primeira varredura automatizada, isoladamente, não sinalizou o componente como malicioso — a identificação dependeu de análise manual combinada (caminho de instalação atípico, ausência de uso prévio de Python pelo usuário, comportamento de repopulação da pasta). Isso é registrado para deixar claro que a ferramenta automatizada não falhou de forma isolada; a técnica de abuso de binário legítimo dificulta a detecção por assinatura tradicional.
