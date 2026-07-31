# 02 — Roteamento e Firewall com pfSense

## Interfaces

O pfSense roda como VM dentro do Proxmox. Não usei VLANs — a segmentação é feita através de interfaces físicas/virtuais distintas do pfSense, cada uma amarrada a um `vmbr` diferente (ver [01 — Virtualização com Proxmox](https://github.com/LilM4Ki/homelab/blob/main/01%20%E2%80%94%20Virtualiza%C3%A7%C3%A3o%20com%20Proxmox.md)):

| Interface pfSense | Bridge | Rede | Papel |
|---|---|---|---|
| WAN | `vmbr0` | Rede doméstica | Uplink para internet via roteador residencial |
| LAN | `vmbr1` | 10.0.0.0/24 | Rede alvo (máquinas vulneráveis) |
| LAN1 | `vmbr2` | 10.0.1.0/24 | Rede de ataque (Kali/Parrot) |

## Regras de firewall

### Interface LAN (rede alvo)

| Ordem | Origem | Destino | Ação | Descrição |
|---|---|---|---|---|
| 1 | 10.0.0.0/24 | Rede doméstica | Bloquear | Máquinas vulneráveis não acessam a LAN residencial |
| 2 | 10.0.0.0/24 | * (internet) | Bloqueado por padrão | Rede alvo nunca tem saída para a internet |

### Interface LAN1 (rede de ataque)

| Ordem | Origem | Destino | Ação | Descrição |
|---|---|---|---|---|
| 1 | 10.0.1.0/24 | Rede doméstica | Bloquear | Kali não acessa a LAN residencial |
| 2 | 10.0.1.0/24 | 10.0.0.0/24 | Permitir | Kali acessa a rede alvo |
| 3 | 10.0.1.0/24 | * (internet) | Alternado manualmente | Ligado durante CTFs, desligado durante uso do lab interno |

> **Ponto-chave de design:** o acesso entre as duas redes isoladas é **unidirecional**. O Kali enxerga e acessa as máquinas vulneráveis, mas elas não conseguem iniciar conexão de volta para o Kali — limita o impacto de qualquer tentativa de "retorno" originada em uma exploração na rede alvo.

### Interface WAN

| Ordem | Origem | Destino | Ação | Descrição |
|---|---|---|---|---|
| 1 | Máquina de administração | Painel pfSense | Permitir | Só esse host acessa o painel administrativo |
| 2 | Máquina de administração | 10.0.0.0/24 | Permitir | Acesso remoto à rede alvo |
| 3 | Máquina de administração | 10.0.1.0/24 | Permitir | Acesso remoto à rede de ataque |

## Acesso remoto

O acesso ao túnel RDP e ao SSH do Kali é feito com **autenticação por chave SSH (sem senha)** — reduz a superfície de ataque contra a própria máquina de administração, já que não há senha para forçar por brute-force.
