# 🎯 Guia Completo - Sistema Multi-Tenant SEM Subdomínios

## 📋 Visão Geral

Este guia mostra como implementar um sistema multi-tenant completo usando **um único domínio**, onde:
- **SyncWave** (empresa master) tem acesso a todas as empresas
- **Empresas clientes** veem apenas seus próprios dados
- **Admin único** em `/admin/` para todas as empresas
- **Segurança automática** com isolamento de dados

---

## 🚀 1. CONFIGURAÇÃO INICIAL

### 1.1 Estrutura de Arquivos
Certifique-se de ter estes arquivos criados:

```
accounts/
├── __init__.py
├── models.py          ✅ (com Company, CustomUser, CompanyPermission)
├── admin.py           ✅ (admin customizado)  
├── decorators.py      ✅ (proteção de views)
├── utils.py           ✅ (BaseCompanyModel)
├── context_processors.py ✅ (variáveis globais)
├── management/
│   └── commands/
│       ├── __init__.py
│       ├── create_master_company.py ✅
│       └── list_companies.py ✅
```

### 1.2 Settings.py - ÚNICA alteração necessária
```python
# settings.py
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [BASE_DIR / 'templates'],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
                'accounts.context_processors.company_context',  # 👈 ADICIONE APENAS ESTA LINHA
            ],
        },
    },
]

# Certifique-se de ter:
AUTH_USER_MODEL = 'accounts.CustomUser'
INSTALLED_APPS = [
    # ... outras apps ...
    'accounts',
]
```

### 1.3 Migrações
```bash
python manage.py makemigrations accounts
python manage.py migrate
```

### 1.4 Criar Empresa Master
```bash
python manage.py create_master_company \
    --name "SyncWave" \
    --slug "syncwave" \
    --email "admin@syncwave.com" \
    --admin-username "admin" \
    --admin-email "admin@syncwave.com" \
    --admin-password "sua_senha_segura"
```

---

## 🎯 2. USANDO OS DECORATORS (Proteção Automática)

### 2.1 Tipos de Decorators Disponíveis

#### `@company_required` - Usuário DEVE ter empresa
```python
from django.contrib.auth.decorators import login_required
from accounts.decorators import company_required

@login_required
@company_required
def dashboard(request):
    # ✅ request.user.company sempre existirá
    # ✅ Bloqueia usuários sem empresa
    empresa = request.user.company
    usuarios = empresa.users.all()
    return render(request, 'dashboard.html', {'usuarios': usuarios})
```

#### `@master_company_required` - Só SyncWave pode acessar
```python
from accounts.decorators import master_company_required

@login_required
@master_company_required
def relatorio_geral(request):
    # ✅ Só usuários da SyncWave podem acessar
    todas_empresas = Company.objects.all()
    return render(request, 'admin/relatorio.html', {'empresas': todas_empresas})
```

#### `@company_access_required('slug')` - Empresa específica
```python
from accounts.decorators import company_access_required

@login_required
@company_access_required('empresa-a')
def dados_empresa_a(request):
    # ✅ Só quem pode acessar "empresa-a"
    # ✅ request.target_company disponível
    empresa = request.target_company
    return render(request, 'empresa_dados.html', {'empresa': empresa})
```

### 2.2 Exemplo Prático - Views Protegidas
```python
# views.py
from django.shortcuts import render
from django.contrib.auth.decorators import login_required
from accounts.decorators import company_required, master_company_required
from .models import Product, Sale

@login_required
@company_required
def produtos_list(request):
    """Lista produtos - cada empresa vê apenas os seus"""
    produtos = Product.objects.for_user(request.user)
    return render(request, 'produtos/list.html', {
        'produtos': produtos,
        'total': produtos.count()
    })

@login_required
@company_required
def produto_create(request):
    """Criar produto - sempre vinculado à empresa do usuário"""
    if request.method == 'POST':
        produto = Product.objects.create(
            name=request.POST['name'],
            price=request.POST['price'],
            company=request.user.company  # ✅ Sempre da empresa do usuário
        )
        return redirect('produtos_list')
    return render(request, 'produtos/create.html')

@login_required
@master_company_required
def admin_dashboard(request):
    """Dashboard master - só SyncWave pode ver"""
    stats = {
        'total_empresas': Company.objects.count(),
        'total_usuarios': CustomUser.objects.count(),
        'total_produtos': Product.objects.count(),
    }
    return render(request, 'admin/dashboard.html', {'stats': stats})
```

---

## 🔧 3. USANDO UTILS (Modelos Inteligentes)

