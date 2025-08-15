# 📚 Documentation Standards Rules

## 🎯 Objetivo
Padrões obrigatórios para documentação viva, API documentation, comentários de código e knowledge management em projetos enterprise.

## 📖 Living Documentation

### **🔄 Documentation as Code**
```yaml
# .github/workflows/docs-sync.yml
name: Documentation Sync
on:
  push:
    branches: [main, develop]
  pull_request:
    paths: ['src/**', 'docs/**', '*.md']

jobs:
  docs-validation:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Validate Documentation
        run: |
          # Check for broken links
          markdown-link-check docs/**/*.md
          
          # Validate code examples in docs
          npm run docs:validate-examples
          
          # Check for outdated documentation
          npm run docs:freshness-check
          
      - name: Generate API Docs
        run: |
          # Auto-generate from code annotations
          npm run docs:api-generate
          
      - name: Deploy to Documentation Site
        if: github.ref == 'refs/heads/main'
        run: |
          npm run docs:deploy
```

### **📝 Documentation Structure**
```
docs/
├── README.md                     # Project overview
├── ARCHITECTURE.md               # System architecture
├── API.md                       # API documentation  
├── DEPLOYMENT.md                # Deployment guide
├── CONTRIBUTING.md              # Contribution guidelines
├── CHANGELOG.md                 # Version history
├── TROUBLESHOOTING.md           # Common issues
├── architecture/
│   ├── decisions/               # Architecture Decision Records (ADRs)
│   │   ├── 001-database-choice.md
│   │   └── 002-authentication-strategy.md
│   ├── diagrams/               # System diagrams
│   └── patterns/               # Design patterns used
├── api/
│   ├── endpoints/              # API endpoint documentation
│   ├── schemas/                # Data schemas
│   └── examples/               # Request/response examples
├── guides/
│   ├── quick-start.md          # Getting started guide
│   ├── development.md          # Development setup
│   └── production.md           # Production deployment
└── runbooks/
    ├── incident-response.md    # Incident procedures
    ├── maintenance.md          # Maintenance tasks
    └── monitoring.md           # Monitoring guidelines
```

## 🏗️ Architecture Decision Records (ADRs)

### **📋 ADR Template**
```markdown
# ADR-001: [Title of Decision]

## Status
[Proposed | Accepted | Deprecated | Superseded]

## Context
[What is the issue that we're seeing that is motivating this decision or change?]

## Decision
[What is the change that we're proposing and/or doing?]

## Consequences
[What becomes easier or more difficult to do because of this change?]

### Positive Consequences
- [Positive consequence 1]
- [Positive consequence 2]

### Negative Consequences  
- [Negative consequence 1]
- [Negative consequence 2]

## Alternatives Considered
- [Alternative 1: brief description and why it was not chosen]
- [Alternative 2: brief description and why it was not chosen]

## Implementation
[How will this decision be implemented?]

## Monitoring
[How will we know if this decision is working?]

---
**Created**: 2024-12-19  
**Updated**: 2024-12-19  
**Author**: [Name]  
**Reviewers**: [Names]
```

### **🔄 ADR Lifecycle Management**
```python
# ADR Management Tool
import os
import datetime
from pathlib import Path

class ADRManager:
    def __init__(self, docs_path: str = "./docs/architecture/decisions"):
        self.docs_path = Path(docs_path)
        self.docs_path.mkdir(parents=True, exist_ok=True)
    
    def create_adr(self, title: str, author: str) -> str:
        """Create new ADR with template"""
        adr_number = self._get_next_number()
        filename = f"{adr_number:03d}-{self._slugify(title)}.md"
        filepath = self.docs_path / filename
        
        template = self._load_template()
        content = template.format(
            number=adr_number,
            title=title,
            date=datetime.date.today().isoformat(),
            author=author
        )
        
        with open(filepath, 'w') as f:
            f.write(content)
        
        return str(filepath)
    
    def list_adrs(self) -> List[Dict]:
        """List all ADRs with metadata"""
        adrs = []
        for file in self.docs_path.glob("*.md"):
            metadata = self._extract_metadata(file)
            adrs.append(metadata)
        
        return sorted(adrs, key=lambda x: x['number'])
    
    def supersede_adr(self, old_number: int, new_adr_path: str):
        """Mark ADR as superseded"""
        old_file = self._find_adr_by_number(old_number)
        if old_file:
            self._update_status(old_file, f"Superseded by {new_adr_path}")
```

