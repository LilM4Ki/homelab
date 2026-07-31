# 03 — Serviços Base: Pi-hole (DNS) e Samba

Ambos os serviços rodam na mesma VM (`ubuntu-server`), separando responsabilidades de infraestrutura de suporte dos labs vulneráveis propriamente ditos.

## Pi-hole (DNS)

O Pi-hole roda em container Docker dentro da `ubuntu-server` e atua como resolvedor DNS para a rede, bloqueando domínios maliciosos e de telemetria/anúncios antes que a resolução chegue até os dispositivos.

**Por que isso importa para segurança:** bloqueio de DNS é uma camada barata e eficaz de defesa — corta comunicação com domínios de C2 (command-and-control) conhecidos e reduz a superfície de telemetria não desejada, sem precisar de agente em cada máquina.

### **Dashboard Pi-Hole and Upstream DNS Server**

<img width="600" height="400" alt="Dashboard Pi-Hole" src="https://github.com/user-attachments/assets/110d3422-8829-424c-9662-80d26a236bc4" />
<img width="250" height="400" alt="Upstream DNS Server" src="https://github.com/user-attachments/assets/b0ab4764-1953-49ff-99e4-1e261748dcd5" />

# Exemplo de instalação via docker-compose
**Arquivo ``docker-compose.yml``**
```bash
services:
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "80:80/tcp"
      - "443:443/tcp"
    environment:
      TZ: 'America/Sao_Paulo'
      FTLCONF_webserver_api_password: 'REDACTED'
      FTLCONF_dns_listeningMode: 'ALL'
    volumes:
      - './etc-pihole:/etc/pihole'
      - './etc-dnsmasq.d:/etc/dnsmasq.d'
    cap_add:
      - SYS_NICE
    restart: unless-stopped
```

## Samba

O Samba compartilha arquivos dentro da rede do homelab — utilizado para o armazenamento de materiais de estudo, entre outros tipos de dados.

**Hardening de permissões no Sistema de Arquivos**
```bash
user@servidor:/$ ls -ls
4 drwxrws--- 6 admin_user network_group 4096 Jul 29 15:30 pasta_principal
```

```bash
user@servidor:/$ ls -ls /pasta_principal
4 drwxrws---  2 admin_user  network_group 4096 jul 29 15:30 pasta_publica
4 drwxr-s--- 16 admin_user  network_group 4096 jul 29 15:30 pasta_pessoal
```
> **Análise das Permissões (OS Level):**
> - **Isolamento Estrito (``---``):** Todas as pastas terminam sem permissões para a classe Others. Isso garante que qualquer usuário ou serviço comprometido fora do grupo ``network_group`` tenha acesso zero (leitura, escrita ou execução) aos dados, prevenindo movimentação lateral.
> - **Herança de Grupo via SGID (``s``):** O bit especial (Set-Group-ID) foi ativado nas permissões de grupo (``drwxrws---`` e ``drwxr-s---``). Isso força o sistema a garantir que qualquer arquivo criado dentro dessas pastas pertença automaticamente ao network_group, mantendo a consistência do acesso colaborativo sem intervenção manual.
> - **Limitação Física da Pasta Pessoal:** Note que a pasta_pessoal possui permissão ``drwxr-s---``. No nível do sistema de arquivos, o grupo possui apenas permissão de leitura (r-x). A autorização final para escrita é delegada e controlada pela camada superior (o serviço Samba).

**Arquivo `smb.conf`**
```bash
[pasta_principal]
        comment = pasta principal
        path = /pasta_principal
        valid users = @network_group
        invalid users = root
        browseable = yes
        read only = no
        guest ok = no
        create mask = 0660
        directory mask = 0770

[pasta_publica]
        comment = pasta para todos utilizarem
        path = /pasta_principal/pasta_publica
        valid users = @network_group
        invalid users = root
        browseable = yes
        read only = no
        guest ok = no
        create mask = 0660
        directory mask = 0770
```
> **Políticas de Acesso (Samba Level):**
> - **Defesa em Profundidade (``invalid users = root``):** O usuário de maior privilégio do sistema foi explicitamente bloqueado de acessar os compartilhamentos via rede. Em caso de roubo de credenciais administrativas, elas não poderão ser usadas para exfiltração de dados via protocolo SMB.
> - **Negação Explícita (``guest ok = no``):** Rejeita qualquer tentativa de conexão anônima ou não autenticada na rede local.
> - **RBAC - Role-Based Access Control (``valid users = @network_group``):** O acesso não é feito por indivíduo, mas sim por mapeamento de grupo (``@``). Isso torna a infraestrutura escalável e fácil de auditar.

**Arquivo `smb.conf` pasta pessoal:**
```Bash
[pasta_pessoal]
        comment = pasta privada user 1
        path = /pasta_principal/pasta_pessoal
        valid users = @network_group
        invalid users = root
        browseable = yes
        write list = user1
        read only = yes
        guest ok = no
        create mask = 0640
        directory mask = 0750      
```
> **Isolamento de Usuário:**
> - Para garantir a privacidade na pasta_pessoal, aplicou-se o Princípio do Menor Privilégio. A pasta é configurada como estritamente leitura (read only = yes) para todos os membros do grupo validados. A única exceção para gravação é dada através da diretiva write list = user1, garantindo que apenas o dono legítimo possa alterar seus arquivos, enquanto a administração permanece padronizada.
>
> **Máscaras de Criação Restritas:**
> - **create mask = 0640 e directory mask** = 0750: Diferente da pasta pública, a pasta pessoal adota um perfil onde o dono tem controle total (leitura/escrita), o grupo tem apenas permissão de leitura, e o restante da rede não tem acesso, alinhando a segurança do Samba perfeitamente com a segurança física do Linux.

## Alocação de recursos

Essa VM roda com 4 vCPU e teve a RAM reduzida para 4 GB depois de constatar que os dois serviços não exigem mais que isso — recurso liberado foi redirecionado para o restante do lab.
