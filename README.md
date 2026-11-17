# BLOCKREADY

**Stronger Neighborhoods. Safer Futures.**

Plataforma CitizenTech de resiliência comunitária: conecta moradores, organizações, voluntários e gestores para
mapear recursos essenciais (água, abrigos, energia, alimentação, comunicação e saúde), identificar insuficiências
e apoiar o planejamento durante emergências, com horizonte de até 72 horas.

> O BLOCKREADY é uma ferramenta de apoio ao planejamento. Não garante a sobrevivência nem a segurança de uma comunidade.

**Estado: Etapa 5 de 5 (aplicação completa).** Autenticação por perfil, recursos comunitários, necessidades e
solicitações, organizações, voluntários, importação de dados públicos (IBGE, CEMADEN, Atlas de Desastres),
Community Resilience Index, quatro insights estratégicos, simulação de cenários (Strategy), mapa Leaflet,
dashboard integrado, Swagger e testes automatizados.

## Tecnologias

Java 25 · Spring Boot 3.5 (MVC, Data JPA, Security, Validation, Actuator) · Maven · PostgreSQL 16 + PostGIS 3.4 ·
Flyway · Hibernate Spatial · JUnit 5 · HTML5/CSS3/JavaScript puro · Leaflet 1.9 · springdoc-openapi (Swagger) ·
Docker Compose.

## Execução com Docker (recomendado)

```bash
cp .env.example .env        # já vem com valores de demonstração prontos para avaliação local
docker compose up --build
```

Acesse http://localhost:8080 (ou a porta de `APP_PORT`). Para encerrar: `docker compose down`
(os dados do banco ficam no volume `blockready-db`; para apagar tudo: `docker compose down -v`).

### Execução local (Maven)

```bash
docker compose up -d db     # sobe só o PostgreSQL/PostGIS
# Linux/macOS
set -a; source .env; set +a
mvn spring-boot:run
```

No Windows (PowerShell), defina as variáveis com `$env:DB_PASSWORD="..."` antes de `mvn spring-boot:run`.
Requer JDK 25 e Maven 3.9+.

## Verificação

```bash
curl -i http://localhost:8080/api/v1/health
```

Resposta esperada (HTTP 200): `status: "UP"`, banco `UP` e a versão do PostGIS. `/actuator/health` também responde.

Testes:

```bash
mvn test      # unitários (CSV, importadores, CRI, estratégias — sem banco nem Docker)
mvn verify    # inclui FluxoComunitarioIT com Testcontainers (exige Docker)
```

## Credenciais de demonstração (login para avaliação)

Usando o `.env.example` copiado para `.env`, o primeiro administrador é criado automaticamente na primeira
execução. Logins **exclusivamente demonstrativos** (válidos apenas na sua instância local — troque-os em
qualquer uso fora de demonstração):

| Perfil | E-mail | Senha |
|---|---|---|
| **Administrador** | `admin@blockready.local` | `AdminBlockready2026` |
| Organização parceira (crie pela tela de cadastro, ex.:) | `org@blockready.local` | `OrgBlockready2026` |

> A conta de organização não vem pré-criada num banco novo: cadastre-a em `/cadastro.html` (perfil
> "Organização parceira") e aprove-a com o administrador. Moradores criam a própria conta livremente.

## Primeiro acesso e perfis

1. Entre em `/login.html` com o administrador acima (ou defina outro `BOOTSTRAP_ADMIN_EMAIL`/`BOOTSTRAP_ADMIN_PASSWORD` no `.env`).
2. Perfis e permissões:

| Perfil | Pode |
|---|---|
| **Morador** | criar conta, consultar recursos e o mapa, registrar necessidades, acompanhar solicitações |
| **Organização parceira** | gerenciar seus dados, cadastrar recursos, atualizar disponibilidade, ver suas solicitações |
| **Gestor comunitário** | validar recursos, administrar organizações, importar dados, insights, cenários, territórios |
| **Administrador** | tudo do gestor + gerenciar usuários/permissões e consultar auditoria |

Contas de gestor são criadas pelo admin (`POST /api/v1/usuarios` ou tela de administração).

