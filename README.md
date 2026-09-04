# Incident Response — Comprometimento de Conta Corporativa e Disparo de Phishing em Massa

> Case anonimizado, baseado em um incidente real. Todos os nomes de empresas, colaboradores, provedores e endereços de e-mail/IP foram substituídos por identificadores genéricos (`Empresa-A`, `Colaborador-01`, `Provedor-A`, etc.). Indicadores diretamente relacionados ao ataque (domínio malicioso, hash, padrões de e-mail) foram preservados, pois possuem valor técnico e não identificam a vítima.

## Objetivo

Investigar a origem de um disparo em massa de e-mails de phishing partindo de uma conta corporativa legítima, determinar se houve comprometimento de credenciais ou execução de malware local, conter o incidente e validar a normalização do ambiente, documentando o processo de investigação como ele realmente ocorreu, incluindo hipóteses levantadas e depois confirmadas ou descartadas.

## Resumo do Incidente

A conta de e-mail de um colaborador (`Colaborador-01`) da `Empresa-A` foi identificada enviando mensagens simulando "comprovantes fiscais" e "documentos disponíveis" para clientes e fornecedores externos, contendo um botão que direcionava a um domínio malicioso.

A investigação seguiu, em linhas gerais, esta progressão:

1. Identificação do disparo a partir da resposta de um cliente externo que não conseguiu acessar o link recebido.
2. Análise dos e-mails suspeitos e de um bounce (falha de entrega) que confirmou a existência de múltiplos disparos, não um caso isolado.
3. Extração dos cabeçalhos de transporte do bounce, revelando que o envio partiu de uma submissão SMTP autenticada com as credenciais reais da conta, não um spoofing simples de "From".
4. Levantamento OSINT complementar (infraestrutura do provedor de e-mail e reputação do IP de origem) para sustentar hipóteses sobre a natureza do comprometimento.
5. Acionamento formal do provedor de e-mail, com pedido de logs de autenticação/envio.
6. Retorno do provedor com os logs, indicando que os envios ocorreram em horário comercial e a partir de IPs nacionais, o que apontou para a hipótese de malware local, e não roubo de credenciais por terceiro externo.
7. Verificação remota do endpoint do colaborador, com identificação e remoção de um componente malicioso com dupla persistência.
8. Levantamento da lista completa de destinatários atingidos junto ao provedor e organização do aviso oficial aos clientes.
9. Validação pós-contenção, com novas varreduras e confirmação, junto ao provedor, de que os disparos haviam cessado.

## Investigação

A investigação evoluiu de forma incremental, com as hipóteses sendo reforçadas ou descartadas conforme novas evidências chegavam:

- **Hipótese inicial**: poderia se tratar de spoofing simples do campo "From", sem uso real da conta.
- **Evidência que refutou a hipótese inicial**: os cabeçalhos do e-mail devolvido (bounce) mostraram uma submissão SMTP autenticada com as credenciais reais da conta, passando pela infraestrutura legítima do provedor — ou seja, a conta foi de fato usada, não apenas falsificada.
- **Hipótese concorrente nº 1**: comprometimento de credenciais por terceiro externo (ex.: senha vazada/força bruta).
- **Hipótese concorrente nº 2**: malware local na máquina do colaborador, utilizando a sessão/autenticação já existente para disparar e-mails.
- **Evidências que pesaram a favor da hipótese nº 2**: os envios ocorreram dentro do horário comercial (atípico para ataques externos clássicos, que costumam ocorrer fora do expediente); os IPs de autenticação eram todos nacionais e variavam conforme a rede em que o notebook do colaborador se conectava (típico de execução local, não de infraestrutura de ataque centralizada); consulta OSINT ao IP de origem mostrou tratar-se de uma linha residencial (DSL), que é um indício circunstancial, não conclusivo.
- 
- **Confirmação**: a varredura remota do endpoint identificou um componente malicioso com persistência dupla em `AppData`, replicando exatamente o comportamento necessário para sustentar os disparos observados.

## Achados

**Confirmado:**
- A conta corporativa foi utilizada, via submissão SMTP autenticada, para enviar phishing em massa a contatos externos (clientes e fornecedores) da empresa, em pelo menos dois dias consecutivos.
- Havia um componente malicioso instalado no endpoint do colaborador, com dois mecanismos distintos de persistência.
- O componente identificado abusava de um binário legítimo (interpretador Python) para executar um script malicioso ofuscado em Base64 em um txt, o hash do binário em si corresponde a um componente legítimo; o comportamento malicioso estava no script executado por ele, não no binário.
- Não foi identificado indício de propagação para outras máquinas da empresa.
- Os disparos cessaram após a remoção do componente malicioso do endpoint, confirmado tanto pela verificação técnica quanto pelo retorno do provedor de e-mail.

**Indício (não conclusivo):**
- O padrão de horário comercial e a variação de IPs nacionais são indícios que reforçam a hipótese de execução local do malware, mas não eliminam por completo a possibilidade de uso indevido das credenciais por terceiros.

