# projeto-samba

```markdown
# 🔗 Projeto SAMBA - Servidor de Compartilhamento Multi-Departamental

> **Implementação prática de servidor SAMBA com controle de acesso por usuário/grupo em Linux**

![Linux](https://img.shields.io/badge/Linux-Debian%2FUbuntu-red?style=flat-square&logo=linux)
![SAMBA](https://img.shields.io/badge/SAMBA-4.x-blue?style=flat-square)
![Status](https://img.shields.io/badge/Status-Completo-brightgreen?style=flat-square)
![Autor](https://img.shields.io/badge/Autor-Vitor%20Fernandes-orange?style=flat-square)

---

## 📚 Sobre este Projeto

Este é um **case study prático** de implementação de servidor SAMBA em Linux, desenvolvido durante meus estudos de infraestrutura no SENAC.

**O objetivo:** Compartilhar conhecimento prático sobre como integrar Linux e Windows em ambiente corporativo, com segurança e controle de acesso granular.

**Para quem é útil:**
- 👨‍🎓 Estudantes de cursos técnicos (SENAC, Impacta, etc)
- 👶 Juniors começando em infraestrutura
- 🔧 Quem quer entender Linux na prática (não só teoria)
- 📖 Quem busca documentação didática de projetos reais

---

## 🎯 O que você vai aprender

Neste projeto você vai ver na prática:

| Conceito | O que é | Por quê importa |
|----------|---------|-----------------|
| **Diretórios Linux** | Estrutura `/var`, FHS | Organização e segurança |
| **Grupos (groupadd)** | Agrupamento de usuários | Controle de acesso centralizado |
| **Permissões (chmod)** | 770 = rwxrwx--- | Segurança: quem acessa o quê |
| **Atribuição (chgrp)** | Associar grupo a pasta | Conectar usuário → pasta |
| **Usuários (useradd)** | Criar contas no Linux | Autenticação e responsabilidade |
| **SAMBA (smb.conf)** | Protocolo SMB/CIFS | Comunicação Linux ↔ Windows |
| **Integração Windows** | \\IP no Explorer | Acesso transparente |

---

## 🏗️ Arquitetura do Projeto

```
┌─────────────────────────────────────────┐
│   SERVIDOR LINUX (Debian/Ubuntu)        │
│                                         │
│  ┌──────────────────────────────────┐  │
│  │     Serviço SAMBA (SMB/CIFS)     │  │
│  └──────────────────────────────────┘  │
│           ↓                             │
│  ┌──────────────────────────────────┐  │
│  │   Grupos Linux (11 grupos)       │  │
│  │   administracao, marketing,      │  │
│  │   ti, financeiro, etc            │  │
│  └──────────────────────────────────┘  │
│           ↓                             │
│  ┌──────────────────────────────────┐  │
│  │   Diretórios em /var/            │  │
│  │   /var/administracao             │  │
│  │   /var/marketing                 │  │
│  │   /var/ti                        │  │
│  │   ... (11 no total)              │  │
│  └──────────────────────────────────┘  │
│           ↓                             │
│  ┌──────────────────────────────────┐  │
│  │   Permissões (chmod 770)         │  │
│  │   Proprietário: rwx              │  │
│  │   Grupo: rwx ← acesso            │  │
│  │   Outros: --x ← bloqueado        │  │
│  └──────────────────────────────────┘  │
└─────────────────────────────────────────┘
         ↓ SMB Protocol ↓
┌─────────────────────────────────────────┐
│   CLIENTES WINDOWS 10/11                │
│   Acesso: \\192.168.X.X                 │
│   Explorador: verifica compartilhamentos│
│   Autenticação: usuário + senha SAMBA   │
└─────────────────────────────────────────┘
```

---

## 📋 Passo a Passo da Implementação

### 1️⃣ Instalação e Preparação

**Atualizar repositórios e instalar SAMBA:**

```bash
# Atualizar pacotes
sudo apt update && sudo apt upgrade -y

# Instalar SAMBA
sudo apt install samba -y

# Iniciar serviço
sudo systemctl start smbd
sudo systemctl enable smbd

# Verificar status
sudo systemctl status smbd
```

**O que faz:**
- `apt update` = busca últimas versões dos pacotes
- `apt install samba` = instala o servidor SAMBA
- `systemctl start smbd` = inicia o serviço
- `systemctl enable smbd` = ativa pra iniciar automaticamente no boot

---

### 2️⃣ Criar Estrutura de Diretórios

**Criar pastas em `/var` para cada departamento:**

```bash
# Ir para /var
cd /var

# Criar todas as pastas de uma vez
sudo mkdir -p administracao atendimento compras contabilidade ecommerce financeiro logistica marketing rh ti vendas

