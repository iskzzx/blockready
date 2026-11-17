# Diagrama de Casos de Uso — BLOCKREADY

Atores e casos de uso do sistema completo (Etapas 1–5), levantados do código implementado
(controllers, regras de autorização por perfil e telas). Renderizável no GitHub/VS Code (Mermaid).

**Atores:** Visitante (não autenticado), Morador, Organização Parceira, Gestor Comunitário e
Administrador. O Administrador herda todos os casos de uso do Gestor; todos os perfis autenticados
compartilham os casos de uso comuns.

```mermaid
flowchart LR
    %% ===== Atores =====
    VIS(["🧑 Visitante"])
    MOR(["🧑 Morador"])
    ORG(["🏢 Organização Parceira"])
    GES(["🧑‍💼 Gestor Comunitário"])
    ADM(["🛡️ Administrador"])

    subgraph BLOCKREADY["Sistema BLOCKREADY"]
        direction LR

        subgraph ACESSO["Acesso"]
            UC01(["Criar conta<br>(morador ou organização)"])
            UC02(["Fazer login / logout"])
        end

        subgraph COMUM["Casos de uso comuns (autenticados)"]
            UC03(["Consultar recursos<br>comunitários"])
            UC04(["Visualizar mapa de recursos<br>(Leaflet + PostGIS)"])
            UC05(["Visualizar painel<br>(dashboard com CRI e alertas)"])
            UC06(["Consultar voluntários"])
        end

        subgraph MORADOR_UC["Morador"]
            UC07(["Registrar necessidade"])
            UC08(["Acompanhar solicitações"])
            UC09(["Cadastrar-se como voluntário"])
        end

        subgraph ORG_UC["Organização parceira"]
            UC10(["Gerenciar dados<br>institucionais"])
            UC11(["Cadastrar recurso<br>comunitário"])
            UC12(["Atualizar capacidade e<br>disponibilidade"])
            UC13(["Consultar histórico<br>do recurso"])
            UC14(["Consultar solicitações<br>relacionadas"])
        end

        subgraph GESTOR_UC["Gestão comunitária"]
            UC15(["Validar ou rejeitar<br>recursos"])
            UC16(["Aprovar / administrar<br>organizações"])
            UC17(["Gerenciar solicitações<br>(fluxo de status)"])
            UC18(["Importar dados externos<br>(IBGE, CEMADEN, Atlas)"])
            UC19(["Pré-visualizar arquivo<br>CSV"])
            UC20(["Consultar histórico<br>de importações"])
            UC21(["Consultar territórios e<br>dados geográficos"])
            UC22(["Consultar insights<br>(CRI + 4 insights)"])
            UC23(["Simular cenário de<br>emergência (72h)"])
            UC24(["Salvar e comparar<br>cenários"])
            UC25(["Calcular Community<br>Resilience Index"])
            UC26(["Priorizar recursos<br>(padrão Strategy)"])
        end

        subgraph ADMIN_UC["Administração"]
            UC27(["Gerenciar usuários<br>e permissões"])
            UC28(["Consultar registros<br>de auditoria"])
            UC29(["Registrar auditoria de<br>operações sensíveis"])
        end
    end

    %% ===== Visitante =====
    VIS --> UC01
    VIS --> UC02

    %% ===== Morador =====
    MOR --> UC02
    MOR --> UC03
    MOR --> UC04
    MOR --> UC05
    MOR --> UC06
    MOR --> UC07
    MOR --> UC08
    MOR --> UC09

    %% ===== Organização =====
    ORG --> UC02
    ORG --> UC03
    ORG --> UC04
    ORG --> UC05
    ORG --> UC06
    ORG --> UC10
    ORG --> UC11
    ORG --> UC12
    ORG --> UC13
    ORG --> UC14

    %% ===== Gestor =====
    GES --> UC02
    GES --> UC03
    GES --> UC04
    GES --> UC05
    GES --> UC06
    GES --> UC15
    GES --> UC16
    GES --> UC17
    GES --> UC18
    GES --> UC20
    GES --> UC21
    GES --> UC22
    GES --> UC23
    GES --> UC24

    %% ===== Administrador (herda Gestor) =====
    ADM -. "«herda»" .-> GES
    ADM --> UC27
    ADM --> UC28

    %% ===== Relações include/extend =====
    UC07 -. "«include»" .-> UC08
    UC11 -. "«extend»" .-> UC15
    UC18 -. "«include»" .-> UC19
    UC18 -. "«include»" .-> UC29
    UC15 -. "«include»" .-> UC29
    UC27 -. "«include»" .-> UC29
    UC24 -. "«include»" .-> UC29
    UC22 -. "«include»" .-> UC25
    UC23 -. "«include»" .-> UC25
    UC23 -. "«include»" .-> UC26
    UC05 -. "«include»" .-> UC25
```

