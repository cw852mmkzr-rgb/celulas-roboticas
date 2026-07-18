# Células Robóticas — Trenisp/Tremps

Sistema de monitoramento de 12 células de solda robótica (robôs 600 a 611 — 5 FANUC + 7 KUKA).

Desenvolvido pelo Eduardo.

## Arquitetura

- **Frontend**: `index.html` — single-file (~9500+ linhas), HTML + CSS + JS puro
- **Backend**: Supabase (banco de dados + Edge Functions)
- **Deploy**: push na branch `main` do GitHub → Netlify + GitHub Pages atualizam automaticamente
- **Repositório**: `cw852mmkzr-rgb/celulas-roboticas` (público)

## Arquitetura do banco

Padrão principal: **"linha única com colunas JSON"**
- Tabela `dados`, linha `id="main"`
- Cada funcionalidade fica numa coluna `jsonb` separada
- Tabelas adicionais separadas quando o volume/complexidade justifica (ex: `componentes`, `componentes_pecas`, `componentes_historico`, `bot_sessoes`)

**Autenticação:** service role key configurada no sistema. Sempre usar service role para escrita.

**RLS:** ativado nas tabelas sensíveis (ex: `bot_sessoes`, tabelas de componentes).

## Edge Functions (Supabase)

| Função          | Descrição                                              |
| --------------- | ------------------------------------------------------ |
| `ia-fabrica`    | IA do sistema (chat de texto e assistente de voz MR.ROBOT) |
| `whatsapp-bot`  | Bot do WhatsApp via UazapiGO                           |
| `tts-voz`       | Geração de voz realista via ElevenLabs (voz masculina Adam) |

## Integrações externas

- **UazapiGO** — plano pago "Por Dispositivo"
  - Servidor: `celulasroboticas.uazapi.com`
  - Número conectado: `5511915267082`
- **ElevenLabs** — voz TTS (Adam)

## Assistente de Voz — MR.ROBOT

- Esfera 3D animada (roxo/verde) como interface visual
- Conversa contínua (streaming)
- Voz gerada pela ElevenLabs (via Edge Function `tts-voz`)
- Também roda como bot no WhatsApp com base de conhecimento completa

## Base de Conhecimento da IA

Armazenada no Supabase, tabela `dados`, campo `base_conhecimento` com a estrutura:
- `geral` — conhecimento geral do sistema
- `infos` — informações operacionais
- `por_celula` — dados específicos de cada célula
- `erros` — catálogo de erros conhecidos

Com sistema de backup/restaurar.

## Funcionalidades existentes

- Bot MR.ROBOT no WhatsApp com base de conhecimento completa
- Assistente de voz MR.ROBOT dentro do sistema
- Base de conhecimento IA (com backup/restaurar)
- Sistema de gabaritagem (com leitura de foto de planilha manuscrita — 3 colunas: CODIGO / QUANTIDADE (prog mês) / REALIZADO (valor após "="))
  - Leitura via Gemini Vision com preview antes de salvar
  - Códigos `PRB*` são criados automaticamente ao ler foto
  - Modo edição: renomear código existente e reordenar com ▲▼
  - Botão "Copiar códigos do mês anterior" (só no modo admin)
- Mapa de produção
- Células robóticas (monitoramento)
- **Controle de Componentes** — arquitetura N:N:
  - Tabela `componentes` (master com estoque único)
  - Tabela `componentes_pecas` (vínculo peça ↔ componente + qtd por peça)
  - Tabela `componentes_historico` (movimentações)
  - Automação: ao registrar produção nas Metas Diárias, subtrai automaticamente dos componentes vinculados
  - 2 níveis de acesso: admin geral + modo host (senha própria)
  - Botão pausar automação

## Acesso

- Senha de admin configurada diretamente no sistema

## Padrões de código a seguir

1. **Single-file:** todo código novo vai no `index.html` — não criar arquivos separados sem confirmar
2. **Padrão visual:** manter cores, tipografia, botões e modais consistentes com as abas existentes
3. **Idioma:** todos os textos em português
4. **Lazy load:** abas grandes só carregam dados quando o usuário abre
5. **Toasts/notificações:** para feedback de sucesso/erro/alertas
6. **Confirmação antes de excluir** qualquer registro
7. **Nunca quebrar fluxos existentes** — sempre verificar que a mudança não afeta funcionalidades já em produção

