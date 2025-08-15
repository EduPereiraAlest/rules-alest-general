# 🌍 Global Compliance & Internationalization Rules

## 🎯 Objetivo
Regras obrigatórias para compliance global (GDPR/LGPD), internacionalização e acessibilidade em aplicações enterprise.

## 🔐 Data Privacy & Compliance

### **📋 GDPR/LGPD Compliance Checklist**
```markdown
## Data Protection Requirements

### **🔍 Data Collection**
- [ ] Explicit consent mechanisms implemented
- [ ] Purpose limitation clearly defined
- [ ] Data minimization principle applied
- [ ] Legal basis for processing documented
- [ ] Cookie consent banner implemented
- [ ] Privacy policy accessible and clear

### **🛡️ Data Security**
- [ ] Data encryption at rest and in transit
- [ ] Access controls and authentication
- [ ] Regular security audits scheduled
- [ ] Data breach notification procedures
- [ ] Staff training on data protection
- [ ] Vendor data processing agreements

### **👤 Individual Rights**
- [ ] Right to access implementation
- [ ] Right to rectification mechanism
- [ ] Right to erasure (deletion) capability
- [ ] Right to data portability
- [ ] Right to object functionality
- [ ] Consent withdrawal mechanism
```

### **🔒 Data Handling Implementation**
```python
# Python Example - GDPR Compliant User Service
from datetime import datetime, timedelta
from enum import Enum

class ConsentType(Enum):
    ESSENTIAL = "essential"
    ANALYTICS = "analytics"
    MARKETING = "marketing"

class GDPRUserService:
    def __init__(self):
        self.logger = self._setup_audit_logger()
    
    def collect_consent(self, user_id: str, consents: Dict[ConsentType, bool]):
        """Collect and store user consent with audit trail"""
        consent_record = {
            'user_id': user_id,
            'consents': consents,
            'timestamp': datetime.utcnow(),
            'ip_address': self._get_anonymized_ip(),
            'user_agent': self._get_user_agent(),
            'consent_version': self.PRIVACY_POLICY_VERSION
        }
        
        # Store consent with audit trail
        self.db.consent_records.insert(consent_record)
        
        # Log for audit
        self.logger.info(f"Consent collected for user {user_id}", 
                        extra={'audit_type': 'consent_collection'})
    
    def request_data_export(self, user_id: str) -> str:
        """Handle data portability requests"""
        # Collect all user data
        user_data = self._collect_all_user_data(user_id)
        
        # Create export file
        export_file = self._generate_export_file(user_data)
        
        # Schedule automatic deletion after 30 days
        self._schedule_export_deletion(export_file, days=30)
        
        # Log request
        self.logger.info(f"Data export requested for user {user_id}",
                        extra={'audit_type': 'data_export'})
        
        return export_file
    
    def delete_user_data(self, user_id: str, deletion_reason: str):
        """Handle right to erasure requests"""
        # Check if user has active contracts/legal obligations
        if self._has_legal_obligations(user_id):
            raise ComplianceException("Cannot delete due to legal obligations")
        
        # Anonymize instead of delete for analytics data
        self._anonymize_analytics_data(user_id)
        
        # Delete personal data
        self._delete_personal_data(user_id)
        
        # Log deletion with reason
        self.logger.info(f"User data deleted: {deletion_reason}",
                        extra={'user_id': user_id, 'audit_type': 'data_deletion'})
```

## 🌐 Internationalization (i18n)

