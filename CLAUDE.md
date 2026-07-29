# Reclaimed Compute — Contexto do Projeto

> Este arquivo é lido automaticamente pelo Claude Code sempre que iniciado nesta pasta. Ele resume o projeto, as decisões de arquitetura já tomadas, e o estado atual do setup — para que qualquer sessão de trabalho comece com o contexto certo, sem precisar reexplicar do zero.

## Visão geral

**Reclaimed Compute** (nome de trabalho) é um projeto pessoal/experimental para transformar dispositivos obsoletos — celulares, PCs antigos e, futuramente, outras arquiteturas — em nós de recursos computacionais reutilizáveis. O primeiro caso de uso concreto é um hub de **automação residencial e de rede**. A visão de longo prazo é um framework reaproveitável por uma comunidade, mas essa generalização só deve acontecer depois de haver evidência real de uso (ver Roadmap abaixo).

Dois dispositivos fazem parte do projeto:
- **`device-old-pc/`** — PC antigo, i5 de 8ª geração, SATA de 1TB, rodando Ubuntu Server. **Ativo, em desenvolvimento agora.**
- **`device-legion-duel/`** — Lenovo Legion Phone Duel (modelo original, 12GB RAM/256GB, Snapdragon 865+), com tela quebrada, sendo repurposed como hub headless. **Em standby**, aguardando ferramentas/serviço de reparo de bateria.

## Princípios de arquitetura (não-negociáveis)

- **Separação `core/` (agnóstico de hardware) vs `device-*/` (específico de hardware)** — nada é abstraído prematuramente; a separação emerge do trabalho real em `device-old-pc/`, não é desenhada antes de existir um segundo device para comparar.
- **IA restrita a papéis de criação/interpretação** — nunca execução direta e autônoma sobre rede/dispositivos.
- **Aprovação humana obrigatória** para qualquer ação sobre dispositivos conectados: fluxo Telegram *propose → confirm → apply*.
- **Home Assistant** deve ser acessado por uma interface abstrata no código, substituível sem reescrever a camada do bot.
- **Minimalismo deliberado**: nada é instalado "porque pode ser útil depois" — só o que a fase atual exige (por isso não instalamos snaps extras nem Prometheus ainda).

## Roadmap de camadas (visão teórica de longo prazo)

O projeto é pensado em três camadas: **Resource Layer** (o que cada máquina oferece), **Fabric Layer** (orquestração multi-nó), **Application Layer** (os "domain drivers", como automação residencial ou, futuramente, um servidor de contexto/inferência de IA). Fases:

- **Fase A — Validação de domínio único (fase atual):** provar que `core/` + `device-*/` funciona na prática, com um domain driver (automação) rodando no PC antigo.
- **Fase B — Extração da Resource Layer:** só depois que o celular também estiver ativo e comparável.
- **Fase C — Prova de multi-domínio.**
- **Fase D — Fabric Layer / multi-nó** (candidatos: k3s, Nomad).
- **Fase E — Domain driver de recursos de IA** (servidor de contexto/inferência distribuída).
- **Fase F — Abertura para a comunidade.**

Cada fase só é desbloqueada por evidência real da fase anterior — não se deve adiantar engenharia de fases futuras.

## Pilha de software planejada (Fase A, `device-old-pc/`)

Ordem de implementação:
1. AdGuard Home / Pi-hole — DNS e bloqueio
2. Tailscale — VPN
3. Hardening de acesso remoto
4. Home Assistant (atrás da interface abstrata)
5. Bot Telegram (propose → confirm → apply)
6. Camada de interpretação de logs
7. Inferência local via Ollama — modelos candidatos: Qwen2.5, Llama 3.2, Gemma 3, Phi-4-mini, e **LFM2.5** (família da Liquid AI, arquitetura feita para CPU/edge; candidatos específicos: LFM2.5-1.2B-Instruct e LFM2.5-230M, recomendados para tarefas agentic/tool-use/RAG, não para código)

## Terminologia do projeto

