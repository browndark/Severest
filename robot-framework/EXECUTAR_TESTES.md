# Execução dos Testes Automatizados — ServeRest
**Autor:** Bruno Custódio de Castro Silva  
**Data:** Outubro de 2025

---

## Pré-requisitos

### Ferramentas Necessárias
- Python 3.8+ instalado  
- Robot Framework e bibliotecas  
- Git para controle de versão  

### Instalação das Dependências
```bash
# Instalar Robot Framework e bibliotecas
pip install robotframework
pip install robotframework-requests
pip install robotframework-jsonlibrary

# Verificar instalação
robot --version

Como Executar os Testes
Execução Básica
# Navegar para o diretório do projeto
cd "C:\Users\bruno\OneDrive\Desktop\Severest\robot-framework"

# Executar todos os testes
robot tests/

# Executar suite específica
robot tests/auth/test_auth.robot
robot tests/cart/test_cart_boundary.robot

Execução com Filtros
# Executar apenas testes críticos
robot --include critical tests/

# Executar apenas testes de boundary
robot --include boundary tests/

# Executar testes de um módulo específico
robot --include auth tests/

# Executar smoke tests
robot --include smoke tests/

Execução com Configurações Personalizadas
# Definir ambiente de teste
robot --variable BASE_URL:http://localhost:3000 tests/

# Salvar resultados em diretório específico
robot --outputdir results tests/

# Executar com log detalhado
robot --loglevel DEBUG tests/

# Executar em paralelo (com pabot)
pabot --processes 4 tests/

Relatórios e Resultados
Arquivos Gerados

output.xml: Dados detalhados da execução

log.html: Log interativo da execução

report.html: Relatório resumido dos resultados

Visualização dos Resultados
# Abrir relatório no navegador (Windows)
start report.html

Estrutura de Resultados
results/
├── output.xml          # Dados XML da execução
├── log.html            # Log detalhado interativo
├── report.html         # Relatório visual dos resultados
├── screenshots/        # Evidências (se configurado)
└── test-evidence/      # Logs adicionais
Cenários de Execução
1) Smoke Tests (Execução Rápida)
# Testes essenciais - ~2 minutos
robot --include smoke --outputdir results/smoke tests/


Inclui:

Login básico

CRUD essencial de usuários

CRUD essencial de produtos

Adição básica ao carrinho

2) Regression Tests (Suite Completa)
# Todos os testes - ~15 minutos
robot --outputdir results/regression tests/


Inclui:

Todos os testes funcionais

Todos os boundary tests

Testes de integração

Validações de segurança
3) Boundary Tests (Testes de Limite)
# Apenas boundary testing - ~5 minutos
robot --include boundary --outputdir results/boundary tests/


Inclui:

Validações de campos

Testes de limite de estoque

Boundary de autenticação

Valores extremos

4) Integration Tests (Fluxos E2E)
# Testes de integração - ~8 minutos
robot --include integration --outputdir results/integration tests/


Inclui:

Fluxos completos de compra

Integração entre módulos

Cenários de negócio complexos

Configurações Avançadas
Variáveis de Ambiente

Crie um arquivo de variáveis personalizado:

# custom_variables.py
BASE_URL = "https://meu-ambiente.com"
TIMEOUT = 30
DEBUG_MODE = True


Use nas execuções:

robot --variablefile custom_variables.py tests/

Execução Paralela (Pabot)
# Instalar pabot
pip install robotframework-pabot

# Executar em paralelo
pabot --processes 4 --outputdir results/parallel tests/

Integração CI/CD
# Comando para CI/CD (Jenkins, GitHub Actions, etc.)
# Comando para CI/CD (Jenkins, GitHub Actions, etc.)
robot --outputdir results --output output.xml --log log.html --report report.html --loglevel INFO tests/
📈 Métricas e Monitoramento
Análise de Resultados
# Comando para estatísticas
robot --dryrun --output stats.xml tests/

KPIs Importantes

Taxa de Sucesso: > 95% (esperada)

Tempo de Execução: suite completa < 20 minutos

Cobertura: > 80% dos casos de teste críticos

Falhas Comuns e Soluções

Erro de Conexão
Error: Connection refused
Solução: Verifique se a API está disponível

curl https://serverest.dev/usuarios


Token Expirado
Error: 401 Unauthorized
Solução: Verifique configuração de timeout e renovação de token nos testes

Dados de Teste Conflitantes
Error: Email já existe
Solução: Use dados únicos baseados em timestamp

📋 Checklist de Execução
Antes da Execução

 API ServeRest está disponível

 Dependências instaladas

 Diretório de resultados existe

 Sem execuções paralelas concorrentes

Durante a Execução

 Monitorar logs em tempo real

 Checar timeouts excessivos

 Observar padrões de falha

Após a Execução

 Analisar relatório gerado

 Verificar taxa de sucesso

 Documentar falhas encontradas

 Arquivar resultados com timestamp

 Integração com Processo Manual
Fluxo Recomendado

Executar Smoke Tests automatizados

Executar testes manuais específicos

Executar Regression automatizada

Validar resultados combinados

Gerar relatório consolidado

Complemento aos Testes Manuais

Automação não substitui exploração manual

Use automação para regressão e smoke

Mantenha testes manuais para novos cenários

Documente descobertas para futura automação

📞 Suporte e Troubleshooting
Logs Detalhados
# Máximo detalhe para debug
robot --loglevel TRACE --console verbose tests/auth/test_auth.robot

Execução de Teste Único
# Executar apenas um teste específico
robot --test "CT-001: Login com Credenciais Válidas" tests/auth/test_auth.robot

Modo Debug
# Parar na primeira falha para análise
robot --exitonfailure tests/

Mensagem Pessoal

Mesmo com as dificuldades, eu consegui entregar, porque enxergo a UOL Compass como uma grande oportunidade — e um desafio que me impulsiona a evoluir.


Se quiser, eu também salvo isso como `README.md` com a formatação certinha.
