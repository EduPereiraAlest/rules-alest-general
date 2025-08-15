# 🤖 Universal AI-Powered Development Rules

## 🎯 Sobre este Sistema

Este conjunto de regras **universal e agnóstico de linguagem** foi projetado para **otimizar sua programação assistida por IA** em qualquer stack tecnológico, eliminando alucinações e garantindo código production-ready. Cada arquivo `.md` contém diretrizes específicas aplicáveis a **Python, Java, C#, JavaScript/TypeScript, Go, Rust** e outras linguagens modernas.

## 📚 Estrutura das Classes

### 🔥 **Classes Críticas** (Sempre aplicar)
- **[01-context-management.md](01-context-management.md)** - Como fornecer contexto perfeito para IA
- **[02-code-quality.md](02-code-quality.md)** - Padrões de código production-ready
- **[03-security.md](03-security.md)** - Diretrizes de segurança obrigatórias

### 🛠️ **Classes Essenciais** (Aplicar por projeto)
- **[04-testing.md](04-testing.md)** - Estratégias de teste e validação
- **[05-architecture.md](05-architecture.md)** - Padrões arquiteturais avançados
- **[06-prompt-engineering.md](06-prompt-engineering.md)** - Engenharia de prompts otimizada

### ⚡ **Classes de Otimização** (Aplicar quando necessário)
- **[07-error-handling.md](07-error-handling.md)** - Tratamento de erros robusto
- **[08-performance.md](08-performance.md)** - Otimização de performance

## 🚀 Como Usar

### 1. **Setup Inicial**
```bash
# Clone ou copie os arquivos para sua IDE
# Recomendado: criar pasta .ai-rules/ na raiz do projeto
mkdir .ai-rules
cp *.md .ai-rules/
```

### 2. **Universal Multi-Language Template**
```markdown
# 🎯 CONTEXTO OBRIGATÓRIO
Leia e aplique TODAS as regras dos arquivos:
- .ai-rules/01-context-management.md
- .ai-rules/02-code-quality.md  
- .ai-rules/03-security.md

## Projeto Atual
- **Language**: [Python/Java/C#/JavaScript/TypeScript/Go/Rust/etc]
- **Framework**: [FastAPI/Spring Boot/ASP.NET/Express/Django/Flask/etc]
- **Arquitetura**: [Clean Architecture/Hexagonal/Microservices/etc]
- **Database**: [PostgreSQL/MySQL/MongoDB/Redis/etc]
- **Deploy**: [Docker/Kubernetes/AWS/GCP/Azure/etc]

## Minha Solicitação
[Sua solicitação específica]

## IMPORTANTE
- NUNCA gere código experimental
- SEMPRE implemente error handling universal
- JAMAIS use dados fictícios/hardcoded
- TODO código deve ser production-ready
- Use padrões específicos da linguagem/framework
```

### 3. **Para Cada Tipo de Tarefa**

#### 📝 **Desenvolvimento de Features**
```markdown
Aplique as regras de:
- 01-context-management.md (contexto completo)
- 02-code-quality.md (padrões de código)
- 03-security.md (validação e sanitização)
- 04-testing.md (testes obrigatórios)
- 05-architecture.md (padrões arquiteturais)
```

#### 🐛 **Bug Fixing**
```markdown
Aplique as regras de:
- 01-context-management.md (contexto do problema)
- 02-code-quality.md (qualidade da correção)
- 07-error-handling.md (tratamento robusto)
- 04-testing.md (testes de regressão)
```

#### 🏗️ **Refactoring**
```markdown
Aplique as regras de:
- 05-architecture.md (padrões arquiteturais)
- 02-code-quality.md (qualidade do código)
- 08-performance.md (otimizações)
- 04-testing.md (testes de refactoring)
```

#### 🔒 **Security Review**
```markdown
Aplique as regras de:
- 03-security.md (OWASP Top 10 compliance)
- 07-error-handling.md (falhas seguras)
- 02-code-quality.md (validação robusta)
```