## 📡 API Documentation

### **🔧 OpenAPI 3.0 Standards**
```yaml
# api-spec.yml
openapi: 3.0.3
info:
  title: Enterprise API
  description: |
    # Enterprise API Documentation
    
    This API provides core business functionality with enterprise-grade security and monitoring.
    
    ## Authentication
    All endpoints require JWT authentication via `Authorization: Bearer <token>` header.
    
    ## Rate Limiting  
    - 1000 requests/hour for authenticated users
    - 100 requests/hour for unauthenticated endpoints
    
    ## Error Handling
    All errors follow RFC 7807 Problem Details format.
    
  version: 1.2.0
  contact:
    name: API Support Team
    email: api-support@company.com
    url: https://company.com/api-support
  license:
    name: MIT
    url: https://opensource.org/licenses/MIT

servers:
  - url: https://api.company.com/v1
    description: Production server
  - url: https://staging-api.company.com/v1  
    description: Staging server
  - url: http://localhost:3000/v1
    description: Development server

paths:
  /users:
    get:
      summary: List users
      description: |
        Retrieve a paginated list of users with optional filtering.
        
        ### Filtering
        - `role`: Filter by user role (admin, user, guest)
        - `active`: Filter by active status (true/false)
        - `created_after`: Filter by creation date (ISO 8601)
        
        ### Sorting
        - `sort`: Sort field (name, email, created_at)
        - `order`: Sort order (asc, desc)
        
        ### Examples
        ```bash
        # Get active admin users
        curl -X GET "https://api.company.com/v1/users?role=admin&active=true"
        
        # Get users created after specific date  
        curl -X GET "https://api.company.com/v1/users?created_after=2024-01-01T00:00:00Z"
        ```
      parameters:
        - name: page
          in: query
          description: Page number (1-based)
          required: false
          schema:
            type: integer
            minimum: 1
            default: 1
        - name: limit
          in: query
          description: Items per page
          required: false
          schema:
            type: integer
            minimum: 1
            maximum: 100
            default: 20
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: array
                    items:
                      $ref: '#/components/schemas/User'
                  pagination:
                    $ref: '#/components/schemas/Pagination'
              examples:
                success:
                  summary: Successful user list
                  value:
                    data:
                      - id: 1
                        name: "João Silva"
                        email: "joao@empresa.com"
                        role: "admin"
                        active: true
                    pagination:
                      page: 1
                      limit: 20
                      total: 150
                      pages: 8

components:
  schemas:
    User:
      type: object
      required: [id, name, email]
      properties:
        id:
          type: integer
          description: Unique user identifier
          example: 1
        name:
          type: string
          description: Full user name
          example: "João Silva"
          minLength: 2
          maxLength: 100
        email:
          type: string
          format: email
          description: User email address
          example: "joao@empresa.com"
        role:
          type: string
          enum: [admin, user, guest]
          description: User role in the system
          example: "user"
        active:
          type: boolean
          description: Whether the user is active
          example: true
        created_at:
          type: string
          format: date-time
          description: User creation timestamp
          example: "2024-01-15T10:30:00Z"
          
  securitySchemes:
    BearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

security:
  - BearerAuth: []
```

