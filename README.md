
# Projeto SAMBA
## Servidor de Compartilhamento Multi-Departamental

> Implementação prática de servidor SAMBA com controle de acesso por usuário/grupo em Linux

---

## 📌 Status do Projeto

- **Linguagem:** Bash / Shell Script
- **SO:** Linux (Debian/Ubuntu)
- **Serviço:** SAMBA 4.x (SMB/CIFS)
- **Status:** ✅ Completo e Funcional
- **Autor:** Vitor Fernandes da Silva

---

## 📖 Sobre Este Projeto

Este é um **case study prático** de implementação de servidor SAMBA em ambiente corporativo, desenvolvido durante meus estudos de infraestrutura no SENAC.

**Objetivo:** Compartilhar conhecimento prático sobre como integrar Linux e Windows de forma segura, com controle de acesso granular baseado em usuários e grupos.

### Para Quem É Útil?

- 👨‍🎓 Estudantes de cursos técnicos (SENAC, Impacta, etc)
- 👶 Juniors começando em infraestrutura de TI
- 🔧 Profissionais que querem entender Linux na prática
- 📚 Quem busca documentação didática de projetos reais

---

## 🎯 Conceitos Práticos

Neste projeto você vai dominar:

**Linux Fundamentals**
- Estrutura de diretórios (FHS - `/var`)
- Criação e gerenciamento de usuários (`useradd`, `usermod`)
- Criação e gerenciamento de grupos (`groupadd`, `chgrp`)
- Permissões de arquivo e pastas (`chmod 770`)
- Gerenciamento de serviços (`systemctl`)

**Segurança e Acesso**
- Controle de acesso baseado em grupos
- Princípio do menor privilégio
- Segregação de dados por departamento
- Autenticação em ambiente híbrido (Linux + Windows)

**Networking**
- Protocolo SMB/CIFS
- Comunicação entre Linux e Windows
- Compartilhamentos de rede
- Acesso remoto via UNC (`\\IP\compartilhamento`)

---

## 🏗️ Arquitetura

```
SERVIDOR LINUX
├── Serviço SAMBA (SMB/CIFS)
├── 11 Grupos Linux
│   └── administracao, marketing, ti, financeiro, etc
├── Diretórios em /var/
│   ├── /var/administracao
│   ├── /var/marketing
│   ├── /var/ti
│   └── ... (11 total)
└── Permissões (chmod 770)
    ├── Proprietário: rwx
    ├── Grupo: rwx (ACESSO)
    └── Outros: --- (BLOQUEADO)
```

**Clientes Windows 10/11**
```
Acesso: \\IP_DO_SERVIDOR
Autenticação: usuário + senha SAMBA
Navegação: Explorador de Arquivos
```

---

## 🚀 Guia de Implementação

### PASSO 1: Instalação Inicial

```bash
# Atualizar sistema
sudo apt update && sudo apt upgrade -y

# Instalar SAMBA
sudo apt install samba -y

# Iniciar serviço
sudo systemctl start smbd

# Ativar no boot
sudo systemctl enable smbd

# Verificar status
sudo systemctl status smbd
```

---

### PASSO 2: Criar Estrutura de Pastas

```bash
cd /var

# Criar 11 pastas departamentais
sudo mkdir -p administracao atendimento compras contabilidade \
  ecommerce financeiro logistica marketing rh ti vendas

# Verificar criação
ls -la /var | grep -E "administracao|marketing|ti"
```

**Resultado:**
```
drwxr-xr-x 2 root root 4096 Feb 6 10:15 administracao
drwxr-xr-x 2 root root 4096 Feb 6 10:15 marketing
drwxr-xr-x 2 root root 4096 Feb 6 10:15 ti
```

---

### PASSO 3: Criar Grupos Linux

```bash
# Criar um grupo para cada departamento
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

# Verificar
cat /etc/group | tail -11
```