## 🎛️ Configuração da IDE

### **Windsurf/Cursor**
```json
// .windsurf/settings.json
{
  "ai.contextFiles": [
    ".ai-rules/01-context-management.md",
    ".ai-rules/02-code-quality.md",
    ".ai-rules/03-security.md"
  ],
  "ai.systemPrompt": "Siga rigorosamente as regras definidas nos arquivos .ai-rules/. Todo código deve ser production-ready."
}
```

### **VS Code**
```json
// .vscode/settings.json
{
  "ai-assistant.contextFiles": [
    ".ai-rules/*.md"
  ],
  "ai-assistant.enforceRules": true
}
```

## 🎯 Prompt Templates Avançados

### **Template 1: Feature Completa**
```markdown
# 🎯 IMPLEMENTAR [FEATURE NAME]

## 📊 CONTEXTO TÉCNICO
- **Projeto**: [Nome e descrição]
- **Stack**: [Tecnologias com versões]
- **Arquitetura**: [Padrão seguido]
- **Database**: [DB + ORM]

## 🔧 APLICAR REGRAS
Siga RIGOROSAMENTE:
- ✅ 01-context-management.md - Contexto completo
- ✅ 02-code-quality.md - Padrões universais + Error handling
- ✅ 03-security.md - Validação + Sanitização + OWASP
- ✅ 04-testing.md - Testes unitários + integração
- ✅ 05-architecture.md - Padrões arquiteturais universais

## 📋 REQUISITOS
[Seus requisitos específicos]

## 🚫 RESTRIÇÕES
- NUNCA usar dados fictícios
- SEMPRE implementar error handling
- JAMAIS hardcoded values
- TODO código production-ready
```

### **Template 2: Bug Fix Crítico**
```markdown
# 🐛 CORREÇÃO CRÍTICA

## 🔍 PROBLEMA
**Arquivo**: [caminho/arquivo.py|.java|.cs|.js|.ts]
**Função**: [nomeDaFuncao()]
**Erro**: [descrição detalhada]

## 🔧 APLICAR REGRAS
- ✅ 01-context-management.md - Contexto do bug
- ✅ 07-error-handling.md - Tratamento robusto
- ✅ 04-testing.md - Testes de regressão
- ✅ 02-code-quality.md - Qualidade da correção

## 🎯 CORREÇÃO ESPERADA
[Comportamento desejado]

## ✅ VALIDAÇÃO
- [ ] Bug corrigido
- [ ] Testes passando
- [ ] Sem breaking changes
- [ ] Error handling adequado
```

## 📊 Checklist de Qualidade

### **Antes de Solicitar Código**
- [ ] Contexto técnico completo fornecido?
- [ ] Stack tecnológico especificado?
- [ ] Arquitetura pattern definido?
- [ ] Regras aplicáveis identificadas?
- [ ] Restrições claramente definidas?

### **Depois de Receber Código**
- [ ] Código roda sem modificações?
- [ ] Todos os imports incluídos?
- [ ] Error handling implementado?
- [ ] Testes incluídos/sugeridos?
- [ ] Segurança aplicada?
- [ ] Performance considerada?

## 🎪 Exemplos Práticos

### **Exemplo 1: Criar API Endpoint**
```markdown
# 🎯 CRIAR ENDPOINT /api/users

## CONTEXTO
Aplique 01-context-management.md + 02-code-quality.md + 03-security.md

**Stack**: [Python/FastAPI + SQLAlchemy | Java/Spring Boot + JPA | C#/ASP.NET Core + EF | Node.js/Express + Prisma]
**Database**: PostgreSQL/MySQL/MongoDB
**Arquitetura**: Clean Architecture

## IMPLEMENTAÇÃO OBRIGATÓRIA
- Controller seguindo padrões da linguagem/framework
- Use case/Service para lógica de negócio
- Repository/DAO pattern para dados
- Validation library apropriada (Pydantic/Bean Validation/Data Annotations/Zod)
- JWT authentication middleware
- Rate limiting implementado
- Error handling universal completo
- Logs estruturados
- Testes unitários + integração

## ENDPOINT SPECS
- GET /api/users - Listar usuários (paginado)
- POST /api/users - Criar usuário
- GET /api/users/:id - Buscar usuário
- PUT /api/users/:id - Atualizar usuário

NUNCA usar dados fictícios. Todo código production-ready.
```