### **📖 Code-Generated Documentation**
```typescript
// API Documentation Annotations
/**
 * @swagger
 * /users/{id}:
 *   get:
 *     summary: Get user by ID
 *     description: |
 *       Retrieve a single user by their unique identifier.
 *       
 *       **Business Rules:**
 *       - Users can only access their own profile unless they have admin role
 *       - Deleted users return 404 even if ID exists
 *       - Suspended users return limited information
 *       
 *     tags: [Users]
 *     security:
 *       - BearerAuth: []
 *     parameters:
 *       - name: id
 *         in: path
 *         required: true
 *         description: User unique identifier
 *         schema:
 *           type: integer
 *           minimum: 1
 *         example: 123
 *     responses:
 *       200:
 *         description: User found successfully
 *         content:
 *           application/json:
 *             schema:
 *               $ref: '#/components/schemas/User'
 *       404:
 *         description: User not found
 *         content:
 *           application/json:
 *             schema:
 *               $ref: '#/components/schemas/Error'
 *             example:
 *               error:
 *                 code: "USER_NOT_FOUND"
 *                 message: "User with ID 123 not found"
 *                 details: {}
 */
export async function getUserById(req: Request, res: Response): Promise<void> {
  const { id } = req.params;
  
  try {
    const user = await userService.findById(parseInt(id));
    if (!user) {
      return res.status(404).json({
        error: {
          code: 'USER_NOT_FOUND',
          message: `User with ID ${id} not found`,
          details: {}
        }
      });
    }
    
    res.json(user);
  } catch (error) {
    // Error handling...
  }
}
```

## 💬 Code Comments Standards

### **📝 Multi-Language Comment Standards**

```python
# Python Documentation Standards
class UserService:
    """
    Service for user management operations.
    
    This service handles all user-related business logic including
    authentication, authorization, and profile management.
    
    Attributes:
        db: Database connection instance
        cache: Redis cache instance for user sessions
        
    Example:
        >>> service = UserService(db_conn, redis_conn)
        >>> user = service.create_user("john@example.com", "password123")
        >>> print(user.id)
        1
    """
    
    def create_user(self, email: str, password: str) -> User:
        """
        Create a new user account.
        
        Args:
            email: User's email address (must be unique)
            password: Plain text password (will be hashed)
            
        Returns:
            User: Created user instance with generated ID
            
        Raises:
            ValidationError: If email is invalid or already exists
            SecurityError: If password doesn't meet requirements
            
        Business Rules:
            - Email must be unique across the system
            - Password must be at least 8 characters
            - User starts with 'user' role by default
            - Account verification email is sent automatically
            
        Example:
            >>> user = service.create_user("jane@example.com", "SecurePass123!")
            >>> assert user.email == "jane@example.com"
            >>> assert user.role == "user"
        """
        # Implementation details...
        pass
```

```java
// Java Documentation Standards
/**
 * Service for managing user operations in enterprise environment.
 * 
 * <p>This service provides comprehensive user management functionality
 * including CRUD operations, authentication, and business rule enforcement.
 * 
 * <p><b>Thread Safety:</b> This class is thread-safe and can be used
 * in multi-threaded environments.
 * 
 * <p><b>Performance:</b> All database operations use connection pooling
 * and include appropriate caching strategies.
 * 
 * @author Development Team
 * @version 1.2.0
 * @since 1.0.0
 * 
 * @see User
 * @see UserRepository
 */
@Service
@Transactional
public class UserService {
    
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
    private final EmailService emailService;
    
    /**
     * Creates a new user account with email verification.
     * 
     * <p>This method performs comprehensive validation and applies all
     * business rules for user registration. The password is automatically
     * hashed using BCrypt algorithm.
     * 
     * @param email the user's email address (must be unique)
     * @param password the plain text password (min 8 chars)
     * @return the created User entity with generated ID
     * 
     * @throws ValidationException if email format is invalid
     * @throws DuplicateEmailException if email already exists
     * @throws WeakPasswordException if password doesn't meet requirements
     * 
     * @implNote The method sends verification email asynchronously
     * @implSpec Password requirements: min 8 chars, 1 upper, 1 lower, 1 digit
     * 
     * <pre>{@code
     * // Example usage:
     * User newUser = userService.createUser("john@example.com", "SecurePass123!");
     * assert newUser.getId() != null;
     * assert newUser.isEmailVerified() == false;
     * }</pre>
     */
    public User createUser(@Valid @NotBlank String email, 
                          @Valid @NotBlank String password) {
        // Implementation...
    }
}
```