### **🗣️ Multi-Language Support**
```typescript
// TypeScript i18n Configuration
interface LocaleConfig {
  code: string;
  name: string;
  rtl: boolean;
  dateFormat: string;
  numberFormat: Intl.NumberFormatOptions;
  currencyFormat: Intl.NumberFormatOptions;
  pluralRules: (count: number) => string;
}

const SUPPORTED_LOCALES: Record<string, LocaleConfig> = {
  'pt-BR': {
    code: 'pt-BR',
    name: 'Português (Brasil)',
    rtl: false,
    dateFormat: 'DD/MM/YYYY',
    numberFormat: { locale: 'pt-BR' },
    currencyFormat: { style: 'currency', currency: 'BRL' },
    pluralRules: (count) => count === 1 ? 'one' : 'other'
  },
  'en-US': {
    code: 'en-US',
    name: 'English (United States)',
    rtl: false,
    dateFormat: 'MM/DD/YYYY',
    numberFormat: { locale: 'en-US' },
    currencyFormat: { style: 'currency', currency: 'USD' },
    pluralRules: (count) => count === 1 ? 'one' : 'other'
  },
  'ar-SA': {
    code: 'ar-SA',
    name: 'العربية (السعودية)',
    rtl: true,
    dateFormat: 'DD/MM/YYYY',
    numberFormat: { locale: 'ar-SA' },
    currencyFormat: { style: 'currency', currency: 'SAR' },
    pluralRules: (count) => count === 0 ? 'zero' : count === 1 ? 'one' : count === 2 ? 'two' : count <= 10 ? 'few' : 'many'
  }
};

class I18nService {
  private translations: Map<string, any> = new Map();
  private currentLocale: string = 'en-US';
  
  async loadTranslations(locale: string): Promise<void> {
    if (!SUPPORTED_LOCALES[locale]) {
      throw new Error(`Unsupported locale: ${locale}`);
    }
    
    // Load translations from API or files
    const translations = await this.fetchTranslations(locale);
    this.translations.set(locale, translations);
    this.currentLocale = locale;
  }
  
  t(key: string, params?: Record<string, any>): string {
    const translation = this.getNestedValue(
      this.translations.get(this.currentLocale),
      key
    );
    
    if (!translation) {
      console.warn(`Translation missing for key: ${key}`);
      return key;
    }
    
    return this.interpolate(translation, params);
  }
  
  formatDate(date: Date): string {
    const locale = SUPPORTED_LOCALES[this.currentLocale];
    return new Intl.DateTimeFormat(locale.code).format(date);
  }
  
  formatCurrency(amount: number): string {
    const locale = SUPPORTED_LOCALES[this.currentLocale];
    return new Intl.NumberFormat(locale.code, locale.currencyFormat).format(amount);
  }
}
```

### **🎨 UI/UX Internationalization**
```css
/* CSS for RTL Support */
.app {
  direction: ltr;
}

.app[dir="rtl"] {
  direction: rtl;
}

/* Use logical properties for i18n */
.content {
  margin-inline-start: 1rem;  /* margin-left in LTR, margin-right in RTL */
  margin-inline-end: 2rem;     /* margin-right in LTR, margin-left in RTL */
  padding-inline: 1rem;        /* padding-left and padding-right */
  border-inline-start: 1px solid #ccc;  /* border-left in LTR, border-right in RTL */
}

/* Text expansion consideration */
.button {
  min-width: 120px;  /* Account for German/Finnish text expansion */
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

/* Font stacks for different languages */
.font-latin {
  font-family: 'Roboto', 'Arial', sans-serif;
}

.font-arabic {
  font-family: 'Noto Sans Arabic', 'Arial', sans-serif;
}

.font-chinese {
  font-family: 'Noto Sans SC', 'PingFang SC', sans-serif;
}
```

## ♿ Accessibility (WCAG 2.1 AA)

### **🎯 WCAG Compliance Checklist**
```markdown
## Accessibility Requirements

### **👁️ Perceivable**
- [ ] Images have alt text
- [ ] Videos have captions/transcripts
- [ ] Color contrast ratio ≥ 4.5:1 (normal text)
- [ ] Color contrast ratio ≥ 3:1 (large text)
- [ ] Text can be resized up to 200% without loss of functionality
- [ ] Content doesn't rely solely on color to convey information

### **⌨️ Operable**
- [ ] All functionality available via keyboard
- [ ] Focus indicators visible and clear
- [ ] No seizure-inducing content (flashing < 3 times/second)
- [ ] Users can pause/stop animations
- [ ] Page has descriptive titles
- [ ] Focus order is logical

### **🧠 Understandable**
- [ ] Language of page is specified (lang attribute)
- [ ] Navigation is consistent across pages
- [ ] Form labels are clear and associated
- [ ] Error messages are descriptive
- [ ] Instructions are provided for complex interactions

### **🔧 Robust**  
- [ ] Valid HTML markup
- [ ] Compatible with assistive technologies
- [ ] ARIA labels used appropriately
- [ ] Semantic HTML elements used correctly
```