# Verificar se foram criadas
ls -la /var | grep -E "administracao|marketing|ti"
```

**Resultado esperado:**
```
drwxr-xr-x  2 root root  4096 Feb  6 10:15 administracao
drwxr-xr-x  2 root root  4096 Feb  6 10:15 atendimento
drwxr-xr-x  2 root root  4096 Feb  6 10:15 compras
...
```

**Por quê `/var`?**
- `/var` = dados variáveis (arquivos que mudam)
- Separado do sistema operacional
- Espaço dedicado para serviços (SAMBA, Apache, etc)
- Boas práticas de Linux (FHS)

---

### 3️⃣ Criar Grupos Linux

**Um grupo para cada departamento:**

```bash
# Criar os grupos
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
cat /etc/group | tail -11
```

**O que aconteceu:**
- Cada `groupadd` cria uma entrada em `/etc/group`
- Grupos são identificadores de acesso
- Usuários podem pertencer a um ou mais grupos

**Comando atalho (mais rápido):**
```bash
for dept in administracao atendimento compras contabilidade ecommerce financeiro logistica marketing rh ti vendas; do
  sudo groupadd $dept
done
```

---

### 4️⃣ Atribuir Grupos aos Diretórios

**Conectar cada pasta ao seu grupo:**

```bash
# Ir para /var
cd /var

# Atribuir grupos
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

# Verificar atribuição
ls -la /var | grep -E "administracao|marketing|ti"
```

**Resultado:**
```
drwxr-xr-x  2 root administracao  4096 Feb  6 10:15 administracao
drwxr-xr-x  2 root marketing      4096 Feb  6 10:15 marketing
drwxr-xr-x  2 root ti             4096 Feb  6 10:15 ti
```

**O que mudou:**
- Coluna 3 = proprietário (root)
- Coluna 4 = **grupo** (antes era root, agora é administracao/marketing/ti)

---

### 5️⃣ Definir Permissões (chmod 770)

**Este é o passo CRUCIAL de segurança:**

```bash
# Ir para /var
cd /var

# Definir permissões 770 em todos os diretórios
sudo chmod 770 administracao
sudo chmod 770 atendimento
sudo chmod 770 compras
sudo chmod 770 contabilidade
sudo chmod 770 ecommerce
sudo chmod 770 financeiro
sudo chmod 770 logistica
sudo chmod 770 marketing
sudo chmod 770 rh
sudo chmod 770 ti
sudo chmod 770 vendas

# Verificar permissões
ls -la /var | grep -E "administracao|marketing|ti"
```

**Resultado:**
```
drwxrwx---  2 root administracao  4096 Feb  6 10:15 administracao
drwxrwx---  2 root marketing      4096 Feb  6 10:15 marketing
drwxrwx---  2 root ti             4096 Feb  6 10:15 ti
```

**Entendendo chmod 770:**

```
7 7 0
│ │ └─ Outros (other): 0 = --- (sem acesso)
│ └─── Grupo (group): 7 = rwx (leitura + escrita + execução)
└───── Proprietário (owner): 7 = rwx (leitura + escrita + execução)

Tabela de valores:
r (read/leitura) = 4
w (write/escrita) = 2
x (execute) = 1

7 = 4+2+1 = rwx (acesso total)
0 = --- (sem acesso)
```

**Por quê 770 e não 777?**
- `777` = QUALQUER PESSOA consegue acessar (INSEGURO! ❌)
- `770` = Só o grupo autorizado acessa (SEGURO! ✅)
- `0` no final = "outros" NÃO podem acessar

**Segurança:**
- ✅ Murilo (grupo TI) consegue acessar `/var/ti`
- ❌ Murilo (grupo TI) NÃO consegue acessar `/var/marketing`
- ✅ Root consegue acessar tudo

---

### 6️⃣ Criar Usuários e Atribuir Grupos

**Exemplo: criar usuário Murilo no grupo TI**

```bash
# Criar usuário (-m = criar diretório home)
sudo useradd -m murilo

# Atribuir ao grupo TI (-g = grupo primário)
sudo usermod -g ti murilo

# Definir senha Linux
sudo passwd murilo
# Digite: padrao1234
# Redigite: padrao1234

# Verificar grupo do usuário
id murilo
# Output: uid=1001(murilo) gid=1002(ti) groups=1002(ti)
```

**Criando múltiplos usuários (exemplo):**

```bash
# Criar 3 usuários para TI
for user in tech1 tech2 tech3; do
  sudo useradd -m $user
  sudo usermod -g ti $user
  echo "$user:senha123" | sudo chpasswd
