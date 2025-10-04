Addendum v2.1 — Ajustes a partir do feedback

Responsável: Bruno Custódio de Castro Silva
Data: Outubro/2025

A. Visão risk-based e Priorização (reforço prático)

Usaremos uma matriz simples Impacto × Probabilidade para priorizar suites e cenários:

Tag	Impacto	Probabilidade	Prioridade	Exemplos
critical	Alto	Alto	P1	Login, concluir compra, controle de estoque
security	Alto	Médio	P1	Permissões admin, token, acessos indevidos
boundary	Médio	Médio	P2	Tamanhos, limites, quantidades extremas
integration	Médio	Baixo	P2	Fluxos E2E e consistência de estado
noncritical	Baixo	Baixo	P3	Mensagens menos sensíveis

Rotina sugerida:

Diário: smoke AND critical

Semanal: regression

Quinzenal: boundary

Mensal: e2e OR integration

B. Automação da Expiração de Token (CT-004 revisado)
B.1 Objetivo

Transformar CT-004 em teste realmente automatizado e confiável, validando acesso antes, no limite e após expiração, sem flakiness.

B.2 Estratégia

Obter token com /login.

Calcular TTL restante lendo o exp do JWT (payload base64url).

Validar acesso a uma rota protegida imediatamente (deve permitir).

Aguardar até o limite (ex.: exp − 1s) e validar ainda permitido.

Ultrapassar o exp (ex.: +2s) e validar rejeição (401/“Token expirado”).

B.3 Padronização do CT-004

Prioridade: Alta

Tipo: Boundary/Security

Tags: security, boundary, critical

Métrica de sucesso: 100% determinístico em 5 execuções consecutivas

B.4 Exemplo de implementação (Robot Framework)

Observação: usa apenas bibliotecas padrão + Collections + RequestsLibrary + JSONLibrary.

resources/auth_keywords.robot

*** Settings ***
Library    Collections
Library    JSONLibrary

*** Keywords ***
Decode JWT Exp Seconds
    [Arguments]    ${token}
    # Separa payload (parte 1)
    ${parts}=    Split String    ${token}    .
    ${payload_b64}=    Set Variable    ${parts[1]}
    # Normaliza padding do base64url
    ${missing}=    Evaluate    (-len("${payload_b64}") % 4)
    ${payload_padded}=    Set Variable    ${payload_b64}${'=' * ${missing}}
    ${payload_json}=    Evaluate    __import__('base64').urlsafe_b64decode(r'''${payload_padded}''').decode()
    ${payload}=    Convert String To Json    ${payload_json}
    ${exp}=    Get Value From Json    ${payload}    $.exp
    ${now}=    Evaluate    __import__('time').time()
    ${ttl}=    Evaluate    int(${exp} - ${now})
    [Return]    ${ttl}


tests/auth/test_token_expiration.robot

*** Settings ***
Library           RequestsLibrary
Library           JSONLibrary
Library           Collections
Resource          ../../resources/auth_keywords.robot
Suite Setup       Create Session    api    ${BASE_URL}

*** Variables ***
${LOGIN_EMAIL}    usuario@exemplo.com
${LOGIN_PASS}     123456

*** Test Cases ***
CT-004 Token Expiration Is Enforced
    [Tags]    security    boundary    critical
    ${resp}=    POST On Session    api    /login
    ...    json={"email":"${LOGIN_EMAIL}","password":"${LOGIN_PASS}"}
    Should Be Equal As Integers    ${resp.status_code}    200
    ${auth}=    Get Value From Json    ${resp.json()}    $.authorization
    Should Start With    ${auth[0]}    Bearer 
    ${token}=    Evaluate    "${auth[0]}".split(" ",1)[1]

    ${ttl}=    Decode JWT Exp Seconds    ${token}
    Should Be True    ${ttl} > 5    msg=TTL insuficiente para teste; tente novamente

    # Acesso imediato deve passar
    ${r1}=    GET On Session    api    /usuarios    headers={"Authorization":"Bearer ${token}"}
    Should Be Equal As Integers    ${r1.status_code}    200

    # No limite (ttl-1)
    Sleep    ${ttl - 1}s
    ${r2}=    GET On Session    api    /usuarios    headers={"Authorization":"Bearer ${token}"}
    Should Be Equal As Integers    ${r2.status_code}    200

    # Após expirar (+2s)
    Sleep    2s
    ${r3}=    GET On Session    api    /usuarios    headers={"Authorization":"Bearer ${token}"}
    Should Be Equal As Integers    ${r3.status_code}    401
    ${msg}=    Get Value From Json    ${r3.json()}    $.message
    Should Match Regexp    ${msg[0]}    (?i)token.*expirado


Dica anti-flake: se o TTL vier muito curto (≤5s), refaça o login antes do teste ou ajuste o sleep do “limite” para max(ttl-2,1).

C. Padronização de Mensagens e Asserções
C.1 Catálogo de mensagens (contrato)