```csharp
// C# Documentation Standards
/// <summary>
/// Service for comprehensive user management in enterprise applications.
/// </summary>
/// <remarks>
/// <para>
/// This service handles all user-related operations including authentication,
/// authorization, profile management, and business rule enforcement.
/// </para>
/// <para>
/// <b>Thread Safety:</b> This service is thread-safe and supports concurrent operations.
/// </para>
/// <para>
/// <b>Dependencies:</b> Requires IUserRepository, IPasswordHasher, and IEmailService.
/// </para>
/// </remarks>
/// <example>
/// <code>
/// var userService = new UserService(repository, hasher, emailService);
/// var user = await userService.CreateUserAsync("john@example.com", "SecurePass123!");
/// Console.WriteLine($"Created user: {user.Id}");
/// </code>
/// </example>
public class UserService : IUserService
{
    /// <summary>
    /// Creates a new user account with automatic email verification.
    /// </summary>
    /// <param name="email">The user's email address (must be unique)</param>
    /// <param name="password">The plain text password (will be hashed)</param>
    /// <returns>
    /// A <see cref="Task{User}"/> representing the asynchronous operation.
    /// The task result contains the created user with generated ID.
    /// </returns>
    /// <exception cref="ArgumentException">Thrown when email format is invalid</exception>
    /// <exception cref="DuplicateEmailException">Thrown when email already exists</exception>
    /// <exception cref="WeakPasswordException">Thrown when password doesn't meet requirements</exception>
    /// <remarks>
    /// <para><b>Business Rules:</b></para>
    /// <list type="bullet">
    /// <item>Email must be unique across the system</item>
    /// <item>Password must be at least 8 characters with mixed case and numbers</item>
    /// <item>New users start with 'User' role by default</item>
    /// <item>Email verification is sent automatically</item>
    /// </list>
    /// <para><b>Performance:</b> This method uses async/await for database operations.</para>
    /// </remarks>
    /// <example>
    /// <code>
    /// try 
    /// {
    ///     var user = await userService.CreateUserAsync("jane@example.com", "SecurePass123!");
    ///     Console.WriteLine($"User created with ID: {user.Id}");
    /// }
    /// catch (DuplicateEmailException ex)
    /// {
    ///     Console.WriteLine("Email already exists");
    /// }
    /// </code>
    /// </example>
    public async Task<User> CreateUserAsync(string email, string password)
    {
        // Implementation...
    }
}
```

### **🔍 Inline Comments Best Practices**
```typescript
// TypeScript/JavaScript Standards

export class PaymentProcessor {
  async processPayment(amount: number, paymentMethod: PaymentMethod): Promise<PaymentResult> {
    // BUSINESS RULE: Minimum payment amount is $1.00
    // This prevents micro-transactions that cost more to process than they're worth
    if (amount < 1.00) {
      throw new ValidationError('Minimum payment amount is $1.00');
    }
    
    // SECURITY: Validate payment method hasn't expired
    // We check this before calling external payment gateway to avoid fees
    if (this.isPaymentMethodExpired(paymentMethod)) {
      throw new PaymentMethodExpiredError('Payment method has expired');
    }
    
    // PERFORMANCE: Use circuit breaker pattern for external calls
    // This prevents cascade failures when payment gateway is down
    const result = await this.circuitBreaker.execute(async () => {
      return this.paymentGateway.charge(amount, paymentMethod);
    });
    
    // AUDIT TRAIL: Log all payment attempts for compliance
    // Required for PCI DSS compliance and fraud detection
    await this.auditLogger.logPaymentAttempt({
      amount,
      result: result.success ? 'SUCCESS' : 'FAILURE',
      timestamp: new Date(),
      // NOTE: Never log sensitive payment details
      paymentMethodType: paymentMethod.type
    });
    
    return result;
  }
  
  private isPaymentMethodExpired(method: PaymentMethod): boolean {
    // TODO: Add grace period for expired cards (business requirement)
    // HACK: Some cards work 1-2 days past expiration - needs business decision
    const now = new Date();
    const expiry = new Date(method.expiryYear, method.expiryMonth - 1);
    return now > expiry;
  }
}
```

