# 01 — Virtualização com Proxmox

## Instalação

O Proxmox VE foi instalado diretamente em um servidor Dell PowerEdge T420, atuando como hypervisor bare-metal (tipo 1) para todo o homelab. O host foi nomeado HOMELAB.

## Hardware disponível
|Recurso | Especificação|
| --- | --- |
|CPU |	2x Intel Xeon E5-2403 v2 @ 1.80GHz (8 cores totais, sem hyperthreading)
|RAM | ~31 GB|
|Armazenamento |	1 HD 1.84 TB|

## Divisão de recursos entre VMs
|VM  |	vCPU |	RAM  |	Função|
| --- | --- | --- | --- |
|ubuntu-server |	4  |	4 GB |	Pi-hole + Samba|
|pfSense | 1 | 1 GB | Firewall rede atacante e alvo|
| Kali | 4 |	8 GB |	Máquina de ataque|
|VM Linux (labs) |	4	 | 4 GB  |	DVWA em container e Juice Shop|
|OWASP BWA |	1  | 1 GB  |	Alvo vulnerável|
|Wazuh Server |	4  |	8 GB  |	SIEM|

## Redes virtuais (vmbr)

O Proxmox permite criar múltiplas Linux Bridges (vmbr), cada uma funcionando como um switch virtual independente. Nesse homelab, foram criadas três:

|Bridge  |	Porta física associada |	Propósito|
| --- | --- | --- |
|vmbr0 |	Sim (uplink para a rede doméstica) |	Gestão do host e saída padrão para a LAN|
|vmbr1 |	Não |	Rede alvo isolada (10.0.0.0/24) — onde vivem os labs vulneráveis|
|vmbr2	| Não  |	Rede de ataque isolada (10.0.1.0/24) — onde vive a VM do Kali|

***O ponto central da segmentação: vmbr1 e vmbr2 não têm nenhuma porta física de rede associada. Elas existem só dentro do Proxmox, então tudo que trafega nelas fica confinado ao próprio host, sujeito apenas às regras do pfSense  ver [( 02 — Roteamento e Firewall)](https://github.com/LilM4Ki/homelab/blob/main/02%20%E2%80%94%20Roteamento%20e%20Firewall%20com%20pfSense.md).***