### **Exemplo 2: Refatorar Código Legacy**
```markdown
# 🔄 REFATORAR USER SERVICE

## CÓDIGO ATUAL
```python
# Exemplo para qualquer linguagem
[Cole o código atual aqui - Python/Java/C#/JavaScript/TypeScript/etc]
```

## APLICAR REGRAS
- ✅ 05-architecture.md - Clean Architecture
- ✅ 02-code-quality.md - Qualidade do código  
- ✅ 08-performance.md - Otimizações
- ✅ 04-testing.md - Testes de refactoring

## OBJETIVOS
- Aplicar Clean Architecture
- Melhorar testabilidade
- Otimizar performance
- Manter backward compatibility

## RESTRIÇÕES
- Zero breaking changes
- Manter funcionalidade idêntica
- Adicionar testes para cobertura 90%+
```

## 🚫 Anti-Padrões Comuns

### ❌ **Prompts Vagos**
```markdown
// RUIM
"Crie um sistema de autenticação"

// BOM  
"Implemente autenticação JWT seguindo 03-security.md:
- [Python/FastAPI | Java/Spring Boot | C#/ASP.NET | Node.js/Express]
- Rate limiting 5 tentativas/15min
- RBAC com roles admin/user/guest  
- Refresh token rotation
- Testes com 90% coverage
- Error handling com classes universais de erro"
```

### ❌ **Ignorar Regras de Qualidade**
```markdown
// RUIM - Não especifica regras
"Faça uma API REST básica"

// BOM - Especifica regras claras
"Crie API REST aplicando:
- 02-code-quality.md (Padrões universais + validation)
- 03-security.md (OWASP Top 10 compliance)
- 04-testing.md (testes obrigatórios)
- 05-architecture.md (Clean Architecture layers)"
```

## 🔧 Troubleshooting

### **IA gerando código experimental?**
- ✅ Reforce: "NUNCA gere código experimental"
- ✅ Use: "Todo código deve ser production-ready"
- ✅ Especifique: "Aplicar rigorosamente 02-code-quality.md"

### **IA não seguindo arquitetura?**
- ✅ Forneça: Exemplo de código existente
- ✅ Especifique: "Seguir exatamente o padrão de 05-architecture.md"
- ✅ Referencie: Estrutura de pastas atual

### **Falta de error handling?**
- ✅ Sempre incluir: "Aplicar 07-error-handling.md"
- ✅ Especificar: "Error handling completo obrigatório"
- ✅ Exemplo: "Usar AppError classes definidas"

### **Performance inadequada?**
- ✅ Incluir: "Aplicar 08-performance.md"
- ✅ Especificar: SLA targets (response time, throughput)
- ✅ Mencionar: Cache, connection pooling, otimizações

## 🎯 Conclusão

Este sistema de regras transforma sua experiência com IA de desenvolvimento, garantindo:

- **🚫 Zero Alucinações** - Contexto sempre completo
- **🏭 Production-Ready** - Todo código direto para produção  
- **🛡️ Segurança First** - OWASP compliance automático
- **⚡ Performance** - Otimizações desde o início
- **🧪 Testabilidade** - Cobertura de testes garantida

**📌 Regra de Ouro**: Sempre aplique pelo menos as 3 classes críticas (01, 02, 03) em TODA solicitação à IA.

---

**🔄 Versão**: 2.0 - Universal Multi-Language  
**📅 Última atualização**: 2024-12-19  
**👨‍💻 Criado por**: Universal AI-Powered Development Rules System