## 📊 Documentation Metrics

### **📈 Documentation Quality Metrics**
```python
# Documentation Quality Analyzer
import ast
import re
from pathlib import Path

class DocumentationAnalyzer:
    """Analyze code documentation quality and coverage."""
    
    def analyze_project(self, project_path: str) -> Dict[str, any]:
        """
        Analyze documentation quality across entire project.
        
        Returns comprehensive metrics including:
        - Code coverage by documentation
        - Comment quality scores
        - API documentation completeness
        - Outdated documentation detection
        """
        metrics = {
            'code_coverage': self._calculate_doc_coverage(project_path),
            'comment_quality': self._analyze_comment_quality(project_path),
            'api_completeness': self._check_api_documentation(project_path),
            'freshness_score': self._calculate_freshness(project_path),
            'readability_score': self._calculate_readability(project_path)
        }
        
        return metrics
    
    def _calculate_doc_coverage(self, path: str) -> float:
        """Calculate percentage of functions/classes with documentation."""
        total_definitions = 0
        documented_definitions = 0
        
        for file_path in Path(path).rglob("*.py"):
            with open(file_path, 'r') as f:
                tree = ast.parse(f.read())
                
            for node in ast.walk(tree):
                if isinstance(node, (ast.FunctionDef, ast.ClassDef)):
                    total_definitions += 1
                    if ast.get_docstring(node):
                        documented_definitions += 1
        
        return (documented_definitions / total_definitions) * 100 if total_definitions > 0 else 0
    
    def _analyze_comment_quality(self, path: str) -> Dict[str, float]:
        """Analyze quality of comments using various heuristics."""
        quality_metrics = {
            'comment_to_code_ratio': 0,
            'avg_comment_length': 0,
            'useful_comments_ratio': 0  # Comments that explain 'why', not 'what'
        }
        
        # Implementation for quality analysis...
        return quality_metrics
```

### **🎯 Documentation Automation**
```yaml
# .github/workflows/docs-quality.yml
name: Documentation Quality Check

on:
  pull_request:
    paths: ['src/**', 'docs/**']

jobs:
  docs-quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Check Documentation Coverage
        run: |
          python scripts/doc_analyzer.py --threshold 80
          # Fail if documentation coverage < 80%
          
      - name: Validate API Documentation
        run: |
          swagger-codegen validate api-spec.yml
          redoc-cli build api-spec.yml --output api-docs.html
          
      - name: Check for Broken Links
        uses: lycheeverse/lychee-action@v1
        with:
          args: --verbose --no-progress 'docs/**/*.md'
          
      - name: Spelling and Grammar Check
        uses: rojopolis/spellcheck-github-actions@0.5.0
        with:
          config_path: .spellcheck.yml
          
      - name: Generate Documentation Report
        run: |
          echo "## 📊 Documentation Quality Report" >> $GITHUB_STEP_SUMMARY
          echo "- Coverage: $(python scripts/doc_coverage.py)%" >> $GITHUB_STEP_SUMMARY
          echo "- Broken Links: $(wc -l < broken_links.txt)" >> $GITHUB_STEP_SUMMARY
          echo "- Spelling Errors: $(wc -l < spelling_errors.txt)" >> $GITHUB_STEP_SUMMARY
```

---

**📚 Regra de Ouro**: Documentação não é opcional - é parte integral do código e deve evoluir junto com ele.

**🔄 Lembrete**: Documentação desatualizada é pior que nenhuma documentação - mantenha sempre sincronizada.

**🔄 Versão**: 1.0  
**📅 Última atualização**: 2024-12-19  
**👨‍💻 Criado por**: Universal AI-Powered Development Rules System
