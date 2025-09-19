# 🚀 Guia de Início Rápido - Sistema Multi-Tenant

## 1. Configuração Inicial

### 1.1 Configurar o projeto
```bash
# Configure o AUTH_USER_MODEL no settings.py
AUTH_USER_MODEL = 'accounts.CustomUser'

# Execute as migrações
python manage.py makemigrations accounts
python manage.py migrate
```

### 1.2 Criar a empresa master e admin principal
```bash
python manage.py create_master_company \
    --name "SyncWave" \
    --slug "syncwave" \
    --email "admin@syncwave.com" \
    --admin-username "admin" \
    --admin-email "admin@syncwave.com" \
    --admin-password "sua_senha_segura_aqui"
```

### 1.3 Iniciar servidor e testar
```bash
python manage.py runserver
```

Acesse `http://127.0.0.1:8000/admin/` com:
- **Username**: admin
- **Password**: sua_senha_segura_aqui

## 2. Criando Empresas Clientes (Via Admin)

### 2.1 Criar nova empresa
1. Login no admin como usuário master
2. Vá em **"Empresas"** → **"Adicionar empresa"**
3. Preencha:
   - Nome: "Empresa A"
   - Slug: "empresa-a" (gerado automaticamente)
   - Email: "contato@empresaa.com"
   - CNPJ: (opcional)
   - Tipo: "Client Company"
   - Ativo: ✅
4. **Salvar**

### 2.2 Criar admin da empresa cliente
1. Vá em **"Usuários"** → **"Adicionar usuário"**
2. Preencha:
   - Username: "admin_empresaa"
   - Password: (senha segura)
   - **Empresa**: Selecione "Empresa A"
   - **Cargo/Função**: "Administrador"
   - Email: "admin@empresaa.com"
   - Nome: "Admin"
   - Sobrenome: "Empresa A"
3. Em **"Permissões"**:
   - ✅ **Ativo**
   - ✅ **Acesso à administração** (is_staff)
   - ❌ **Status de superusuário** (deixe desmarcado)
4. **Salvar**

## 3. Testando o Sistema

### 3.1 Login como usuário da empresa cliente
- Faça logout do admin master
- Faça login com: `admin_empresaa` / `senha_da_empresa`
- Observe que ele só vê dados da "Empresa A"

### 3.2 Login como usuário master
- Faça login com o usuário `admin` original
- Observe que ele vê dados de **todas** as empresas

## 4. Próximos Passos

### 4.1 Criar outros usuários
Para cada empresa, crie usuários com diferentes roles:
- **Admin**: Acesso total à empresa
- **Manager**: Gerenciamento operacional
- **Employee**: Funcionário padrão
- **Viewer**: Apenas visualização

### 4.2 Desenvolver seus módulos
Todos os seus próximos models devem herdar de `BaseCompanyModel`:

```python
from accounts.utils import BaseCompanyModel

class SeuModel(BaseCompanyModel):
    nome = models.CharField(max_length=100)
    # outros campos...
    
    class Meta:
        verbose_name = "Seu Model"
```

### 4.3 Proteger suas views
Use os decorators fornecidos:

```python
from accounts.decorators import company_required, master_company_required

@login_required
@company_required
def minha_view(request):
    # Usuário está autenticado e vinculado a empresa ativa
    pass

@login_required  
@master_company_required
def view_apenas_master(request):
    # Apenas usuários da SyncWave podem acessar
    pass
```

## 5. Comandos Úteis

```bash
# Listar todas as empresas
python manage.py list_companies

# Criar superuser adicional (se necessário)
python manage.py createsuperuser

# Shell para testes
python manage.py shell
```

## 6. Estrutura de Acesso

```
SyncWave (Master)
├── Admin Master → Acesso TOTAL
├── Users SyncWave → Acesso a todas empresas
│
Empresa A (Cliente)  
├── Admin Empresa A → Acesso apenas à Empresa A
├── Users Empresa A → Acesso apenas à Empresa A
│
Empresa B (Cliente)
├── Admin Empresa B → Acesso apenas à Empresa B  
└── Users Empresa B → Acesso apenas à Empresa B
```

## ✅ Checklist de Implementação

- [ ] Configurar AUTH_USER_MODEL
- [ ] Executar migrações
- [ ] Criar empresa master via comando
- [ ] Testar login do admin master
- [ ] Criar primeira empresa cliente via admin
- [ ] Criar admin da empresa cliente via admin
- [ ] Testar isolamento de dados
- [ ] Implementar seus próprios models
- [ ] Proteger views com decorators

**Agora você tem um sistema multi-tenant completo e funcional! 🎉**