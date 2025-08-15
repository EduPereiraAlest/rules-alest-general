# 🔍 Post-Implementation Review Rules

## 🎯 Objetivo
Checklist obrigatório para validação após implementar código, garantindo qualidade e production-readiness antes do deploy.

## ⚡ Quick Check (30 segundos)
```bash
# Execute imediatamente após escrever código
```

### **🚨 Checklist Crítico**
- [ ] **Compila/executa sem erros**
- [ ] **Imports/dependencies corretos**  
- [ ] **Sem hardcoded values**
- [ ] **Error handling presente**
- [ ] **Logs estruturados implementados**

## 🔬 Deep Analysis (5 minutos)

### **📋 Checklist de Segurança**
- [ ] Input validation implementada
- [ ] SQL injection prevenido (prepared statements)
- [ ] XSS protection ativo
- [ ] CSRF tokens validados
- [ ] Authentication/authorization verificados
- [ ] Dados sensíveis não expostos em logs
- [ ] Rate limiting implementado (APIs)

### **🏗️ Checklist de Arquitetura**
- [ ] Separation of concerns respeitada
- [ ] Single responsibility principle aplicado
- [ ] Dependency injection utilizada
- [ ] Interfaces/abstractions definidas
- [ ] Clean Architecture layers respeitadas

### **🧪 Checklist de Testes**
- [ ] Unit tests criados (mínimo 80% coverage)
- [ ] Integration tests para APIs
- [ ] Edge cases cobertos
- [ ] Error scenarios testados
- [ ] Mocks apropriados utilizados

### **⚡ Checklist de Performance**
- [ ] Database queries otimizadas
- [ ] N+1 queries evitadas
- [ ] Caching implementado onde apropriado
- [ ] Connection pooling configurado
- [ ] Lazy loading utilizado
- [ ] Memory leaks verificados

### **📝 Checklist de Code Quality**
- [ ] Nomes de variáveis/funções descritivos
- [ ] Comentários explicando "porquê", não "como"
- [ ] Dead code removido
- [ ] TODOs documentados com issues
- [ ] Magic numbers extraídos para constantes

## 🚨 Red Flags - Pare Tudo!

### **❌ Código NÃO pode ir para produção se:**
- Contém `console.log`, `print()`, `System.out.println()` em produção
- Tem senhas/tokens hardcoded
- Não trata exceções adequadamente
- Retorna stack traces para usuários finais
- Não valida input do usuário
- Usa queries SQL dinâmicas sem sanitização
- Não tem logs de auditoria para ações críticas

## 🔧 Automated Checks

### **Python**
```bash
# Security & Quality
bandit -r .
safety check
pylint src/
black --check src/
mypy src/

# Tests
pytest --cov=src --cov-report=term-missing
```

### **Java**
```bash
# Security & Quality  
mvn spotbugs:check
mvn checkstyle:check
mvn pmd:check
./gradlew sonarqube

# Tests
mvn test jacoco:report
```

### **C#**
```bash
# Security & Quality
dotnet format --verify-no-changes
dotnet build --configuration Release
SonarScanner.MSBuild.exe begin

# Tests
dotnet test --collect:"XPlat Code Coverage"
```

### **JavaScript/TypeScript**
```bash
# Security & Quality
npm audit
eslint src/
tsc --noEmit
prettier --check src/

# Tests
npm test -- --coverage
```

## 📊 Manual Code Review

### **🔍 Code Smell Detection**
```markdown
## Revisar se o código tem:
- [ ] Funções > 20 linhas (refatorar)
- [ ] Classes > 200 linhas (dividir)
- [ ] Parâmetros > 5 (usar objetos)
- [ ] Nested loops > 2 níveis (extrair métodos)
- [ ] Try-catch muito genéricos (especificar exceções)
```

### **🛠️ Refactoring Opportunities**
- [ ] Duplicação de código identificada
- [ ] Métodos complexos quebrados
- [ ] Magic numbers extraídos
- [ ] Long parameter lists simplificadas
- [ ] God classes divididas

## 🌐 Environment-Specific Checks

### **🏠 Development**
- [ ] Debug logs habilitados
- [ ] Hot reload funcionando
- [ ] Database seeds executadas
- [ ] Mock services funcionando

### **🧪 Testing/Staging**
- [ ] Environment variables configuradas
- [ ] Database migrations executadas
- [ ] External services mockados
- [ ] Load balancer configurado

### **🏭 Production**
- [ ] Debug logs DESABILITADOS
- [ ] SSL/TLS configurado
- [ ] Monitoring/alerting ativo
- [ ] Backup/restore testado
- [ ] Health checks implementados
- [ ] Graceful shutdown implementado

## 🚀 Deployment Readiness

### **📋 Pre-Deploy Checklist**
- [ ] Feature flags configuradas
- [ ] Database migrations testadas
- [ ] Rollback plan documentado
- [ ] Monitoring dashboards atualizados
- [ ] Documentation atualizada
- [ ] Team notificado sobre deploy

### **🔄 Post-Deploy Verification**
```bash
# Health Checks
curl -f https://api.example.com/health
curl -f https://api.example.com/metrics

# Error Monitoring
tail -f /var/log/app.log | grep ERROR
```

## 🎯 Quality Gates

### **❌ Bloquear Deploy se:**
- Code coverage < 80%
- Security vulnerabilities encontradas
- Performance benchmarks falharam
- Health checks não passaram
- Critical logs de erro apareceram

### **⚠️ Warning (revisar mas pode deplotar):**
- Code coverage entre 70-80%
- Minor performance degradation
- Non-critical warnings em logs

## 📈 Metrics to Track

### **🎯 KPIs de Qualidade**
- Response time médio < 200ms
- Error rate < 0.1%  
- Code coverage > 80%
- Security vulnerabilities = 0
- Memory usage estável
- CPU usage < 70%

### **📊 Business Metrics**
- Feature adoption rate
- User satisfaction score
- Bug reports pós-deploy
- System uptime > 99.9%

## 🔧 Tools Integration

### **CI/CD Pipeline Integration**
```yaml
# .github/workflows/post-implementation-review.yml
name: Post-Implementation Review
on: [push, pull_request]
jobs:
  quality-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Security Scan
        run: npm audit
      - name: Code Quality
        run: npm run lint
      - name: Tests
        run: npm test -- --coverage
      - name: Build
        run: npm run build
```

## 🎓 Team Review Process

### **👥 Code Review Requirements**
- [ ] Minimum 2 reviewers
- [ ] Security expert review (for sensitive changes)  
- [ ] Senior developer approval
- [ ] QA team sign-off
- [ ] Documentation review

### **📝 Review Checklist Template**
```markdown
## Code Review Checklist
- [ ] Code follows team standards
- [ ] Security best practices applied
- [ ] Performance considerations addressed
- [ ] Tests are comprehensive
- [ ] Documentation is updated
- [ ] Error handling is robust
- [ ] Logging is appropriate

## Business Impact
- [ ] Feature requirements met
- [ ] User experience improved
- [ ] Performance impact acceptable
- [ ] Security posture maintained

## Deployment Ready
- [ ] All checks passed
- [ ] Rollback plan documented
- [ ] Monitoring configured
```

---
**📌 Regra de Ouro**: Se algum item do checklist crítico falhar, **NÃO DEPLOY**. Corrija primeiro.

**🔄 Versão**: 1.0
**📅 Última atualização**: 2024-12-19
**👨‍💻 Criado por**: Universal AI-Powered Development Rules System
