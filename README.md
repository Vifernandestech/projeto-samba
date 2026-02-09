
# 🖥️ Servidor SAMBA no Kali Linux | Setup Corporativo

> Um projeto prático de **Administração de Sistemas Linux** com SAMBA rodando em **Kali Linux (VMware)**, demonstrando compartilhamento seguro entre Linux e Windows em um ambiente corporativo real.

![Status](https://img.shields.io/badge/Status-Completo-brightgreen)
![Linux](https://img.shields.io/badge/OS-Kali%20Linux-blue)
![SAMBA](https://img.shields.io/badge/Serviço-SAMBA%204.x-red)
![License](https://img.shields.io/badge/License-MIT-green)

---

## 🎯 Visão Geral

Este projeto implementa um **servidor SAMBA corporativo** do zero com:

✅ **11 compartilhamentos** organizados por departamento (Administração, Marketing, TI, Vendas, Contabilidade, Financeiro, Logística, RH, Compras, Atendimento, E-commerce)

✅ **Gerenciamento de usuários e grupos** Linux com controle de acesso granular

✅ **Permissões por departamento** (chmod 770) com segurança validada

✅ **Autenticação dupla** (Senha Linux ≠ Senha SAMBA)

✅ **Integração Windows → Linux** via `\\IP-SERVER` (Ambiente corporativo real)

✅ **Documentação completa** com screenshots de cada etapa

### Por que este projeto importa?

Em qualquer infraestrutura corporativa, você precisa fazer **Linux e Windows conversarem**. Este projeto:

- 🎓 Consolida conceitos de **Estrutura Linux (FHS)**
- 🎓 Aplica **Gerenciamento de Usuários e Permissões** na prática
- 🎓 Implementa **Controle de Acesso** baseado em grupos
- 🎓 Demonstra **Integração cross-platform** (SAMBA/SMB)
- 🎓 Desenvolve **Habilidades de Administração de Sistemas**

---

## 📦 Pré-Requisitos

### Hardware
- **VM VMware** com **Kali Linux** instalado
- Mínimo: 2GB RAM, 20GB disco, 1 vCPU
- Conexão de rede (Bridge ou NAT)

### Software
```bash
# Verifique a versão do Kali Linux
lsb_release -a

# Verifique permissões root
sudo whoami  # Deve retornar: root
```

### Conhecimentos Base
- ✅ Comandos Linux básicos (ls, mkdir, cd, pwd)
- ✅ Estrutura de diretórios (FHS - /, /bin, /etc, /var, /home)
- ✅ Usuários e grupos (conceito teórico)
- ✅ Permissões de arquivo (chmod, chown, chgrp - conceito)
- ✅ Editor de texto nano

---

## 🏗️ Arquitetura do Projeto

```
SERVIDOR SAMBA (Kali Linux)
│
├── Usuários Linux + Grupos (11 grupos)
│   ├── Grupo: administracao → pasta /var/administracao
│   ├── Grupo: marketing → pasta /var/marketing
│   ├── Grupo: ti → pasta /var/ti
│   └── ... (8 departamentos mais)
│
├── Configuração SAMBA
│   └── /etc/samba/smb.conf (11 compartilhamentos configurados)
│
└── Acesso de Clientes
    └── Windows Explorer: \\192.168.X.X
 
```

### Fluxo de Autenticação

```
Cliente Windows
       ↓
   Digite: \\192.168.X.X
       ↓
   Credenciais: murilo / ***********
       ↓
   SAMBA valida no arquivo /etc/samba/smbpasswd
       ↓
   Verifica se usuário está no grupo correto
       ↓
   ✅ Acesso autorizado → Pasta específica do departamento
```

---

## 🚀 Setup Passo a Passo

### **Etapa 1️⃣: Atualizar Sistema e Instalar SAMBA**

```bash
# Atualizar repositórios
sudo apt update && sudo apt upgrade -y

# Instalar SAMBA
sudo apt install smbd -y

# Iniciar o serviço
sudo systemctl start smbd

# Habilitar na inicialização
sudo systemctl enable smbd

# Verificar status
sudo systemctl status smbd

# Resultado esperado:
# ● smbd.service - Samba SMB Daemon
#    Loaded: loaded (/lib/systemd/system/smbd.service)
#    Active: active (running) ✓
```

### **Etapa 2️⃣: Criar Estrutura de Diretórios (/var)**

```bash
# Navegar para /var
cd /var

# Criar pastas para cada departamento (11 no total)
sudo mkdir -p administracao atendimento compras contabilidade ecommerce \
               financeiro logistica marketing rh ti vendas

# Verificar criação
ls -la /var 

# Resultado esperado:
# drwxr-xr-x  2 root root  4096 Fev  8 20:00 administracao
# drwxr-xr-x  2 root root  4096 Fev  8 20:00 atendimento
# drwxr-xr-x  2 root root  4096 Fev  8 20:00 compras
# ... (e assim por diante)
```

### **Etapa 3️⃣: Criar Grupos Linux (Um por Departamento)**

```bash
# Criar 11 grupos
sudo groupadd administracao
sudo groupadd atendimento
sudo groupadd compras
sudo groupadd contabilidade
sudo groupadd ecommerce
sudo groupadd financeiro
sudo groupadd logistica
sudo groupadd marketing
sudo groupadd rh
sudo groupadd ti
sudo groupadd vendas

# Verificar grupos criados
cat /etc/group

# Resultado esperado:
# administracao:x:1001:
# atendimento:x:1002:
# compras:x:1003:
# ... (e assim por diante)
```

### **Etapa 4️⃣: Atribuir Grupos aos Diretórios**

```bash
# Mudar proprietário do grupo para cada pasta
sudo chgrp administracao /var/administracao
sudo chgrp atendimento /var/atendimento
sudo chgrp compras /var/compras
sudo chgrp contabilidade /var/contabilidade
sudo chgrp ecommerce /var/ecommerce
sudo chgrp financeiro /var/financeiro
sudo chgrp logistica /var/logistica
sudo chgrp marketing /var/marketing
sudo chgrp rh /var/rh
sudo chgrp ti /var/ti
sudo chgrp vendas /var/vendas
```

### **Etapa 5️⃣: Definir Permissões (chmod 770)**

```bash
# Definir permissões 770 em cada pasta
# 7 = proprietário (rwx)
# 7 = grupo (rwx)  ← Aqui! O grupo consegue ler, escrever e executar
# 0 = outros (---) ← Bloqueado! Ninguém mais tem acesso

sudo chmod 770 /var/administracao
sudo chmod 770 /var/atendimento
sudo chmod 770 /var/compras
sudo chmod 770 /var/contabilidade
sudo chmod 770 /var/ecommerce
sudo chmod 770 /var/financeiro
sudo chmod 770 /var/logistica
sudo chmod 770 /var/marketing
sudo chmod 770 /var/rh
sudo chmod 770 /var/ti
sudo chmod 770 /var/vendas

# Verificar permissões
ls -la /var | grep -E "admin|atend|compras|contabil|ecomm|financ|logis|market|rh|ti|vendas"

# Resultado esperado (note: drwxrwx---):
# drwxrwx---  2 root administracao  4096 Fev  8 20:00 administracao
# drwxrwx---  2 root atendimento    4096 Fev  8 20:00 atendimento
# ... (note: 3 primeiros "rwx", 3 segundos "rwx", 3 últimos "---")
```

### **Etapa 6️⃣: Criar Usuários e Atribuir aos Grupos**

```bash
# Criar usuário de exemplo (Murilo - Departamento TI)
sudo useradd -m -d /home/murilo murilo

# Adicionar ao grupo "ti"
sudo usermod -g ti murilo

# Definir senha Linux
sudo passwd murilo
# Digite a senha quando solicitado: padrao1234

# Criar mais usuários (opcional)
sudo useradd -m mariana
sudo usermod -g marketing mariana
sudo passwd mariana  # Senha: padrao1234

# Verificar usuários e seus grupos
id murilo
# uid=1001(murilo) gid=1002(ti) groups=1002(ti)

id mariana
# uid=1002(mariana) gid=1003(marketing) groups=1003(marketing)
```

### **Etapa 7️⃣: Configurar SAMBA (smb.conf)**

```bash
# Abrir arquivo de configuração SAMBA com nano
sudo nano /etc/samba/smb.conf
```

**Dentro do nano:**

1. Pressione `Ctrl+End` para ir ao fim do arquivo
2. Adicione esta configuração (copie e cole no nano):

```ini
# ===== COMPARTILHAMENTOS DEPARTAMENTAIS =====

[administracao]
   path = /var/administracao
   valid users = @administracao
   write list = @administracao
   browseable = yes
   read only = no
   force create mode = 0770
   comment = Compartilhamento Administração

[marketing]
   path = /var/marketing
   valid users = @marketing
   write list = @marketing
   browseable = yes
   read only = no
   force create mode = 0770
   comment = Compartilhamento Marketing

[ti]
   path = /var/ti
   valid users = @ti
   write list = @ti
   browseable = yes
   read only = no
   force create mode = 0770
   comment = Compartilhamento TI

[vendas]
   path = /var/vendas
   valid users = @vendas
   write list = @vendas
   browseable = yes
   read only = no
   force create mode = 0770
   comment = Compartilhamento Vendas

[contabilidade]
   path = /var/contabilidade
   valid users = @contabilidade
   write list = @contabilidade
   browseable = yes
   read only = no
   force create mode = 0770
   comment = Compartilhamento Contabilidade

[financeiro]
   path = /var/financeiro
   valid users = @financeiro
   write list = @financeiro
   browseable = yes
   read only = no
   force create mode = 0770
   comment = Compartilhamento Financeiro

[logistica]
   path = /var/logistica
   valid users = @logistica
   write list = @logistica
   browseable = yes
   read only = no
   force create mode = 0770
   comment = Compartilhamento Logística

[rh]
   path = /var/rh
   valid users = @rh
   write list = @rh
   browseable = yes
   read only = no
   force create mode = 0770
   comment = Compartilhamento RH

[compras]
   path = /var/compras
   valid users = @compras
   write list = @compras
   browseable = yes
   read only = no
   force create mode = 0770
   comment = Compartilhamento Compras

[atendimento]
   path = /var/atendimento
   valid users = @atendimento
   write list = @atendimento
   browseable = yes
   read only = no
   force create mode = 0770
   comment = Compartilhamento Atendimento

[ecommerce]
   path = /var/ecommerce
   valid users = @ecommerce
   write list = @ecommerce
   browseable = yes
   read only = no
   force create mode = 0770
   comment = Compartilhamento E-commerce
```

3. Pressione `Ctrl+O` para salvar, depois `Enter`
4. Pressione `Ctrl+X` para sair do nano

```bash
# Validar sintaxe do arquivo
sudo testparm

# Resultado esperado:
# Load smb config files from /etc/samba/smb.conf
# Loaded services file OK.
# ✓ Syntax is OK
```

### **Etapa 8️⃣: Criar Senha SAMBA (Diferente da Senha Linux!)**

```bash
# Adicionar usuário SAMBA
sudo smbpasswd -a murilo
# Digite a senha SAMBA quando solicitado: Senac@123

# Resultado esperado:
# Added user murilo.
# New SMB password: Senac@123
# Retype new SMB password: Senac@123
# ✓ Updated smbpasswd

# ⚠️ IMPORTANTE:
# Senha Linux (useradd): padrao1234
# Senha SAMBA (smbpasswd): Senac@123
# Podem ser DIFERENTES!

# Adicionar outro usuário (opcional)
sudo smbpasswd -a mariana
# Senha: Senac@123
```

### **Etapa 9️⃣: Reiniciar SAMBA e Testar**

```bash
# Reiniciar o serviço SAMBA
sudo systemctl restart smbd

# Verificar status
sudo systemctl status smbd

# Resultado esperado:
# ● smbd.service - Samba SMB Daemon
#    Active: active (running) ✓

# Verificar se está ouvindo nas portas corretas
sudo ss -tlnp | grep smbd

# Resultado esperado:
# LISTEN  0  255  0.0.0.0:139   0.0.0.0:*  smbd
# LISTEN  0  255  0.0.0.0:445   0.0.0.0:*  smbd
```

---

## 📊 Estrutura de Compartilhamentos

| Departamento | Caminho Linux | Grupo Linux | Permissões | Usuários |
|---|---|---|---|---|
| Administração | `/var/administracao` | `administracao` | `770` | @administracao |
| Marketing | `/var/marketing` | `marketing` | `770` | @marketing |
| TI | `/var/ti` | `ti` | `770` | @ti |
| Vendas | `/var/vendas` | `vendas` | `770` | @vendas |
| Contabilidade | `/var/contabilidade` | `contabilidade` | `770` | @contabilidade |
| Financeiro | `/var/financeiro` | `financeiro` | `770` | @financeiro |
| Logística | `/var/logistica` | `logistica` | `770` | @logistica |
| RH | `/var/rh` | `rh` | `770` | @rh |
| Compras | `/var/compras` | `compras` | `770` | @compras |
| Atendimento | `/var/atendimento` | `atendimento` | `770` | @atendimento |
| E-commerce | `/var/ecommerce` | `ecommerce` | `770` | @ecommerce |

---

## 🔧 Configuração SAMBA

### Entendendo smb.conf

```ini
[nome_compartilhamento]
   path = /caminho/local                    # Onde a pasta está no Linux
   valid users = @grupo_samba               # Quem pode acessar (@ = grupo)
   write list = @grupo_samba                # Quem pode escrever
   browseable = yes                         # Visível ao listar compartilhamentos
   read only = no                           # Permite escrita
   force create mode = 0770                 # Permissão padrão para arquivos criados
   comment = Descrição                      # Descrição do compartilhamento
```

### Parâmetros Importantes

| Parâmetro | Função | Exemplo |
|---|---|---|
| `path` | Caminho absoluto da pasta | `/var/administracao` |
| `valid users` | Usuários com acesso | `@administracao` ou `murilo maria` |
| `write list` | Quem pode escrever | `@administracao` |
| `browseable` | Visível ao listar | `yes` / `no` |
| `read only` | Bloqueia escrita | `no` = permite escrita |
| `comment` | Descrição amigável | `Pasta do Departamento de TI` |

---

## 🧪 Teste de Conectividade

### **Teste 1: Do próprio Kali (Linux)**

```bash
# Listar compartilhamentos disponíveis
smbclient -L localhost -U murilo%Senac@123

# Resultado esperado:
#	Sharename       Type      Comment
#	---------       ----      -------
#	administracao   Disk      Compartilhamento Administração
#	marketing       Disk      Compartilhamento Marketing
#	ti              Disk      Compartilhamento TI
#	... (todos os 11 compartilhamentos)

# Conectar a um compartilhamento específico
smbclient //localhost/ti -U murilo%Senac@123

# Dentro do smbclient (prompt: smb: \>):
#   ls                 # Listar arquivos
#   put arquivo.txt    # Enviar arquivo
#   get arquivo.txt    # Baixar arquivo
#   quit               # Sair
```

### **Teste 2: De um Cliente Windows**

```batch
REM Abrir Explorador de Arquivos (Windows + E)
REM Na barra de endereço, digite:

\\192.168.X.X

REM Substitua X.X pelo IP do Kali
REM Exemplo: \\192.168.1.100

REM Quando solicitado:
REM   Usuário: murilo
REM   Senha: Senac@123

REM Resultado: Você verá todos os 11 compartilhamentos!
```

### **Teste 3: Montar Compartilhamento (Linux)**

```bash
# Criar ponto de montagem
sudo mkdir -p /mnt/samba_ti

# Montar compartilhamento
sudo mount -t cifs //localhost/ti -o username=murilo,password=Senac@123 /mnt/samba_ti

# Verificar montagem
mount | grep samba_ti

# Resultado esperado:
# //localhost/ti on /mnt/samba_ti type cifs (...)

# Listar conteúdo
ls -la /mnt/samba_ti

# Desmontar quando terminar
sudo umount /mnt/samba_ti
```

---

## 🔍 Troubleshooting

### **Problema: "Acesso Negado" ao conectar do Windows**

```bash
# Verificar se usuário existe em SAMBA
sudo pdbedit -L

# Resultado deve mostrar: murilo

# Se não aparecer, adicione:
sudo smbpasswd -a murilo

# Verificar status do SAMBA
sudo systemctl status smbd

# Se inativo, reinicie:
sudo systemctl restart smbd
```

### **Problema: Compartilhamento não aparece**

```bash
# Validar sintaxe smb.conf
sudo testparm

# Procure por erros no arquivo

# Se houver erro, verifique:
# - Aspas e parênteses corretos
# - Sem caracteres especiais fora do lugar
# - Formato de indentação

# Reinicie após corrigir:
sudo systemctl restart smbd
```

### **Problema: Porta 139 ou 445 já em uso**

```bash
# Verificar qual processo está usando
sudo ss -tlnp | grep -E "139|445"

# Matar processo (se necessário)
sudo killall smbd

# Reiniciar
sudo systemctl restart smbd
```

### **Problema: Não consegue escrever na pasta**

```bash
# Verificar permissões
ls -la /var/administracao

# Deve mostrar: drwxrwx--- (770)

# Se diferente, corrija:
sudo chmod 770 /var/administracao

# Verificar proprietário do grupo
ls -la /var/administracao | awk '{print $3, $4}'

# Deve mostrar: root administracao

# Se não, corrija:
sudo chgrp administracao /var/administracao
```

---

## 📚 Conceitos Aprendidos

### 1️⃣ **Estrutura Linux (FHS)**
- `/var` → Diretório para dados variáveis (logs, compartilhamentos)
- `/etc` → Configurações do sistema (smb.conf)
- `/home` → Home dos usuários

### 2️⃣ **Gerenciamento de Usuários**
```bash
useradd -m usuario      # Criar usuário com home
usermod -g grupo user   # Adicionar a grupo
passwd usuario          # Definir senha
```

### 3️⃣ **Gerenciamento de Grupos**
```bash
groupadd nome_grupo     # Criar grupo
chgrp grupo pasta       # Atribuir grupo à pasta
grep grupo /etc/group   # Listar usuários do grupo
```

### 4️⃣ **Permissões de Arquivo (chmod)**
```bash
chmod 770 pasta
# 7 = proprietário (read=4 + write=2 + execute=1 = 7)
# 7 = grupo (read=4 + write=2 + execute=1 = 7)
# 0 = outros (nenhuma permissão)
```

### 5️⃣ **Serviços Linux (systemctl)**
```bash
systemctl start smbd       # Iniciar
systemctl stop smbd        # Parar
systemctl restart smbd     # Reiniciar
systemctl status smbd      # Verificar
systemctl enable smbd      # Ativar na inicialização
```

### 6️⃣ **SAMBA e SMB/CIFS**
- SAMBA = Implementação Linux do protocolo SMB (Windows)
- Permite compartilhamento de arquivos entre Windows e Linux
- Autenticação baseada em usuários/grupos

### 7️⃣ **Autenticação Dupla**
- Senha Linux (`/etc/shadow`) ≠ Senha SAMBA (`/etc/samba/smbpasswd`)
- SAMBA usa seu próprio banco de senhas

---

⚠️ **Este projeto é para fins educacionais em laboratório**

---

## 👨‍💻 Autor

**Vitor Fernandes**
- 📧 Email: vifernandes.tech@gmail.com
- 🔗 LinkedIn: [linkedin.com/in/vifernandescybersec](https://www.linkedin.com/in/vifernandescybersec/)
- 🐙 GitHub: [@vifernandestech](https://github.com/vifernandestech)
- 📍 Localização: Santo André, SP - Brasil

---

## 🎓 Agradecimentos

- **SENAC** - Por proporcionar formação em Linux
- **Kali Linux** - Pela excelente distribuição
- **Comunidade de TI** - Pela troca de conhecimento
- **Meus mentores** - Pela orientação e aprendizado

---

**⭐ Se este projeto foi útil, deixe uma star! ⭐**

---

**Última atualização:** Fevereiro 2026  
**Status:** Projeto Completo ✅  
**Versão:** 1.0.0
```


