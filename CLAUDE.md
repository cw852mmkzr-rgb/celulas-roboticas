# Células Robóticas — Trenisp/Tremps

Sistema de monitoramento de 12 células de solda robótica (robôs 600 a 611).

## Arquitetura

- **Frontend**: `index.html` — single-file (~9000 linhas), HTML + CSS + JS puro
- **Backend**: Supabase (banco de dados + Edge Functions)
- **Deploy**: push na branch `main` do GitHub → Netlify atualiza automaticamente

## Edge Functions (Supabase)

| Função          | Descrição                                              |
| --------------- | ------------------------------------------------------ |
| `ia-fabrica`    | IA do sistema (chat de texto e assistente de voz MR.ROBOT) |
| `whatsapp-bot`  | Bot do WhatsApp via UazapiGO                           |
| `tts-voz`       | Geração de voz realista via ElevenLabs                 |

## Assistente de Voz — MR.ROBOT

- Esfera 3D animada como interface visual
- Conversa contínua (streaming)
- Voz gerada pela ElevenLabs (via Edge Function `tts-voz`)

## Base de Conhecimento da IA

Armazenada no Supabase, tabela `dados`, campo `base_conhecimento` com a estrutura:

- `geral` — conhecimento geral do sistema
- `infos` — informações operacionais
- `por_celula` — dados específicos de cada célula
- `erros` — catálogo de erros conhecidos

## Acesso

- Senha de admin: `3102`
