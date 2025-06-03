# Documentação do fluxo BD
## 1. Áreas funcionais e tabelas principais

| Domínio                             | Propósito                                                              | Tabelas principais (na ordem de uso típico)                                |
| ----------------------------------- | ---------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| **Usuários & Recursos**             | Pessoas que atuam no sistema e os ativos físicos/digitais que consomem | `usuarios`, `recursos`                                                     |
| **Gestão de Contratos & Chamados**  | Acordos comerciais formais e a solicitação inicial do cliente          | `contratos` → `chamados` → `pre_contratos`                                 |
| **Projetos & Ciclos**               | Execução técnica e períodos de referência                              | `projetos`, `ciclos`                                                       |
| **Ciclo de Vida dos Documentos**    | Entregas de engenharia/projeto, revisões e emissões                    | `documentos` → `revisoes_doc` → `rejeicoes_revisao` → `emissoes_documento` |
| **Progresso & Faturamento**         | Medição da execução, entregas, aprovações e faturas                    | `medicoes_documento`, `entregas_ciclo`, `aprovacoes_cli`, `cobrancas`      |
| **Preços & Orçamento**              | Catálogo de preços unitários e orçamento geral do chamado              | `precos_unitarios`, `orcamentos`                                           |
| **Fornecedores & Pagamentos**       | Fornecedores externos e o que é pago a eles                            | `fornecedores`, `pagamentos_forn`                                          |
| **Uso de Recursos**                 | Consumo de recursos internos por documento                             | `uso_recursos`                                                             |
| **Responsabilidades & Dicionários** | Responsáveis por cada documento / códigos de motivo                    | `responsaveis_documento`, `tipos_rejeicao`                                 |
## 2. Narrativa do fluxo de dados (visão geral)
``` mermaid
flowchart LR
  subgraph Comercial
    CH[chamados]
    CH --> PC[pre_contratos]
    PC --> C[contratos]
  end
  subgraph Execução
    C --> P[projetos]
    P --> D[documentos]
    D --> R[revisoes_doc]
    R --> RR[rejeicoes_revisao]
    D --> E[emissoes_documento]
    D --> M[medicoes_documento]
    D --> EC[entregas_ciclo]
    D --> A[aprovacoes_cli]
  end
  subgraph Financeiro
    PC --> O[orcamentos]
    C --> PU[precos_unitarios]
    EC --> CB[cobrancas]
    A --> CB
    FM[fornecedores] --> PG[pagamentos_forn]
  end
  RU[recursos] --> U[uso_recursos]
  D --> U

```

## 2.1 Início do processo comercial

- O cliente realiza um contato inicial por meio de um `chamado`, descrevendo a necessidade técnica ou comercial.
    
- Esse chamado serve como ponto de partida para a estruturação de uma proposta.
    

## 2.2 Pré-contrato e proposta orçamentária

- Com base no chamado, a equipe comercial elabora um `pre_contratos`, que pode conter escopos parciais, prazos e valores estimados.
    
- Um `orcamento` é então gerado, referenciando itens do catálogo de preços (`precos_unitarios`) quando aplicável.
    

## 2.3 Formalização do contrato

- Após aprovação da proposta pelo cliente, um `contrato` é formalmente registrado.
    
- Este contrato consolida os dados acordados: escopo, datas de início e término, valores e vínculo ao chamado de origem.
    

## 2.4 Criação de projeto e planejamento de execução

- O contrato é desmembrado em `projetos`, geralmente por disciplina técnica ou pacote de trabalho.
    
- Períodos de acompanhamento (mensal, quinzenal, por marco, etc.) são registrados em `ciclos`.
    

## 2.5 Gestão do ciclo de vida documental

- As entregas são registradas na tabela `documentos`, identificadas por projeto e código.
    
- Os papéis de cada pessoa envolvida (autor, verificador, aprovador) são definidos em `responsaveis_documento`.
    
- Cada nova versão é registrada em `revisoes_doc`, e eventuais recusas são detalhadas em `rejeicoes_revisao`, com base nos códigos definidos em `tipos_rejeicao`.
    
- Uma versão aprovada é formalmente emitida em `emissoes_documento`.
    
- O uso de recursos internos vinculados ao documento (licenças, impressoras, etc.) é registrado em `uso_recursos`.
    

## 2.6 Medição de progresso e faturamento

- A execução técnica é quantificada em `medicoes_documento`, multiplicando-se as quantidades por seus respectivos preços unitários.
    
- Ao final de cada ciclo, as entregas são consolidadas em `entregas_ciclo`, servindo como base para a emissão da cobrança.
    
- A aceitação formal do cliente é registrada em `aprovacoes_cli`, desbloqueando a geração de fatura em `cobrancas`.
    

## 2.7 Pagamento a fornecedores