**Ou usar loop (mais rápido):**
```bash
for dept in administracao atendimento compras contabilidade ecommerce \
  financeiro logistica marketing rh ti vendas; do
  sudo groupadd $dept
done
```

---

### PASSO 4: Atribuir Grupos às Pastas

```bash
cd /var

# Conectar cada pasta ao seu grupo
sudo chgrp administracao administracao
sudo chgrp atendimento atendimento
sudo chgrp compras compras
sudo chgrp contabilidade contabilidade
sudo chgrp ecommerce ecommerce
sudo chgrp financeiro financeiro
sudo chgrp logistica logistica
sudo chgrp marketing marketing
sudo chgrp rh rh
sudo chgrp ti ti
sudo chgrp vendas vendas

# Verificar
ls -la /var | grep -E "administracao|marketing|ti"
```

**Resultado:**
```
drwxr-xr-x 2 root administracao 4096 Feb 6 10:15 administracao
drwxr-xr-x 2 root marketing     4096 Feb 6 10:15 marketing
drwxr-xr-x 2 root ti            4096 Feb 6 10:15 ti
```

---

### PASSO 5: Definir Permissões (chmod 770)

**ESTE É O PASSO CRÍTICO DE SEGURANÇA**

```bash
cd /var

# Aplicar chmod 770 em todas as pastas
sudo chmod 770 administracao atendimento compras contabilidade \
  ecommerce financeiro logistica marketing rh ti vendas

# Verificar
ls -la /var | grep -E "administracao|marketing|ti"
```

**Resultado:**
```
drwxrwx--- 2 root administracao 4096 Feb 6 10:15 administracao
drwxrwx--- 2 root marketing     4096 Feb 6 10:15 marketing
drwxrwx--- 2 root ti            4096 Feb 6 10:15 ti
```

**Entendendo chmod 770:**
```
7 = Proprietário (rwx = 4+2+1)
7 = Grupo (rwx = 4+2+1) ← ACESSO TOTAL
0 = Outros (--- = sem acesso) ← BLOQUEADO
```

**Por que 770 e não 777?**
- `777` = Qualquer pessoa acessa ❌ INSEGURO
- `770` = Só o grupo autorizado ✅ SEGURO
- Bloqueamos "outros" completamente

---

### PASSO 6: Criar Usuários

```bash
# Exemplo: criar usuário Murilo para TI
sudo useradd -m murilo

# Atribuir ao grupo TI
sudo usermod -g ti murilo

# Definir senha Linux
sudo passwd murilo
# Digite: padrao1234
# Confirme: padrao1234

# Verificar
id murilo
# uid=1001(murilo) gid=1002(ti) groups=1002(ti)
```

---

### PASSO 7: Configurar SAMBA (smb.conf)

```bash
sudo nano /etc/samba/smb.conf
```

**Ir para a seção [Share Definitions] e adicionar:**

```ini
[ti]
   path = /var/ti
   valid users = @ti
   write list = @ti
   browseable = yes
   read only = no
   create mask = 0770
   directory mask = 0770

[marketing]
   path = /var/marketing
   valid users = @marketing
   write list = @marketing
   browseable = no
   read only = no
   create mask = 0770
   directory mask = 0770

[administracao]
   path = /var/administracao
   valid users = @administracao
   write list = @administracao
   browseable = yes
   read only = no
   create mask = 0770
   directory mask = 0770

[financeiro]
   path = /var/financeiro
   valid users = @financeiro
   write list = @financeiro
   browseable = no
   read only = no
   create mask = 0770
   directory mask = 0770
```

**Salvar:** `Ctrl + X` → `Y` → `Enter`

---

### PASSO 8: Criar Senha SAMBA

**IMPORTANTE: Diferente da senha Linux!**

```bash
# Criar senha SAMBA para Murilo
sudo smbpasswd -a murilo

# Digite: Senac@123
# Confirme: Senac@123

# Ativar usuário
sudo smbpasswd -e murilo
```