- O projeto se encaixa na categoria **homelab** (servidor doméstico com hardware reaproveitado), não em VPS (que é virtualizado, remoto, gerenciado por terceiro).
- Tecnicamente, os nós funcionam como **edge computing nodes** — hardware fraco/doméstico rodando cargas de trabalho localmente.

## Estado atual do `device-old-pc/` (checklist vivo)

- [x] Ubuntu Server instalado (26.04 LTS), sem pacotes de desktop
- [x] Conectividade via Wi-Fi (sem cabo disponível no momento da instalação)
- [x] IP fixo reservado no roteador (Huawei OptiXstar K562e-10, painel em `192.168.0.1`) — IP atual do servidor: `192.168.0.59`
- [x] Acesso SSH funcionando via autenticação por chave ED25519 (usuário: `thecounter66`)
- [x] Login por senha via SSH desabilitado (`PasswordAuthentication no` — nota: nesta instalação, a configuração efetiva está em um arquivo dentro de `/etc/ssh/sshd_config.d/`, não só no `sshd_config` principal)
- [x] `PermitRootLogin no`
- [x] Firewall (`ufw`) ativo, permitindo apenas OpenSSH
- [x] `unattended-upgrades` configurado
- [x] `HandleLidSwitch=ignore` configurado em `/etc/systemd/logind.conf` (necessário porque o "servidor" é fisicamente um notebook — sem isso, fechar a tampa suspende a máquina)
- [x] Claude Code instalado nativamente (`curl -fsSL https://claude.ai/install.sh | bash`), PATH ajustado em `~/.bashrc`
- [x] Repositório do projeto criado e versionado no GitHub: [`Samueldial/reclaimed-compute`](https://github.com/Samueldial/reclaimed-compute) (público; `gh` CLI autenticado como `Samueldial`, protocolo `https`; `.gitignore` na home usa allow-list para não versionar `.ssh`, `.cache`, `.claude` etc.)
- [x] Docker instalado (Docker Engine + Compose plugin, via repositório oficial; usuário `thecounter66` no grupo `docker`)
- [x] Homepage dashboard (gethomepage.dev) rodando via Docker em `device-old-pc/services/homepage/` — painel de status dos serviços do roadmap, acessível na LAN em `http://192.168.0.59:3030` (porta liberada no `ufw` só para `192.168.0.0/24`)
- [x] AdGuard Home rodando via Docker em `device-old-pc/services/adguard-home/` — DNS na porta 53 (preso ao IP `192.168.0.59`, não `0.0.0.0`, para não colidir com `systemd-resolved`), painel admin em `http://192.168.0.59:3000`; bloqueio testado e confirmado (blocklist padrão ativa). Ainda **não** configurado como DNS da rede toda no roteador — só validado neste servidor por enquanto.
- [x] Tailscale instalado nativamente (apt), servidor conectado à tailnet como `vostro66` (IP Tailscale `100.90.173.40`). Só este nó, sem subnet router (resto da LAN não está exposto). `ufw` libera toda entrada pela interface `tailscale0` (`ufw allow in on tailscale0`) — a autenticação real fica por conta do próprio Tailscale. Acesso remoto validado: Homepage e SSH acessíveis pelo IP Tailscale com o celular fora da rede local (dados móveis).
- [ ] Home Assistant
- [ ] Bot Telegram
- [ ] Camada de interpretação de logs
- [ ] Inferência local (Ollama)

## Notas de ambiente

- O usuário desenvolve a partir de um notebook pessoal Windows com **WSL** (usuário `samueldial`), separado do servidor (usuário `thecounter66`, hostname `vostro66`). Dev e alvo de deploy são propositalmente mantidos em máquinas diferentes.
- O `device-old-pc/` é fisicamente um notebook antigo, usado como servidor headless — a tela pode ser fechada com segurança (já configurado para não suspender).

## Como usar este arquivo

Atualize a seção "Estado atual" conforme os passos forem concluídos. Se decisões de arquitetura novas forem tomadas fora deste ambiente (ex: em conversas no Claude.ai), traga um resumo delas para cá manualmente — este arquivo não sincroniza automaticamente com outras conversas.