- Fornecedores externos registrados em `fornecedores` podem ser associados a documentos ou etapas específicas do projeto.
    
- Os pagamentos realizados são registrados em `pagamentos_forn`, mantendo rastreabilidade com o contrato e o documento que originou o custo.

---

## 3. Exemplo completo (caminho ideal)

| Etapa | O que acontece                                              | Tabelas                                              |
| ----- | ----------------------------------------------------------- | ---------------------------------------------------- |
| 1     | Cliente entra em contato → chamado CH-778 registrado        | `chamados`                                           |
| 2     | Pré-contrato PC-145-A redigido com estimativa de R$ 120.000 | `pre_contratos`                                      |
| 3     | Orçamento ORC-145-v0 aprovado                               | `orcamentos`                                         |
| 4     | Contrato CT-2025-001 é assinado                             | `contratos`                                          |
| 5     | Lista de preços unitários importada (ENG-001...)            | `precos_unitarios`                                   |
| 6     | Projeto “Subestação Elétrica” criado                        | `projetos` (id = 8)                                  |
| 7     | Ciclo Maio/25 iniciado                                      | `ciclos` (id = 5)                                    |
| 8     | Documento DOC-E-001 registrado                              | `documentos` (id = 55) + `responsaveis_documento`    |
| 9     | Primeira revisão A enviada ao cliente                       | `revisoes_doc`, `emissoes_documento (ultima = true)` |
| 10    | Cliente rejeita por legenda ausente “LEG-FALT”              | `rejeicoes_revisao`, código de `tipos_rejeicao`      |
| 11    | Revisão B aprovada                                          | Nova `revisoes_doc`, `emissoes_documento` atualizada |
| 12    | 3 horas de licença CAD usadas                               | `uso_recursos` (quantidade = 3)                      |
| 13    | Medição: 1 doc × R$ 8.000                                   | `medicoes_documento` (valor = 8.000)                 |
| 14    | Entrega parcial registrada (70%)                            | `entregas_ciclo` (valor = 5.600)                     |
| 15    | Cliente aprova → `aprovacoes_cli`                           | (`documento_id = 55`)                                |
| 16    | Fatura FAT-2025-201 emitida                                 | `cobrancas` (valor = 5.600)                          |
| 17    | Fornecedor “Desenhos Ltda” recebe 30%                       | `fornecedores`, `pagamentos_forn`                    |
| 18    | Contrato encerrado                                          | `contratos.updated_at` atualizado                    |


## 4. Por que isso importa

- **Rastreabilidade**: Cada versão, motivo de rejeição, custo e pagamento está ligado por chaves estrangeiras até o contrato.
    
- **Faturamento progressivo**: `medicoes_documento` + `entregas_ciclo` permitem cobrança parcial sem perder o controle da entrega final.
    
- **Responsabilidade de recursos**: Recursos internos e pagamentos externos são rastreados por entrega, permitindo controle real de P&L (lucros e perdas).
    
- **Integridade dos dados**: Índices únicos compostos (`documentos`, `revisoes_doc`, `precos_unitarios`) evitam duplicatas e facilitam consultas determinísticas.
    

🧠 **Recomendação final**: Comunicação mais segura e direta com as partes envolvidas, Incode <--> projetta

## **Atualização 2.0 29/052025**
Estrutura do Banco de Dados Atualizada – Supabase/PostgreSQL

1. Pré-contratos adicionados
	•	Antes: Chamado ia direto para Contrato.
	•	Agora: Chamado → Pré-Contrato → Contrato.
	•	Garante controle da negociação antes de formalizar o contrato.
	•	Campos como valor estimado, status e data de validade agora estão centralizados.

2. Relações entre tabelas ajustadas
	•	Todas as foreign keys têm regras de exclusão (ON DELETE SET NULL ou CASCADE).
	•	Os vínculos foram organizados para garantir integridade referencial.

3. Campos updated_at automatizados
	•	Função + trigger aplicados nas tabelas contratos, projetos, documentos e ciclos.
	•	Sempre que um registro é atualizado, o updated_at muda automaticamente.

4. Chaves únicas e índices aplicados
	•	contratos.codigo, documentos (projeto_id, codigo_doc), precos_unitarios (contrato_id, codigo).
	•	Evita duplicidade e melhora performance nas consultas.

5. Tabelas bem divididas por tema
	•	Atendimento: chamados, pre_contratos
	•	Contratos: contratos, ciclos, projetos
	•	Produção técnica: documentos, revisões, emissões
	•	Medição/cobrança: medicoes, entregas_ciclo, cobrancas
	•	Suprimentos: recursos, uso, fornecedores, pagamentos
	•	Equipe: usuarios, responsaveis_documento

🎯 Resultado
	•	O banco agora reflete o fluxo real da Projeta.
	•	Mais controle, rastreabilidade e pronto para uso com Supabase e automações.