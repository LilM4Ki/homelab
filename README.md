# Homelab de Segurança Ofensiva e Defensiva

## Sobre o projeto

Este repositório documenta, passo a passo, a infraestrutura de um homelab pessoal montado para estudo prático de segurança ofensiva e defensiva, dentro da minha transição de carreira para Segurança da Informação.

A ideia central: ter um ambiente vulnerável de propósito para atacar — e um SIEM observando tudo — sem colocar em risco a minha própria rede doméstica.

<img width="3200" height="2000" alt="diagrama-rede-homelab" src="https://github.com/user-attachments/assets/e5fa6b14-9052-4508-99a1-fd620490431e" />

## Stack utilizada

- Hardware: Dell PowerEdge T420 (2x Xeon E5-2403 v2, ~31 GB RAM)
- Hypervisor: Proxmox VE
- Máquina de administração: MacBook (Apple Silicon M1) — acesso remoto controlado às redes isoladas
- Firewall: pfSense (VM) - segmentação e roteamento entre as redes
- Ataque: Kali Linux
- Alvos: OWASP Juice Shop, DVWA, OWASP BWA
- DNS/rede: Pi-hole
- Compartilhamento de arquivos: Samba (File Server para casa)
- Monitoramento/SIEM: Wazuh

## Índice da documentação

- [x] [Virtualização com Proxmox](https://github.com/LilM4Ki/homelab/blob/main/01%20—%20Virtualização%20com%20Proxmox.md)
- [x] [Roteamento e Firewall com pfSense](https://github.com/LilM4Ki/homelab/blob/main/02%20—%20Roteamento%20e%20Firewall%20com%20pfSense.md)
- [x] [Serviços base — DNS e Samba](https://github.com/LilM4Ki/homelab/blob/main/03%20-%20Serviços%20Base%20-%20DNS%20e%20Samba.md)

### Em andamento...

- [ ] Monitoramento — Wazuh SIEM
- [ ] Simulação de ataque e defesa

## Roadmap

- [x] VM atacante dedicada (Kali/Parrot) rodando nativamente no Proxmox, em bridge isolado
- [x] Alternância de acesso à internet da rede de ataque conforme o contexto de uso (CTFs vs. lab interno)
- [ ] Monitoramento e alertas de tráfego tanto na rede de ataque quanto nos alvos, para validar que nenhum tráfego fora do esperado está passando pela regra de liberação manual
- [ ] Regras customizadas no Wazuh para detectar os próprios ataques executados nos labs (abordagem Purple Team)

## Nota

Endereços IP e detalhes específicos da rede doméstica foram omitidos ou generalizados intencionalmente. O objetivo é documentar **decisões de arquitetura e segurança**, não expor a topologia real da minha rede.

Autor: Renan Rodrigues Makiya · Em transição de carreira para Segurança da Informação [LinkedIn](https://www.linkedin.com/in/renan-rodrigues-makiya-636a951b3/)