**Diferença de senhas:**
- **Linux:** `passwd` → `padrao1234` → SSH/Terminal
- **SAMBA:** `smbpasswd` → `Senac@123` → Windows/SMB

---

### PASSO 9: Validar e Reiniciar

```bash
# Validar sintaxe
testparm
# Deve retornar: "Loaded services file OK"

# Reiniciar SAMBA
sudo systemctl restart smbd

# Verificar status
sudo systemctl status smbd
```

---

### PASSO 10: Testar do Windows

**No Windows 10/11:**

1. Abra **Explorador de Arquivos**
2. Na barra de endereços: `\\192.168.X.X` (IP do servidor)
3. Credenciais:
   - Usuário: `murilo`
   - Senha: `Senac@123`

**Teste:**
- ✅ Crie arquivo em `\\IP\ti` (funciona)
- ❌ Tente em `\\IP\marketing` (nega acesso)

---

## 🔒 Matriz de Segurança

| Usuário | Grupo | TI | Marketing | Financeiro | Admin |
|---------|-------|-----|-----------|-----------|--------|
| murilo | ti | ✅ RW | ❌ Bloqueado | ❌ Bloqueado | ❌ Bloqueado |
| carlos | marketing | ❌ Bloqueado | ✅ RW | ❌ Bloqueado | ❌ Bloqueado |
| patricia | financeiro | ❌ Bloqueado | ❌ Bloqueado | ✅ RW | ❌ Bloqueado |
| root | admin | ✅ RW | ✅ RW | ✅ RW | ✅ RW |

---

## 🐛 Troubleshooting

### ❌ "Permissão negada"

```bash
# 1. Verificar usuário
id murilo

# 2. Verificar grupo
groups murilo

# 3. Verificar pasta
ls -la /var/ti

# 4. Testar acesso local
sudo -u murilo touch /var/ti/teste.txt
```

### ❌ "Senha inválida"

```bash
# Resetar senha SAMBA
sudo smbpasswd murilo

# Listar usuários SAMBA
sudo pdbedit -L
```

### ❌ "Servidor não encontrado"

```bash
# 1. Ver IP
hostname -I

# 2. Status SAMBA
sudo systemctl status smbd

# 3. Firewall
sudo ufw allow 137,138,139,445/tcp
```

### ❌ "Pasta sem acesso"

```bash
# Verificar proprietário
ls -la /var/ti

# Corrigir grupo
sudo chgrp ti /var/ti

# Corrigir permissão
sudo chmod 770 /var/ti

# Recarregar
sudo systemctl reload smbd
```

---

## 📚 Próximos Passos

- [ ] Implementar cotas de disco por grupo
- [ ] Backup automático dos compartilhamentos
- [ ] Logs de auditoria (quem acessou o quê)
- [ ] VPN para acesso remoto
- [ ] Active Directory (LDAP)
- [ ] Docker para portabilidade
- [ ] Monitoramento (Prometheus + Grafana)

---

## 🔗 Links Úteis

- [SAMBA Official](https://www.samba.org/)
- [Linux FHS Standard](https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard)
- [Man Pages](https://man7.org/)

---

## 🤝 Como Contribuir

1. Faça um fork
2. Crie uma branch: `git checkout -b feature/melhoria`
3. Commit: `git commit -m 'Descrição da melhoria'`
4. Push: `git push origin feature/melhoria`
5. Abra um Pull Request

---

## 💬 Dúvidas?

Deixe uma issue no repositório ou me contacte:
- **GitHub:** [@vifernandestech](https://github.com/vifernandestech)
- **LinkedIn:** [Vitor Fernandes](https://www.linkedin.com/in/seu-perfil)

---

## 📄 Licença

MIT License - Você é livre para usar, modificar e distribuir.

---

**Criado com ❤️ por Vitor Fernandes da Silva**
*Junior IT Technician | Linux Enthusiast*
**Fevereiro, 2026**
```



***

**Conseguiu copiar? Tá pronto para subir no GitHub agora? 🚀**