**Não determinado:**
- A origem exata da infecção inicial (o relatório aponta indícios de que seria anterior ao período analisado, sem evidências suficientes para precisar a origem).
- Se a senha da conta foi efetivamente trocada pelo provedor — há um conflito entre dois retornos do próprio provedor sobre esse ponto (ver Timeline).

## Malware / Achados do endpoint

Foi identificado, no endpoint do colaborador, um componente leve (classificado tecnicamente como infostealer/dropper) que:

- Se instalava em uma pasta não convencional dentro de `AppData\Local`, fora do padrão de instalação de software legítimo;
- Utilizava um binário legítimo do interpretador Python (hash verificado como pertencente a um componente saudável) para executar um script ofuscado em Base64;
- Mantinha um segundo processo, hospedado em uma pasta oculta disfarçada como diretório da Microsoft, responsável por recriar os artefatos do primeiro componente quando este era finalizado;
- Gerava arquivos locais nomeados com base no endereço de e-mail da conta comprometida, sugerindo coleta/uso de dados de contatos locais para os disparos.

A análise aprofundada do comportamento do malware (engenharia reversa, análise estática/dinâmica do script) será documentada separadamente na branch `malware-analysis`. Este README da main não avança além do que foi observado durante a resposta ao incidente.

## Contenção

- Remoção manual dos dois componentes maliciosos identificados (processo principal finalizado via linha de comando; processo secundário finalizado via Gerenciador de Tarefas).
- Isolamento dos arquivos maliciosos identificados em quarentena local, preservados para eventual análise futura.
- Varredura completa (ferramenta de segurança da Microsoft) e verificação manual de itens de inicialização (Autoruns) e processos ativos, repetidas após a remoção.
- Monitoramento da máquina por um período para confirmar que a pasta maliciosa não era mais recriada.
- Acionamento do provedor de e-mail para bloqueio de novos disparos, extração de logs e apoio na investigação.
- Reset de senha solicitado ao provedor
- Levantamento e comunicação da lista de destinatários externos atingidos, para envio de aviso oficial.

## Monitoramento

Após a remoção do componente malicioso, novas varreduras não identificaram vestígios adicionais, e não houve novos disparos suspeitos partindo da conta — confirmado tanto pela verificação técnica interna quanto pelo retorno do provedor de e-mail em contato telefônico posterior. Não foram identificados sinais de propagação para outras máquinas da empresa. O relatório oficial reforça, no entanto, que não é possível garantir com certeza absoluta a ausência de qualquer código dormente remanescente, a conclusão é sustentada pelas evidências disponíveis até o momento, não uma garantia categórica.

## IOCs

A lista completa de indicadores de comprometimento identificados durante a investigação está documentada em [`Incident Response/IOC.txt`](./Incident%20Response/IOC.txt), separando claramente indicadores de rede, e-mail, host, processo, persistência e hashes — de observações comportamentais que não são, por si só, IOCs.

## MITRE ATT&CK

Técnicas com evidência suficiente nos documentos (justificativas detalhadas em `IOC.txt`):

| ID | Técnica |
| --- | --- |
| T1566.002 | Phishing: Spearphishing Link |
| T1547.001 | Boot or Logon Autostart Execution: Registry Run Keys |
| T1036.005 | Masquerading: Match Legitimate Name or Location |
| T1059.006 | Command and Scripting Interpreter: Python |
| T1027 | Obfuscated Files or Information |
| T1114.001 | Email Collection: Local Email Collection |
| T1071.003 | Application Layer Protocol: Mail Protocols |

## Aprendizados

- **Cabeçalhos completos de e-mail são essenciais e nem sempre estão disponíveis por padrão.** A cópia original de um dos e-mails suspeitos não preservava cabeçalhos de transporte por ter sido extraída de uma resposta/encaminhamento — a confirmação técnica da submissão SMTP autenticada só foi possível graças a uma cópia do bounce, que por acaso preservou essa cadeia. Vale padronizar a coleta de `.eml` originais (não encaminhados) desde o primeiro sinal de incidente.
- **Autenticação corporativa sem MFA é um ponto de falha relevante.** O ambiente de e-mail utilizado não oferece suporte a MFA para o tipo de serviço contratado, o que eliminou uma camada de proteção que poderia ter dificultado (ou ao menos evidenciado mais rapidamente) o uso indevido da sessão/credencial autenticada.
- **Verificações OSINT complementares (reputação de IP, exposição de infraestrutura do provedor) agregam contexto, mas precisam ser tratadas como achados, não confirmações.** Achados como a porta Java RMI exposta no provedor são pontos válidos para cobrar do fornecedor, mas não devem ser tratados como causa confirmada do incidente sem validação adicional.
- **Persistência dupla em malwares leves reforça a importância de reverificação após a primeira remoção.** A simples finalização do processo principal não foi suficiente, foi necessário identificar o processo "vigia" responsável por recriar os artefatos.
- **Comunicação simultânea com o cliente durante a investigação técnica ajuda a conter o dano reputacional.** O aviso proativo aos destinatários externos, mesmo antes da confirmação definitiva da causa raiz, reduziu a chance de novas vítimas caírem no golpe.