## Variáveis de ambiente

| Variável | Padrão | Descrição |
|---|---|---|
| `DB_PASSWORD` | — (obrigatória) | Senha do PostgreSQL |
| `BOOTSTRAP_ADMIN_EMAIL` / `BOOTSTRAP_ADMIN_PASSWORD` | vazio | Primeiro administrador (senha de 8 a 72 caracteres) |
| `SESSION_COOKIE_SECURE` | `false` | `true` quando servir por HTTPS |
| `DB_NAME` / `DB_USER` | `blockready` | Banco e usuário |
| `DB_HOST` / `DB_PORT` | `localhost` / `5432` | Endereço do banco (no Compose, `DB_HOST` é `db`) |
| `APP_PORT` | `8080` | Porta HTTP |
| `LOG_LEVEL_ROOT` / `LOG_LEVEL_APP` | `INFO` | Níveis de log |
| `LOG_DIR` | `logs` | Pasta dos logs (rotação diária/10 MB, 14 dias) |

Nenhuma credencial é versionada: `.env` está no `.gitignore`.

## Integração de dados públicos (Etapa 3)

As três fontes oficiais são integradas por **importação de arquivos CSV públicos** (tela **Dados externos**,
perfis gestor/admin), mecanismo previsto quando os dados são distribuídos como arquivos — nenhum endpoint foi
inventado. Cada importação tem pré-visualização, validação, controle de duplicidade (hash do arquivo e chaves
naturais dos registros), histórico com totais e erros, e marcação explícita de **dados demonstrativos**.

| Fonte | Abrangência | Colunas aceitas (apelidos também) | Uso |
|---|---|---|---|
| **IBGE — Censo 2022** | setor censitário | `CD_SETOR; NM_MUN; SIGLA_UF; v0001 (população); v0002 (domicílios); AREA_KM2?; WKT?` | demanda estimada e cobertura territorial |
| **CEMADEN — pluviometria** | ponto da estação | `codEstacao; nomeEstacao; municipio; uf; latitude; longitude; datahora (UTC); valorMedida` | monitoramento de prontidão |
| **Atlas de Desastres — MIDR** | município | `municipio; uf; descricao_tipologia; cobrade?; data_registro; DH_MORTOS; DH_TOTAL_AFETADOS; DH_DESABRIGADOS; DH_DESALOJADOS; prejuizo_total?` | histórico de cenários |

Tratamento aplicado: horários do CEMADEN convertidos/armazenados em **UTC**; valores ausentes ou inválidos
rejeitados linha a linha com o motivo; estação/período de referência identificados; geometrias em **SRID 4326**
(o mesmo dos recursos), com índices GiST; registros do Atlas **nunca** atribuídos a bairros (abrangência municipal);
medições de estação associadas a recursos apenas por proximidade espacial explícita.

Arquivos de demonstração (marque **"Dados demonstrativos"** ao importar): `docs/dados-exemplo/`
(`ibge-setores-demo.csv`, `cemaden-chuva-demo.csv`, `atlas-desastres-demo.csv`). As medições de demonstração
cobrem datas próximas a nov/2026; dados reais devem ser baixados nos portais oficiais das fontes.

## Inteligência analítica (Etapa 4)

**Community Resilience Index**: `CRI = capacidade disponível ÷ demanda estimada × 100`, por categoria.
Capacidade = soma dos recursos **validados** com unidade compatível com a escala da categoria (litros p/ água,
vagas p/ abrigo, refeições p/ alimentação, pessoas p/ energia/comunicação, atendimentos p/ saúde — recursos com
unidade incompatível são excluídos e informados). Demanda = população dos setores do IBGE × parâmetros de
planejamento **configuráveis** (padrões: 3 L/pessoa/dia, 2 refeições/pessoa/dia, abrigo p/ 30% dos afetados,
energia/comunicação p/ 20%, saúde p/ 5% — premissas, não dados oficiais). Faixas visuais configuráveis
(≥90% suficiente, ≥50% parcial, abaixo insuficiente, sem base = SEM_DADOS). Divisão por zero e ausência de dados
são tratadas como SEM_DADOS; **100% de cobertura não é garantia de segurança**.