done
```

---

### 7️⃣ Configurar SAMBA (smb.conf)

**Editar arquivo de configuração:**

```bash
# Abrir arquivo com nano
sudo nano /etc/samba/smb.conf
```

**Ir para seção [Share Definitions] e adicionar:**

```ini
# ==================== COMPARTILHAMENTO TI ====================
[ti]
   path = /var/ti
   valid users = @ti
   write list = @ti
   browseable = yes
   read only = no
   create mask = 0770
   directory mask = 0770

# ==================== COMPARTILHAMENTO MARKETING ====================
[marketing]
   path = /var/marketing
   valid users = @marketing
   write list = @marketing
   browseable = no
   read only = no
   create mask = 0770
   directory mask = 0770

# ==================== COMPARTILHAMENTO ADMINISTRAÇÃO ====================
[administracao]
   path = /var/administracao
   valid users = @administracao
   write list = @administracao
   browseable = yes
   read only = no
   create mask = 0770
   directory mask = 0770

# ==================== COMPARTILHAMENTO FINANCEIRO ====================
[financeiro]
   path = /var/financeiro
   valid users = @financeiro
   write list = @financeiro
   browseable = no
   read only = no
   create mask = 0770
   directory mask = 0770
```

**Explicação dos parâmetros:**

| Parâmetro | Função | Exemplo |
|-----------|--------|---------|
| `path` | Caminho do diretório | `/var/ti` |
| `valid users` | Quem pode conectar | `@ti` (grupo ti) |
| `write list` | Quem pode escrever | `@ti` (permissão de gravação) |
| `browseable` | Visível na listagem? | `yes` (aparece) / `no` (escondido) |
| `read only` | Permitir escrita? | `no` (permite escrita) |
| `create mask` | Permissão de novos arquivos | `0770` (rwxrwx---) |
| `directory mask` | Permissão de novos diretórios | `0770` (rwxrwx---) |

**Salvar e sair:**
- Pressione `Ctrl + X`
- Digite `Y` (yes)
- Pressione `Enter`

---

### 8️⃣ Criar Senha SAMBA

**IMPORTANTE: Diferente da senha Linux!**

```bash
# Criar senha SAMBA para Murilo
sudo smbpasswd -a murilo

# Digite a senha: Senac@123
# Redigite: Senac@123

# Ativar usuário SAMBA
sudo smbpasswd -e murilo
```

**Diferença de senhas:**

| Tipo | Comando | Uso | Exemplo |
|------|---------|-----|---------|
| **Linux** | `passwd murilo` | SSH, terminal local | `padrao1234` |
| **SAMBA** | `smbpasswd -a murilo` | Acesso Windows/SMB | `Senac@123` |

Podem ser diferentes! (e é recomendado por segurança)

---

### 9️⃣ Testar Configuração e Reiniciar

**Validar syntax do smb.conf:**

```bash
# Verificar se configuração está OK
testparm

# Deve retornar "Loaded services file OK"
```

**Reiniciar SAMBA:**

```bash
sudo systemctl restart smbd

# Verificar se está rodando
sudo systemctl status smbd
```

---

### 🔟 Testar do Windows

**No Windows 10/11:**

1. Abra **Explorador de Arquivos**
2. Na barra de endereços, digite:
   ```
   \\192.168.X.X
   ```
   *(substitua X.X pelo IP do servidor Linux)*

3. Digite credenciais:
   ```
   Usuário: murilo
   Senha: Senac@123
   ```

4. Você verá os compartilhamentos:
   - ✅ `ti` (visível - browseable yes)
   - ❌ `marketing` (oculto - browseable no, mas ainda funciona se sabe o nome)

**Teste de acesso:**
- Tente criar uma pasta em `\\IP\ti` (deve funcionar)
- Tente criar uma pasta em `\\IP\marketing` (deve dar permissão negada)

---

## 🔍 Troubleshooting (Resolvendo Problemas)

### ❌ "Permissão negada ao conectar"

**Solução:**
```bash
# 1. Verificar se usuário existe
id murilo

# 2. Verificar se está no grupo correto
groups murilo

# 3. Verificar permissões da pasta
ls -la /var/ti

# 4. Testar acesso local
sudo -u murilo touch /var/ti/teste.txt
```

### ❌ "Senha inválida"

**Solução:**
```bash
# Resetar senha SAMBA
sudo smbpasswd murilo
# Digite nova senha

# Verificar se usuário está ativo
sudo pdbedit -L
```

### ❌ "Servidor não encontrado"

**Solução:**
```bash
# 1. Verificar IP
hostname -I

# 2. Verificar se SAMBA está rodando
sudo systemctl status smbd

