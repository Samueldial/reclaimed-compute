# Reclaimed Compute

Projeto pessoal/experimental para transformar dispositivos obsoletos — celulares, PCs antigos e, futuramente, outras arquiteturas — em nós de recursos computacionais reutilizáveis. O primeiro caso de uso concreto é um hub de **automação residencial e de rede**, rodando num notebook antigo reaproveitado como servidor headless (homelab / edge computing).

Contexto completo de arquitetura, decisões e o checklist de estado vivo do projeto estão em [`CLAUDE.md`](./CLAUDE.md).

## Princípios de arquitetura

- **Separação `core/` (agnóstico de hardware) vs `device-*/` (específico de hardware)** — nada é abstraído prematuramente; a separação só emerge do trabalho real.
- **IA restrita a papéis de criação/interpretação** — nunca execução direta e autônoma sobre rede/dispositivos.
- **Aprovação humana obrigatória** para qualquer ação sobre dispositivos conectados.
- **Minimalismo deliberado** — só se instala o que a fase atual exige.

## Dispositivos

| Pasta | Descrição | Status |
|---|---|---|
| `device-old-pc/` | PC antigo, i5 8ª geração, Ubuntu Server | Ativo, em desenvolvimento |
| `device-legion-duel/` | Lenovo Legion Phone Duel, hub headless | Em standby (aguardando reparo de bateria) |

## Serviços rodando hoje (`device-old-pc`)

Todos containerizados via Docker Compose, um diretório por serviço em `device-old-pc/services/`.

| Serviço | Função | Acesso |
|---|---|---|
| Homepage | Painel de status do projeto | LAN e Tailscale, porta `3030` |
| AdGuard Home | DNS e bloqueio de anúncios | DNS `53`, admin `3000` (validado, ainda não é o DNS da rede toda) |
| Tailscale | VPN mesh para acesso remoto | — |

Roadmap completo (próximos passos, fases de longo prazo) em [`CLAUDE.md`](./CLAUDE.md).

## Segurança

Este repositório é público. Nenhuma chave, senha, token ou arquivo de configuração com credenciais é versionado — diretórios que armazenam esse tipo de dado (ex: `conf/` do AdGuard Home) são explicitamente excluídos via `.gitignore`.