### **🛠️ Accessibility Implementation**
```html
<!-- Semantic HTML with ARIA -->
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>Sistema de Gestão - Dashboard Principal</title>
  <meta name="description" content="Dashboard principal do sistema de gestão empresarial">
</head>
<body>
  <!-- Skip link for keyboard users -->
  <a href="#main-content" class="skip-link">Pular para conteúdo principal</a>
  
  <!-- Main navigation -->
  <nav aria-label="Navegação principal">
    <ul>
      <li><a href="/dashboard" aria-current="page">Dashboard</a></li>
      <li><a href="/users">Usuários</a></li>
      <li><a href="/reports">Relatórios</a></li>
    </ul>
  </nav>
  
  <!-- Main content -->
  <main id="main-content">
    <h1>Dashboard Principal</h1>
    
    <!-- Data table with accessibility -->
    <table role="table" aria-label="Lista de usuários ativos">
      <caption>Usuários ativos no sistema (32 total)</caption>
      <thead>
        <tr>
          <th scope="col">Nome</th>
          <th scope="col">Email</th>
          <th scope="col">Último Acesso</th>
          <th scope="col">Ações</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td>João Silva</td>
          <td>joao@empresa.com</td>
          <td>
            <time datetime="2024-12-19T14:30:00Z">
              19/12/2024 às 14:30
            </time>
          </td>
          <td>
            <button aria-label="Editar usuário João Silva">
              <span aria-hidden="true">✏️</span>
              Editar
            </button>
          </td>
        </tr>
      </tbody>
    </table>
    
    <!-- Form with proper labels -->
    <form aria-labelledby="user-form-title">
      <h2 id="user-form-title">Adicionar Novo Usuário</h2>
      
      <div class="form-group">
        <label for="user-name">Nome Completo *</label>
        <input 
          type="text" 
          id="user-name" 
          name="name" 
          required 
          aria-describedby="name-help"
          aria-invalid="false"
        >
        <div id="name-help" class="form-help">
          Digite o nome completo do usuário
        </div>
      </div>
      
      <div class="form-group">
        <label for="user-email">Email *</label>
        <input 
          type="email" 
          id="user-email" 
          name="email" 
          required 
          aria-describedby="email-error"
          aria-invalid="true"
        >
        <div id="email-error" class="error-message" role="alert">
          Por favor, insira um email válido
        </div>
      </div>
      
      <button type="submit">Criar Usuário</button>
    </form>
  </main>
  
  <!-- Screen reader live region for status updates -->
  <div aria-live="polite" aria-atomic="true" class="sr-only" id="status-updates">
  </div>
</body>
</html>
```

## 🌏 Cultural Considerations

### **🎭 Cultural Adaptation Matrix**
```markdown
## Cultural Considerations by Region

### **🇧🇷 Brazil**
- **Colors**: Green/yellow positive, avoid purple (mourning)
- **Numbers**: Use comma for decimals (1,50 not 1.50)
- **Names**: Support compound surnames
- **Address**: CEP format, state abbreviations
- **Holidays**: Consider Carnival, regional holidays

### **🇺🇸 United States**  
- **Colors**: Red/white/blue patriotic, green for money
- **Numbers**: Period for decimals (1.50)
- **Names**: Middle name common
- **Address**: ZIP codes, state abbreviations
- **Holidays**: Thanksgiving, Independence Day

### **🇸🇦 Saudi Arabia**
- **Language**: RTL layout, Arabic numerals
- **Colors**: Green positive, avoid yellow
- **Calendar**: Hijri calendar support
- **Holidays**: Islamic holidays, Friday weekend
- **Culture**: Gender considerations in UI

### **🇩🇪 Germany**
- **Privacy**: Strict data protection expectations
- **Numbers**: Comma for decimals, space for thousands
- **Language**: Compound words cause text expansion
- **Holidays**: Oktoberfest, regional variations
```

