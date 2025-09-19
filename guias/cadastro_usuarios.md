# 🎯 Admin Melhorado - Guia de Uso

## 📋 O que foi melhorado no Admin

### ✨ **Cadastro de Usuários Simplificado**
- Formulário inteligente que adapta campos baseado no usuário logado
- Validações automáticas (email duplicado, empresa obrigatória)
- Interface mais limpa e intuitiva
- Campos organizados em seções lógicas

### 🎨 **Interface Visual Aprimorada**
- Badges coloridos para roles dos usuários
- Status visual (ativo/inativo) com ícones
- Informações da empresa com ícones distintivos
- Links diretos entre empresas e usuários
- Contadores de usuários por empresa

### 🔐 **Segurança e Isolamento**
- Usuários não-master só veem dados da própria empresa
- Filtros automáticos baseados na empresa do usuário logado
- Permissões contextuais (criar/editar/deletar)
- Proteção contra criação de usuários em empresas inadequadas

## 🚀 Como Usar

### 1. **Criar uma Nova Empresa (Admin Master)**

1. Login como admin master (SyncWave)
2. Vá em **"Empresas"** → **"Adicionar empresa"**
3. Preencha os dados organizados por seções:
   - **Informações Básicas**: Nome, slug, tipo
   - **Contato**: Email, telefone, CNPJ  
   - **Endereço**: Dados de localização (opcional)

### 2. **Criar Usuário para Empresa**

#### 🎯 **Formulário Inteligente**
- Se você é admin master → Pode selecionar qualquer empresa
- Se você é admin de empresa → Automaticamente vincula à sua empresa

#### 📝 **Campos do Formulário**
```
Dados de Acesso:
├── Username (obrigatório)
├── Password1 (obrigatório)  
└── Password2 (confirmação)

Informações Pessoais:
├── Nome (obrigatório)
├── Sobrenome (obrigatório)
├── E-mail (obrigatório, único)
└── Telefone (opcional)

Vinculação com Empresa:
├── Empresa (seleção automática ou manual)
├── Cargo/Função (Admin, Manager, Employee, Viewer)
└── Departamento (opcional: Vendas, TI, etc.)

Permissões:
└── Acesso à administração (is_staff)
```

### 3. **Gerenciar Usuários Existentes**

#### 🔍 **Lista de Usuários Melhorada**
- **Empresa**: Mostra nome + tipo (Master/Cliente)
- **Cargo**: Badge colorido por role
- **Status**: Ícones visuais (✓ Ativo / ✗ Inativo)
- **Filtros**: Por empresa, cargo, status, data

#### ⚡ **Filtros Disponíveis**
- Status (Ativo/Inativo)
- Staff (Acesso admin)
- Superuser
- Tipo de empresa (Master/Cliente)
- Cargo/Função
- Acesso a todas empresas
- Data de cadastro

### 4. **Permissões entre Empresas**

Para casos especiais onde uma empresa precisa acessar dados de outra:

1. Vá em **"Permissões entre Empresas"**
2. Configure:
   - **Empresa Proprietária**: Dona dos dados
   - **Empresa com Acesso**: Quem receberá acesso
   - **Tipo de Permissão**: read, write, admin

## 🎨 Recursos Visuais

### 🏷️ **Badges de Cargo**
```
🔴 Admin     (Vermelho)
🟠 Manager   (Laranja)  
🟢 Employee  (Verde)
⚪ Viewer    (Cinza)
```

### 🏢 **Tipos de Empresa**
```
🏢 Master   (SyncWave - acesso total)
🏪 Cliente  (Empresas clientes - acesso restrito)
```

### 📊 **Contador de Usuários**
- Link direto: clique no número de usuários para ver a lista filtrada
- Cores: Verde (com usuários) / Vermelho (sem usuários)

## 🔧 Funcionalidades por Tipo de Usuário

### 👑 **Admin Master (SyncWave)**
✅ Ver todas as empresas  
✅ Criar/editar/deletar qualquer empresa  
✅ Ver todos os usuários de todas as empresas  
✅ Criar usuários em qualquer empresa  
✅ Gerenciar permissões entre empresas  
✅ Todas as funcionalidades administrativas  

### 👨‍💼 **Admin de Empresa Cliente**
✅ Ver apenas sua empresa  
✅ Editar dados da sua empresa  
✅ Ver usuários apenas da sua empresa  
✅ Criar usuários apenas na sua empresa  
✅ Gerenciar permissões da sua empresa  
❌ Não pode criar outras empresas  
❌ Não pode deletar sua empresa  

### 👤 **Usuário Comum (Employee/Viewer)**
✅ Ver seus próprios dados  
✅ Editar alguns dados pessoais  
❌ Não tem acesso ao admin (a menos que is_staff=True)  

## 📱 Dicas de Uso

### ⚡ **Atalhos Rápidos**
- **Criar usuário**: Empresas → clique no número de usuários → "Adicionar usuário"
- **Filtrar por empresa**: Use os filtros laterais
- **Busca rápida**: Digite nome, email ou empresa na caixa de busca

### 🎯 **Boas Práticas**
1. **Sempre preencha nome e sobrenome** - melhora a identificação
2. **Use emails corporativos** - facilita a organização
3. **Configure departamentos** - ajuda no agrupamento
4. **Defina roles adequados** - controla o nível de acesso
5. **Ative is_staff apenas quando necessário** - segurança

### 🚨 **Cuidados Importantes**
- **Não delete a empresa master** (SyncWave)
- **Cuidado com permissões de superuser** - use apenas quando necessário
- **Verifique a empresa antes de criar usuários** - não pode ser alterada depois facilmente
- **Mantenha emails únicos** - evita conflitos de login

## 🔄 Workflow Recomendado

```
1. Criar Empresa (admin master)
   ↓
2. Criar Admin da Empresa  
   ↓
3. Admin da empresa cria seus usuários
   ↓
4. Configurar permissões especiais (se necessário)
   ↓
5. Usuários fazem login e usam o sistema
```

Agora o admin está muito mais intuitivo e funcional! 🎉