### 3.1 BaseCompanyModel - Para seus modelos
```python
# products/models.py
from accounts.utils import BaseCompanyModel

class Product(BaseCompanyModel):
    """Produto vinculado automaticamente à empresa"""
    name = models.CharField(max_length=200)
    description = models.TextField(blank=True)
    price = models.DecimalField(max_digits=10, decimal_places=2)
    is_active = models.BooleanField(default=True)
    
    # ✅ company, created_at, updated_at já incluídos via BaseCompanyModel
    # ✅ Manager personalizado já incluído
    
    class Meta:
        verbose_name = "Produto"
        unique_together = ['company', 'name']  # Nome único por empresa
    
    def __str__(self):
        return f"{self.name} - {self.company.name}"

class Sale(BaseCompanyModel):
    """Venda vinculada à empresa"""
    product = models.ForeignKey(Product, on_delete=models.CASCADE)
    quantity = models.PositiveIntegerField()
    unit_price = models.DecimalField(max_digits=10, decimal_places=2)
    total = models.DecimalField(max_digits=10, decimal_places=2)
    
    def save(self, *args, **kwargs):
        # ✅ Garante que produto e venda são da mesma empresa
        if self.product.company != self.company:
            raise ValueError("Produto deve ser da mesma empresa")
        self.total = self.quantity * self.unit_price
        super().save(*args, **kwargs)
```

### 3.2 Queries Automáticas
```python
# Em suas views - filtragem automática
def produtos_view(request):
    # ✅ Filtra automaticamente por empresa
    produtos = Product.objects.for_user(request.user)
    # Se user é master: TODOS os produtos
    # Se user é cliente: SÓ produtos da empresa dele
    
    # ✅ Só empresas ativas
    produtos_ativos = Product.objects.for_user(request.user).filter(is_active=True)
    
    # ✅ Combinações
    vendas_recentes = Sale.objects.for_user(request.user).filter(
        created_at__gte=timezone.now() - timedelta(days=30)
    )
```

---

## 🎨 4. TEMPLATES INTELIGENTES (Context Processors)

### 4.1 Variáveis Automáticas Disponíveis
Em **TODOS** os seus templates, essas variáveis estão disponíveis automaticamente:

```html
<!-- base.html -->
<!DOCTYPE html>
<html>
<head>
    <title>{{ user_company.name }} - Sistema</title>
</head>
<body>
    <header>
        <h1>{{ user_company.name }}</h1>
        <p>Bem-vindo, {{ user.get_full_name }}!</p>
        
        {% if is_master_user %}
            <div class="alert alert-warning">
                <strong>🏢 MODO MASTER</strong> - Você tem acesso a todas as empresas
            </div>
        {% endif %}
    </header>
    
    <nav>
        <a href="/dashboard/">Dashboard</a>
        <a href="/produtos/">Produtos</a>
        
        {% if is_master_user %}
            <a href="/admin-dashboard/">Admin Master</a>
            
            <!-- Seletor de empresa para masters -->
            <select onchange="switchCompany(this.value)">
                <option value="">Ver como empresa:</option>
                {% for empresa in accessible_companies %}
                    <option value="{{ empresa.slug }}">{{ empresa.name }}</option>
                {% endfor %}
            </select>
        {% endif %}
    </nav>
    
    <main>
        {% block content %}{% endblock %}
    </main>
</body>
</html>
```

### 4.2 Template de Lista
```html
<!-- produtos/list.html -->
{% extends 'base.html' %}

{% block content %}
<div class="container">
    <h2>Produtos {% if is_master_user %}(Todas as Empresas){% else %}({{ user_company.name }}){% endif %}</h2>
    
    <div class="row">
        {% for produto in produtos %}
            <div class="col-md-4">
                <div class="card">
                    <div class="card-body">
                        <h5>{{ produto.name }}</h5>
                        <p>R$ {{ produto.price }}</p>
                        
                        {% if is_master_user %}
                            <small class="text-muted">
                                Empresa: {{ produto.company.name }}
                            </small>
                        {% endif %}
                    </div>
                </div>
            </div>
        {% endfor %}
    </div>
</div>
{% endblock %}
```

---

## 🔐 5. ADMIN INTELIGENTE (Já Configurado)

### 5.1 Funcionalidades Automáticas
- ✅ **Filtros automáticos**: Usuários veem apenas dados da própria empresa
- ✅ **Formulários inteligentes**: Campos se adaptam ao usuário logado
- ✅ **Validações**: Impede criação de dados inconsistentes
- ✅ **Interface visual**: Badges, cores, ícones