## ⚠️ BUGS VISUAIS — VISTORIA OBRIGATÓRIA

**Antes de fazer commit e push, SEMPRE verifique se a alteração introduziu algum destes bugs de tela. Estes bugs já aconteceram no projeto e NÃO PODEM se repetir:**

### Bugs a caçar em toda alteração:

1. **Scroll subindo sozinho** — a tela rola pro topo automaticamente ao interagir com botões, modais ou inputs. Causas comuns:
   - `href="#"` em botões (sempre use `type="button"` ou `preventDefault()`)
   - `<form>` sem `preventDefault()` no submit
   - Re-renderização completa de listas grandes
   - Foco automático em elementos fora da viewport

2. **Tela piscando (flickering)** — a interface pisca ao atualizar dados. Causas comuns:
   - Re-render desnecessário do DOM inteiro em vez de atualizar só o que mudou
   - Estilos aplicados via JS que sobrescrevem CSS constantemente
   - Loops de atualização sem `debounce`/`throttle`
   - Polling do Supabase sem comparar estado antes de re-renderizar

3. **Scroll bugado dentro de modais/abas** — scroll não funciona, trava, ou permite scroll no fundo enquanto modal está aberto. Causas comuns:
   - Faltar `overflow: hidden` no body quando modal abre
   - `overflow-y: auto` sem `max-height` definido no container
   - Múltiplos `position: fixed` conflitantes
   - Modal sem scroll interno próprio quando conteúdo é maior que a viewport

4. **Layout quebrando em telas menores** — elementos saem da tela, se sobrepõem, ou ficam ilegíveis
5. **Botões/inputs não clicáveis** — z-index incorreto ou elemento invisível por cima
6. **Modal não fecha** — ao clicar no X, no backdrop, ou pressionar Esc
7. **Toast/notificação empilhando infinito** — sem limite ou sem auto-dismiss
8. **Loading infinito** — spinner que nunca desaparece se a requisição falhar
9. **Dados sumindo ao trocar de aba** — estado não persistido corretamente
10. **Duplicação visual de elementos** — listas renderizando items duplicados por bug no `map`/loop

### Checklist antes do commit:

- [ ] Abrir a aba/modal alterado e navegar por ele completamente
- [ ] Testar em resolução desktop e mobile
- [ ] Rolar até o fim e voltar ao topo — o scroll se comporta bem?
- [ ] Clicar em vários botões seguidos — a tela pisca? Sobe sozinho?
- [ ] Abrir e fechar modais várias vezes — funciona sempre?
- [ ] Verificar o console do navegador — algum erro/warning novo?
- [ ] Nenhum `console.log` esquecido de debug

**Se algum destes bugs for detectado, CORRIJA ANTES de fazer o push. Não empurre bug pra produção.**

## Fluxo Git

Após qualquer implementação:
1. **Rodar o checklist de bugs visuais acima**
2. Commit com mensagem descritiva em português
3. Push na branch `main`
4. Netlify + GitHub Pages fazem deploy automático

## MCP disponível

- **MCP do Supabase** conectado em modo **read-only** — pode consultar o banco pra validar dados, verificar estruturas, confirmar que registros foram criados/deletados
- Para migrations (CREATE TABLE, etc), o Eduardo executa manualmente no Supabase Dashboard → SQL Editor

## Como abordar tarefas

1. **Ler o `index.html` completo antes de editar** — o arquivo é grande e as funcionalidades interagem
2. **Localizar a seção certa** — abas seguem padrão de nomes claros
3. **Verificar integrações existentes** — antes de criar algo novo, ver se já não existe função similar
4. **Testar com MCP Supabase** quando envolver banco — confirmar que INSERTs/DELETEs realmente aconteceram
5. **Rodar checklist de bugs visuais** antes do commit
6. **Explicar o que foi feito** ao final, principalmente se mexeu em fluxos críticos (produção, automações)

## Bugs conhecidos historicamente

- Função de desvincular componente já teve bug de não deletar do banco (só do estado local) — sempre confirmar que DELETEs vão até o Supabase
- Leitura de foto na gabaritagem: sempre usar prompt detalhado para OCR manuscrito (3 colunas, extrair valor após "=" no REALIZADO, remover separador de milhar)
- Scroll subindo sozinho em algumas abas — sempre validar comportamento de scroll após alterações
- Tela piscando em atualizações — evitar re-render completo, atualizar só o que mudou