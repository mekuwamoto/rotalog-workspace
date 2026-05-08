# Débitos Técnicos — rotalog-workspace

## Alta Severidade

| # | Arquivo | Linha(s) | Problema |
|---|---------|----------|----------|
| 1 | `docker-compose.yml` | 8–10 | **Credenciais hardcoded** — `POSTGRES_USER: rotalog_admin` e `POSTGRES_PASSWORD: rotalog123` expostos em texto plano no arquivo versionado |
| 2 | `docker-compose.yml` | 4–27 | **Ponto único de falha** — um único container PostgreSQL sem réplica, failover ou backup automatizado; queda do container = perda total de disponibilidade |
| 3 | `init-schemas.sql` | 13 | **Violação de bounded context** — `api-notificacoes` acessa diretamente os schemas `frotas` e `entregas` via SQL raw; comentário `FIXME` confirma a violação |
| 4 | `docker-compose.yml` | 25 | **Ausência de health checks** — nenhum `healthcheck:` configurado no serviço `postgres`; serviços dependentes sobem antes do banco estar pronto, causando falhas de conexão silenciosas na inicialização |
| 5 | `docker-compose.yml` | 26 | **Ausência de resource limits** — sem `deploy.resources.limits` para CPU/memória; o container pode consumir todos os recursos do host |
| 6 | `03-migration-entregas.sql` | 31, 46 | **FK ausente entre schemas** — `entregas.motorista_id` e `entregas.veiculo_placa` não possuem FK para as tabelas em `frotas`; orphaned records possíveis sem constraint de integridade referencial |
| 7 | `03-migration-entregas.sql` | 46 | **FK ausente em `rastreamentos.entrega_id`** — permite registros de rastreamento órfãos |

## Média Severidade

| # | Arquivo | Linha(s) | Problema |
|---|---------|----------|----------|
| 8 | `08-add-motorista-veiculo-info.sql` | 20–38 | **Dados denormalizados propagados manualmente** — `motorista_nome` e `veiculo_modelo` foram adicionados à tabela `entregas` e populados por UPDATE hardcoded; qualquer atualização em `frotas` não reflete em `entregas` |
| 9 | `02-migration-frotas.sql` | 37 | **FK ausente em `manutencoes.veiculo_id`** — manutenções podem apontar para veículos inexistentes |
| 10 | `02-migration-frotas.sql` e `03-migration-entregas.sql` | 18, 32–33 | **Ausência de índices essenciais** — tabelas `motoristas`, `veiculos`, `manutencoes` sem índices em colunas de filtro comuns (`status`, `data_cadastro`); `rastreamentos` sem índice em `data_evento` |
| 11 | `03-migration-entregas.sql` e `04-migration-notificacoes.sql` | 33, 26 | **Sem particionamento** em tabelas de crescimento contínuo — `rastreamentos` e `notificacoes` são append-only sem estratégia de particionamento por data; degradação de performance inevitável em produção |
| 12 | `init-schemas.sql` | 17–19 | **Um único usuário com GRANT ALL** em todos os schemas — `rotalog_admin` tem `ALL PRIVILEGES` em tudo; princípio do menor privilégio violado sem roles separados por serviço |
| 13 | `init-schemas.sql` | 22 | **Ausência de tabelas de auditoria** — comentários `TODO: Add audit tables` em vários arquivos, nunca implementados; não há rastreamento de quem alterou o quê |
| 14 | `docker-compose.yml` | 27 | **Ausência de estratégia de backup** — nenhum volume de backup, `pg_dump` agendado ou política de retenção configurada |

## Baixa Severidade

| # | Arquivo | Linha(s) | Problema |
|---|---------|----------|----------|
| 15 | `docker-compose.yml` | 29–33 | **Serviços de infraestrutura planejados e nunca implementados** — comentários `TODO` para Redis, Kafka, Elasticsearch, Prometheus e Grafana; o sistema foi construído como se eles existissem |
| 16 | `08-add-motorista-veiculo-info.sql` | 34 | **Inconsistência de placa** — o `UPDATE` usa `veiculo_placa = 'JKL3M45'`, mas o seed de frotas cadastrou `'JKL0M12'` para o Iveco Daily; dado nunca é populado corretamente |
| 17 | `08-add-motorista-veiculo-info.sql` | 39–42 | **Script de migration executa `SELECT` ao final** — o `SELECT` de verificação inline polui os logs de inicialização do Docker |
| 18 | `docker-compose.yml` | 1 | **`version: '3.8'` obsoleto** — a chave `version` foi descontinuada na especificação Compose v2; Docker moderno emite warning |
| 19 | `init-schemas.sql` | 23 | **Ausência de Row-Level Security** — comentário `TODO: Add row-level security` nunca implementado; todos os serviços leem dados uns dos outros sem restrição de linha |

---

**Resumo:** Alta: 7 | Média: 7 | Baixa: 5