# 3. Iniciar se estiver parado
sudo systemctl start smbd

# 4. Verificar firewall
sudo ufw status
sudo ufw allow 137,138,139,445/tcp
```

### ❌ "Pasta aparece mas não consigo acessar"

**Solução:**
```bash
# 1. Verificar proprietário
ls -la /var/ti

# 2. Se o grupo está errado
sudo chgrp ti /var/ti

# 3. Se permissão está errada
sudo chmod 770 /var/ti

# 4. Recarregar SAMBA
sudo systemctl reload smbd
```

---

## 📊 Matriz de Acesso

Essa é a segurança implementada:

| Usuário | Grupo | `/var/ti` | `/var/marketing` | `/var/financeiro` | `/var/admin` |
|---------|-------|-----------|------------------|-------------------|--------------|
| murilo | ti | ✅ RW | ❌ Bloqueado | ❌ Bloqueado | ❌ Bloqueado |
| carlos | marketing | ❌ Bloqueado | ✅ RW | ❌ Bloqueado | ❌ Bloqueado |
| patricia | financeiro | ❌ Bloqueado | ❌ Bloqueado | ✅ RW | ❌ Bloqueado |
| root | admin | ✅ RW | ✅ RW | ✅ RW | ✅ RW |

---

## 🎓 Conceitos Aprendidos

### Linux Fundamentals
- ✅ Estrutura de diretórios (FHS - Filesystem Hierarchy Standard)
- ✅ Gerenciamento de usuários (`useradd`, `usermod`)
- ✅ Gerenciamento de grupos (`groupadd`, `chgrp`)
- ✅ Permissões de arquivo (`chmod`, `chown`)
- ✅ Serviços do sistema (`systemctl`)

### Networking & Sharing
- ✅ Protocolo SMB/CIFS (comunicação Windows-Linux)
- ✅ Autenticação por usuário/grupo
- ✅ Compartilhamentos granulares
- ✅ Controle de acesso baseado em papéis (RBAC)

### DevOps & Best Practices
- ✅ Separação de dados (`/var`)
- ✅ Princípio do menor privilégio (permissões 770)
- ✅ Segurança por segregação
- ✅ Documentação técnica
- ✅ Scripting para automação

---

## 📚 Próximos Passos

Para expandir este projeto:

- [ ] Implementar cotas de disco por grupo
- [ ] Adicionar backup automático dos compartilhamentos
- [ ] Configurar logs de auditoria (quem acessou o quê)
- [ ] Implementar VPN para acesso remoto seguro
- [ ] Adicionar Active Directory (LDAP) para autenticação centralizada
- [ ] Docker: containerizar SAMBA para portabilidade
- [ ] Monitoramento: Prometheus + Grafana para métricas

---

## 📖 Referências & Recursos

### Documentação Oficial
- [SAMBA Official Docs](https://www.samba.org/samba/docs/)
- [Linux man pages](https://man7.org/)
- [FHS - Filesystem Hierarchy Standard](https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard)

### Cursos & Estudos
- SENAC - Administração de Sistemas
- Linux Academy - SAMBA Configuration
- YouTube: "SAMBA Server Tutorial"

### Ferramentas Úteis
- `testparm` - Validar configuração SAMBA
- `pdbedit -L` - Listar usuários SAMBA
- `smbclient -L \\IP` - Listar compartilhamentos
- `mount -t cifs` - Montar compartilhamento em Linux

---

## 🤝 Contribuições

Se você encontrou erros, tem sugestões ou quer melhorar este projeto:

1. Faça um fork
2. Crie uma branch com sua feature (`git checkout -b feature/melhoria`)
3. Commit suas mudanças (`git commit -m 'Descrição da melhoria'`)
4. Push para a branch (`git push origin feature/melhoria`)
5. Abra um Pull Request

---

## 💬 Dúvidas? Feedback?

Deixe uma issue no repositório ou me contacte:

- **LinkedIn:** [Vitor Fernandes](https://www.linkedin.com/in/seu-perfil)
- **GitHub:** [@vifernandestech](https://github.com/vifernandestech)
- **Email:** seu.email@exemplo.com

---

## 📝 Licença

Este projeto é open source e está sob licença MIT. Você é livre para usar, modificar e distribuir.

---

## 🙏 Agradecimentos

- **SENAC** - Pela excelente formação técnica
- **Comunidade Linux** - Por todo conhecimento compartilhado
- **Você** - Por estar estudando e crescendo em TI! 💪

---

**Criado com ❤️ por Vitor Fernandes da Silva**  
*Junior IT Technician | Linux Enthusiast | Open Source Contributor*

**Última atualização:** Fevereiro, 2026
```
