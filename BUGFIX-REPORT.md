# Relatório de Bugs — Ruflo Workspace

> Gerado em: 2026-06-08 | Auditoria de workflows CI/CD

## Resumo Executivo

Durante a auditoria de configuração dos workflows Ruflo, foram identificados **2 testes E2E falhando** no repositório ClubFlow que causam falhas repetidas no CI/CD. Os demais repositórios não apresentam falhas ativas.

## ClubFlow — Testes E2E com Falha

### Status
- **Workflow**: Frontend CI/CD (`frontend.yml`)
- **Job falhando**: `e2e` (Playwright)
- **Duração por execução**: ~3 min 17s
- **Execuções com falha**: 89+ execuções
- **Impacto financeiro**: Minutos dentro do incluso (billed: $0,00)

### Testes Falhando

#### Teste 1: `scheduled-queue-ordering.spec.ts:642`
```
[staging] > tests/scheduled-queue-ordering.spec.ts:642:5
> /fila-tennis — agendamentos respeitam horário e ordem
> dentro de um slot a ordem é por chegada (created_at), não alfabética
```
**Causa provavel**: Lógica de ordenação por `created_at` não está sendo respeitada.
**Prioridade**: Alta (afeta UX de fila de tênis)

#### Teste 2: `sport-queue-ordering.spec.ts:352`
```
[staging] > tests/sport-queue-ordering.spec.ts:352:5
> /fila-tennis — ordem e liberação
> mantém ordem por chegada e adiciona nova reserva no final
```
**Causa provavel**: Insercão de nova reserva não respeita posição correta na fila.
**Prioridade**: Alta (afeta lógica de fila de quadras)

### Métricas do Ultimo Run
- **Testes passando**: 10 de 10 (job `frontend`)
- **Testes não rodados no E2E**: 144
- **Testes E2E falhando**: 2
- **Exit code**: 1 (Process completed with exit code 1)

## Ações Recomendadas

### Para ClubFlow (Prioridade Alta)

1. **Investigar a lógica de ordenação da fila**
   - Arquivo: `tests/scheduled-queue-ordering.spec.ts` (linha 642)
   - Arquivo: `tests/sport-queue-ordering.spec.ts` (linha 352)
   - Verificar queries SQL/Supabase que buscam agendamentos da fila
   - Garantir que `ORDER BY created_at ASC` está aplicado

2. **Verificar RLS (Row Level Security) do Supabase**
   - Checar se as políticas de RLS não estão filtrando registros indevidamente
   - Testar com service_role key para isolar o problema

3. **Corrigir a inserção de reservas na fila**
   - Verificar funções que adicionam reservas à fila
   - Garantir que novas reservas vão para o final da fila (maior `position` ou maior `created_at`)

4. **Adicionar logs de diagnóstico nos testes**
   ```typescript
   // Antes da assertion, adicionar:
   console.log('Queue order:', await page.evaluate(() => 
     document.querySelectorAll('[data-testid="queue-item"]')
       .map(el => el.textContent)
   ));
   ```

### Para todos os repositórios

- Os testes unitarios (Vitest) do ClubFlow estão passando (10/10)
- BRIDGE, VetBooking, PetSkin, CONSULTORIA-ECOCIL, LACONELLI: sem falhas ativas identificadas
- Os workflows Ruflo Multi-Agent CI/CD estão com status pendente de ativação dos secrets

## Monitoramento de Custos

| Métrica | Valor |
|---|---|
| Custo bruto Actions (junho/2026) | $9,92 |
| Custo cobrado | $0,00 |
| Uso incluso no plano Pro | $9,96 |
| Budget configurado | $5,00 (com bloqueio) |
| Status geral | **DENTRO DO LIMITE GRATUITO** |

## Links de Referência

- [ClubFlow Actions](https://github.com/antonioneto19/ClubFlow/actions)
- [ClubFlow Frontend Workflow](https://github.com/antonioneto19/ClubFlow/blob/main/.github/workflows/frontend.yml)
- [GitHub Billing](https://github.com/settings/billing/usage)
- [GitHub Budgets](https://github.com/settings/billing/budgets)