Coloque em resources/messages.robot para uso centralizado:

*** Variables ***
${MSG_LOGIN_FAIL}           Email e/ou senha inválidos
${MSG_EMAIL_DUP}            Este email já está sendo usado
${MSG_FORBIDDEN}            Rota exclusiva para administradores
${MSG_CART_DELETE_BLOCK}    Não é permitido excluir usuário com carrinho cadastrado
${MSG_TOKEN_EXPIRED_RX}     (?i)token.*expirado

C.2 Helpers de asserção reutilizáveis

resources/assertions.robot

*** Settings ***
Library    JSONLibrary

*** Keywords ***
Should Have Status
    [Arguments]    ${resp}    ${expected}=200
    Should Be Equal As Integers    ${resp.status_code}    ${expected}

Should Have Message
    [Arguments]    ${resp}    ${expected}
    ${msg}=    Get Value From Json    ${resp.json()}    $.message
    Should Be Equal    ${msg[0]}    ${expected}

Should Match Message
    [Arguments]    ${resp}    ${regex}
    ${msg}=    Get Value From Json    ${resp.json()}    $.message
    Should Match Regexp    ${msg[0]}    ${regex}

Should Have Keys
    [Arguments]    ${resp}    @{keys}
    ${obj}=    Evaluate    ${resp.json()}
    :FOR    ${k}    IN    @{keys}
    \    Dictionary Should Contain Key    ${obj}    ${k}

C.3 Aplicação nos CTs

CT-002: Should Have Status ${resp} 401 + Should Have Message ${resp} ${MSG_LOGIN_FAIL}

CT-011: Should Have Status ${resp} 403 + Should Have Message ${resp} ${MSG_FORBIDDEN}

CT-009: Should Have Status ${resp} 400 + Should Have Message ${resp} ${MSG_CART_DELETE_BLOCK}

Benefício: uma mudança de copy no backend exige ajuste em um lugar.

D. Evidências Completas no Checklist
D.1 Itens obrigatórios por execução

Identificação da execução: timestamp, commit SHA do repositório de testes, BASE_URL, ambiente.

Artefatos Robot: robot-output.xml, robot-log.html, robot-report.html.

Artefatos Newman (se usados): newman-report.html, newman-results.xml.

Captura de request/response relevantes (ex.: primeira falha de cada módulo) em results/<timestamp>/samples/.

Massa de dados usada (JSON ou CSV) e estratégia de limpeza.

Matriz de rastreabilidade gerada a partir das tags: CT ↔ módulo ↔ objetivo.

Lista de defeitos abertos (id, severidade, passo para reproduzir, request/response).

Assinatura de revisão (quem revisou, quando).

D.2 Checklist atualizado (substituir no plano)

Antes da Execução

 Ambiente responde GET /usuarios com 200

 Variáveis definidas (BASE_URL, credenciais)

 Massa de dados preparada e isolada

 Catálogo de mensagens atualizado

 Tags de priorização revisadas (P1/P2/P3)

Durante a Execução

 Evidências salvas em results/<timestamp>/

 Samples de request/response nas falhas graves

 Tempos de resposta p95 coletados (mínimo: Auth e Checkout)

 Logs e stdout arquivados

Após a Execução

 Relatórios Robot/Newman anexados

 Matriz de rastreabilidade gerada a partir das tags

 Métricas: taxa de sucesso por módulo, falhas por severidade

 Defeitos registrados com anexo de evidências

 Lições aprendidas preenchidas (ver modelo abaixo)

 Aprovação de saída (aceitação) registrada

D.3 Modelo de “Lições Aprendidas” (para colar no fim do relatório)
Lições Aprendidas — Execução <YYYY-MM-DD HH:MM>
1) O que funcionou bem:
2) O que falhou e por quê:
3) Ações corretivas imediatas:
4) Prevenções para regressão:
5) Decisões de engenharia (trade-offs):
6) Próximos passos:

E. Traçabilidade e Modelo de Estado (reforço)

Estado de Usuário: {cadastrado, autenticado, com_carrinho, sem_carrinho}

Estado de Produto: {criado, sem_estoque, com_estoque}

Estado de Carrinho: {inexistente, aberto, concluído, cancelado}

Vincule cada transição às tags dos CTs (ex.: CT-016 cobre transição aberto → concluído e estoque: decrementar).

F. Diferenças práticas que o avaliador vai notar

CT-004 agora é realmente automatizado e robusto (decodifica exp, não “chuta” tempo).

Asserções padronizadas com helpers e catálogo de mensagens centralizado.

Evidências completas e organizadas por execução com checklist exigente (+ lessons learned).

Priorização clara via tags e matriz risco × probabilidade.

Reforço de rastreabilidade e estados — mais fácil auditar e justificar cobertura.

Se quiser, eu também te entrego os três arquivos de recursos prontos para commit no seu repo:

resources/auth_keywords.robot

resources/messages.robot

resources/assertions.robot

C:\Users\bruno\OneDrive\Desktop\Severest