## Descrição resumida dos casos de uso

| # | Caso de uso | Atores | Resumo |
|---|---|---|---|
| UC01 | Criar conta | Visitante | Cadastro público como Morador ou Organização (organização nasce pendente de aprovação) |
| UC02 | Login/logout | Todos | Sessão autenticada (cookie HttpOnly + CSRF, senha BCrypt) |
| UC03 | Consultar recursos | Autenticados | Lista com filtros (categoria, status, nome, raio geográfico); pendentes nunca aparecem como confirmados |
| UC04 | Mapa de recursos | Autenticados | Recursos, malha dos setores IBGE e estações CEMADEN no Leaflet |
| UC05 | Painel | Autenticados | Contagens do banco, CRI por categoria e alertas informativos |
| UC06 | Consultar voluntários | Autenticados | Voluntários ativos e suas disponibilidades |
| UC07 | Registrar necessidade | Morador | Necessidade por categoria gera solicitação com prioridade |
| UC08 | Acompanhar solicitações | Morador (próprias) / Org / Gestor | Fluxo ABERTA → EM_ANALISE → EM_ATENDIMENTO → ATENDIDA, com histórico |
| UC09 | Cadastrar-se como voluntário | Morador | Habilidades e disponibilidade |
| UC10 | Gerenciar dados institucionais | Organização | Atualização dos dados da própria organização |
| UC11 | Cadastrar recurso | Organização (aprovada) / Gestor | Recurso nasce pendente de validação |
| UC12 | Atualizar disponibilidade | Organização dona / Gestor | Capacidade total/disponível com registro histórico |
| UC13 | Histórico do recurso | Organização dona / Gestor | Alterações e registros de disponibilidade |
| UC14 | Solicitações relacionadas | Organização | Solicitações das categorias que a organização atende |
| UC15 | Validar/rejeitar recursos | Gestor | Só recursos validados contam nas análises; gera auditoria |
| UC16 | Administrar organizações | Gestor | Aprovar, rejeitar, suspender |
| UC17 | Gerenciar solicitações | Gestor | Transições de status com responsável e registro de atendimento |
| UC18 | Importar dados externos | Gestor | CSV do IBGE/CEMADEN/Atlas com validação e duplicidade |
| UC19 | Pré-visualizar arquivo | Gestor | Primeiras linhas + colunas antes de importar |
| UC20 | Histórico de importações | Gestor | Fonte, arquivo, período, totais, status, erros |
| UC21 | Territórios | Gestor | Setores censitários, estações, resumo da base geográfica |
| UC22 | Insights | Gestor | CRI + 4 insights estratégicos com explicabilidade |
| UC23 | Simular cenário 72h | Gestor | Tipo de emergência, horizonte, % afetados, parâmetros de demanda |
| UC24 | Salvar/comparar cenários | Gestor | Persistência da simulação e dos indicadores de cobertura |
| UC25 | Calcular CRI | Sistema | Capacidade ÷ demanda × 100 por categoria (unidades compatíveis) |
| UC26 | Priorizar recursos | Sistema | Estratégia por tipo de emergência (padrão Strategy) |
| UC27 | Gerenciar usuários | Administrador | Criar contas, alterar perfil, ativar/desativar |
| UC28 | Consultar auditoria | Administrador | Registros de operações sensíveis |
| UC29 | Registrar auditoria | Sistema | Include das operações sensíveis (validação, importação, permissões, cenários) |