### **📅 Multi-Calendar Support**
```javascript
// Multi-calendar system support
class CalendarService {
  private calendars = {
    gregorian: new Intl.DateTimeFormat('en-US'),
    hijri: new Intl.DateTimeFormat('ar-SA-u-ca-islamic'),
    persian: new Intl.DateTimeFormat('fa-IR-u-ca-persian'),
    hebrew: new Intl.DateTimeFormat('he-IL-u-ca-hebrew')
  };
  
  formatDate(date: Date, locale: string, calendar: string = 'gregorian'): string {
    const formatter = new Intl.DateTimeFormat(locale, {
      calendar: calendar,
      year: 'numeric',
      month: 'long',
      day: 'numeric'
    });
    
    return formatter.format(date);
  }
  
  getLocalHolidays(locale: string, year: number): Holiday[] {
    // Return localized holidays for the region
    const holidayMap = {
      'pt-BR': this.getBrazilHolidays(year),
      'en-US': this.getUSHolidays(year),
      'ar-SA': this.getSaudiHolidays(year),
      'de-DE': this.getGermanHolidays(year)
    };
    
    return holidayMap[locale] || [];
  }
}
```

## 🔍 Compliance Monitoring

### **📊 Audit Trail System**
```python
# Compliance audit trail implementation
class ComplianceAuditLogger:
    def __init__(self):
        self.logger = self._setup_compliance_logger()
    
    def log_data_access(self, user_id: str, accessed_data: str, purpose: str):
        """Log data access for GDPR compliance"""
        self.logger.info({
            'event_type': 'data_access',
            'user_id': user_id,
            'accessed_data': accessed_data,
            'purpose': purpose,
            'timestamp': datetime.utcnow().isoformat(),
            'ip_address': self._get_client_ip(),
            'retention_period': self._get_retention_period(accessed_data)
        })
    
    def log_consent_change(self, user_id: str, old_consent: dict, new_consent: dict):
        """Log consent changes for audit"""
        self.logger.info({
            'event_type': 'consent_change',
            'user_id': user_id,
            'old_consent': old_consent,
            'new_consent': new_consent,
            'timestamp': datetime.utcnow().isoformat(),
            'changes': self._diff_consents(old_consent, new_consent)
        })
    
    def schedule_data_retention_cleanup(self):
        """Schedule automatic data cleanup based on retention policies"""
        retention_policies = self._load_retention_policies()
        
        for policy in retention_policies:
            self._schedule_cleanup_job(
                data_type=policy.data_type,
                retention_days=policy.retention_days,
                cleanup_method=policy.cleanup_method
            )
```

### **🛡️ Compliance Testing**
```yaml
# Compliance test automation
compliance_tests:
  gdpr:
    - test: "consent_collection"
      endpoint: "/api/consent"
      validates: "explicit consent recorded"
      
    - test: "data_export"
      endpoint: "/api/users/{id}/export"
      validates: "complete data export generated"
      
    - test: "data_deletion"
      endpoint: "/api/users/{id}/delete"
      validates: "all personal data removed"
      
  accessibility:
    - test: "wcag_aa_compliance"
      tool: "axe-core"
      pages: ["login", "dashboard", "forms"]
      
    - test: "keyboard_navigation"
      validates: "all features accessible via keyboard"
      
  internationalization:
    - test: "text_expansion"
      locales: ["de-DE", "fi-FI"]
      validates: "UI handles 30% text expansion"
      
    - test: "rtl_layout"
      locales: ["ar-SA", "he-IL"]
      validates: "proper RTL rendering"
```

---

**🌍 Regra de Ouro**: Compliance não é opcional - toda aplicação enterprise deve atender regulamentações globais desde o primeiro dia.

**🔐 Lembrete**: Privacidade e acessibilidade são direitos fundamentais, não recursos opcionais.

**🔄 Versão**: 1.0  
**📅 Última atualização**: 2024-12-19  
**👨‍💻 Criado por**: Universal AI-Powered Development Rules System