### 5.2 Como Usar
```bash
# 1. Acesse o admin
http://localhost:8000/admin/

# 2. Login como admin master
Username: admin
Password: sua_senha_segura

# 3. Crie empresas clientes:
- Vá em "Empresas" → "Adicionar empresa"
- Preencha: Nome, Email, CNPJ (opcional)
- Tipo: "Client Company"

# 4. Crie usuários para empresas:
- Vá em "Usuários" → "Adicionar usuário"  
- Preencha dados + selecione empresa
- Marque "Acesso à administração" se necessário
```

---

## 🎯 6. FLUXO COMPLETO DO SISTEMA

### 6.1 Para Usuário Master (SyncWave)
```
1. Login → admin@syncwave.com
2. Acesso → TODOS os dados de TODAS as empresas
3. Admin → Pode criar empresas e usuários
4. Views → Vê dados agregados
5. Templates → Mostram "MODO MASTER"
```

### 6.2 Para Usuário Cliente (Empresa A)
```
1. Login → admin@empresaa.com  
2. Acesso → APENAS dados da Empresa A
3. Admin → Só usuários da Empresa A
4. Views → Dados filtrados automaticamente
5. Templates → Mostram "Empresa A"
```

### 6.3 Exemplo de Sessão
```python
# Usuario master logado:
request.user.company.name           # "SyncWave"
request.user.is_master_user()       # True
request.user.can_access_all_companies # True
Product.objects.for_user(request.user) # TODOS os produtos

# Usuario cliente logado:
request.user.company.name           # "Empresa A"
request.user.is_master_user()       # False  
request.user.can_access_all_companies # False
Product.objects.for_user(request.user) # SÓ produtos da Empresa A
```

---

## 🚀 7. DESENVOLVIMENTO DE NOVOS MÓDULOS

### 7.1 Criando Novo Model
```python
# sales/models.py
from accounts.utils import BaseCompanyModel

class Sale(BaseCompanyModel):
    customer_name = models.CharField(max_length=200)
    total = models.DecimalField(max_digits=10, decimal_places=2)
    
    # ✅ company, created_at, updated_at automáticos
    # ✅ Filtragem automática por empresa
```

### 7.2 Criando Nova View
```python
# sales/views.py  
from accounts.decorators import company_required

@login_required
@company_required
def sales_list(request):
    # ✅ Automaticamente filtrado por empresa
    sales = Sale.objects.for_user(request.user)
    return render(request, 'sales/list.html', {'sales': sales})
```

### 7.3 Criando Novo Admin
```python
# sales/admin.py
from django.contrib import admin
from accounts.utils import CompanyManager
from .models import Sale

@admin.register(Sale)
class SaleAdmin(admin.ModelAdmin):
    list_display = ['customer_name', 'company', 'total', 'created_at']
    list_filter = ['company', 'created_at']
    
    def get_queryset(self, request):
        # ✅ Filtro automático por empresa
        return super().get_queryset(request).for_user(request.user)
```

---

## ✅ 8. CHECKLIST FINAL

### ✅ Configuração
- [ ] Settings.py com context_processors
- [ ] Migrações executadas
- [ ] Empresa master criada
- [ ] Admin master funcionando

### ✅ Segurança  
- [ ] Views protegidas com decorators
- [ ] Models usando BaseCompanyModel
- [ ] Admin com filtros automáticos
- [ ] Templates mostrando empresa correta

### ✅ Funcionalidades
- [ ] Usuário master vê tudo
- [ ] Usuário cliente vê só sua empresa
- [ ] Criação de empresas pelo admin
- [ ] Criação de usuários pelo admin

### ✅ Testes
- [ ] Login como master funciona
- [ ] Login como cliente funciona  
- [ ] Isolamento de dados funciona
- [ ] Templates mostram dados corretos

---

## 🎉 PRONTO!

Seu sistema multi-tenant está **100% funcional**:

- 🏢 **Empresa Master**: SyncWave com acesso total
- 🏪 **Empresas Clientes**: Dados isolados automaticamente  
- 🔐 **Segurança**: Proteção automática em todas as camadas
- 🎨 **Interface**: Admin e templates inteligentes
- ⚡ **Performance**: Consultas otimizadas e filtradas
- 🚀 **Escalável**: Fácil de adicionar novos módulos

**Próximos passos**: Desenvolver seus módulos específicos (vendas, produtos, relatórios, etc.) usando as bases fornecidas!