**Quatro insights** (tela **Insights**), todos com pergunta, dados, fontes, metodologia, resultado, interpretação,
decisão recomendada, impacto e limitações — e aviso de "dados insuficientes" quando a base não permite a análise:
1. **Historical Disaster Exposure** — tipos de desastre com maior impacto histórico (Atlas, municipal).
2. **Community Resource Coverage** — população × distribuição espacial dos recursos (IBGE + PostGIS `ST_Contains`).
3. **Emergency Readiness Monitoring** — chuva observada (CEMADEN, 72 h) × disponibilidade × solicitações
   (descritivo; sem inferência causal).
4. **Smart Resource Allocation** — déficit por categoria + priorização explicável via **padrão Strategy**.

**Cenários (72h Planning)** (tela **Cenários**): enchente, tempestade intensa, interrupção de energia e
interrupção de serviços essenciais; horizonte configurável até 72 h; % da população afetada e parâmetros de
demanda ajustáveis; cada tipo usa sua estratégia de priorização; cenários podem ser salvos e comparados.
É ferramenta de planejamento, não previsão.

## Telas

Login/Register · Painel (dashboard com CRI e alertas) · Recursos (+cadastro/validação/histórico) ·
Mapa (Leaflet + PostGIS: recursos, malha IBGE, estações CEMADEN) · Necessidades · Solicitações · Organizações ·
Voluntários · Territórios · Dados externos (fontes/importação/histórico) · Insights · Cenários.

## Endpoints REST

Documentação interativa: **http://localhost:8080/swagger-ui.html** (OpenAPI em `/v3/api-docs`).

Resumo (`/api/v1`): `auth/*` (register, login, logout, me) · `usuarios` · `organizacoes` · `recursos`
(+`/disponibilidade`, `/validar`, `/rejeitar`, `/historico`) · `necessidades` · `solicitacoes` · `voluntarios` ·
`dashboard` · `auditoria` · `health` · **Etapa 3:** `fontes`, `importacoes` (+`/previa`), `territorios/*`
(resumo, setores, geojson, estacoes) · **Etapa 4:** `analytics/cri`, `analytics/insights`, `cenarios`
(+`/simular`).

## Segurança e privacidade (LGPD)

Sessão com cookie HttpOnly + CSRF por cookie; senhas com BCrypt; autorização por perfil (`@PreAuthorize`) e
regras de dono nos serviços (organização só altera os próprios recursos); endereço/contato de solicitações
visíveis apenas a quem atende; operações sensíveis auditadas (`auditoria_registro`); dados pessoais fora do
dashboard e dos agregados; nenhum segredo no repositório.

## Estrutura

```
src/main/java/br/com/blockready/
  config  controller  service  repository  model/entity  dto  mapper
  exception  security  integration (ibge, cemaden, disasteratlas)  analytics (strategy)
src/main/resources/
  application.yml  logback-spring.xml  db/migration (V1..V8)
  static/ (assets, css, js, pages, components)
docs/  modelo-de-dados.md  diagrama-classes.md  dados-exemplo/
```

Arquitetura em camadas (MVC); repositories Spring Data JPA no papel de DAO; Strategy em `analytics/strategy`;
importadores desacoplados dos controllers em `integration`. Diagrama de classes UML em
[docs/diagrama-classes.md](docs/diagrama-classes.md); casos de uso em
[docs/diagrama-casos-de-uso.md](docs/diagrama-casos-de-uso.md); modelo de dados em
[docs/modelo-de-dados.md](docs/modelo-de-dados.md).

## Limitações conhecidas

- As análises dependem dos dados importados/cadastrados; os parâmetros de demanda são premissas configuráveis.
- Registros do Atlas têm abrangência municipal; medições do CEMADEN valem para o ponto da estação.
- O mapa usa tiles do OpenStreetMap e Leaflet via CDN (requer internet para exibir o mapa).
- Setores sem malha (WKT) não participam dos cruzamentos espaciais.
