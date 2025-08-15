# 🏗️ Environment Management Rules

## 🎯 Objetivo
Regras obrigatórias para gerenciamento de ambientes (dev/staging/prod), configuração, secrets management e paridade entre ambientes.

## 🌍 Environment Parity

### **📋 12-Factor App Environment Standards**
```yaml
# Environment Configuration Matrix
environments:
  development:
    purpose: "Local development and testing"
    data_strategy: "Synthetic/anonymized data"
    scaling: "Single instance"
    monitoring: "Basic logging"
    security: "Relaxed for development speed"
    
  testing:
    purpose: "Automated testing and QA validation"  
    data_strategy: "Test fixtures and mocks"
    scaling: "Single instance with test doubles"
    monitoring: "Test result reporting"
    security: "Production-like security testing"
    
  staging:
    purpose: "Production replica for final validation"
    data_strategy: "Production-like anonymized data"
    scaling: "Production-equivalent scaling"
    monitoring: "Full production monitoring"
    security: "Identical to production"
    
  production:
    purpose: "Live customer-facing environment"
    data_strategy: "Real customer data"
    scaling: "Auto-scaling based on demand"
    monitoring: "Comprehensive monitoring and alerting"
    security: "Maximum security posture"
```

### **🔧 Environment Bootstrapping**
```bash
#!/bin/bash
# bootstrap-environment.sh

set -euo pipefail

ENVIRONMENT=${1:-development}
PROJECT_NAME=${2:-myapp}

echo "🚀 Bootstrapping ${ENVIRONMENT} environment for ${PROJECT_NAME}"

# Create environment-specific infrastructure
case $ENVIRONMENT in
  "development")
    echo "📱 Setting up development environment"
    docker-compose -f docker-compose.dev.yml up -d
    ./scripts/seed-dev-data.sh
    ;;
    
  "staging")
    echo "🎭 Setting up staging environment"
    terraform workspace select staging || terraform workspace new staging
    terraform apply -var-file="envs/staging.tfvars"
    ./scripts/migrate-database.sh staging
    ./scripts/seed-staging-data.sh
    ;;
    
  "production")
    echo "🏭 Setting up production environment"
    # Production deployments require additional approvals
    if [[ ! -f ".prod-deployment-approved" ]]; then
      echo "❌ Production deployment requires approval file"
      exit 1
    fi
    
    terraform workspace select production
    terraform plan -var-file="envs/production.tfvars" -out=production.tfplan
    
    # Require manual confirmation for production
    read -p "Deploy to PRODUCTION? (yes/no): " confirm
    if [[ $confirm == "yes" ]]; then
      terraform apply production.tfplan
      ./scripts/migrate-database.sh production
    fi
    ;;
    
  *)
    echo "❌ Unknown environment: $ENVIRONMENT"
    exit 1
    ;;
esac

echo "✅ Environment $ENVIRONMENT bootstrapped successfully"
```

## 🔐 Configuration Management

### **📝 Environment-Specific Configuration**
```python
# Python Configuration Management
import os
from dataclasses import dataclass
from typing import Optional
from enum import Enum

class Environment(Enum):
    DEVELOPMENT = "development"
    TESTING = "testing" 
    STAGING = "staging"
    PRODUCTION = "production"

@dataclass
class DatabaseConfig:
    host: str
    port: int
    database: str
    username: str
    password: str  # Retrieved from secrets manager
    ssl_mode: str = "require"
    connection_pool_size: int = 10
    connection_timeout: int = 30

@dataclass
class RedisConfig:
    host: str
    port: int = 6379
    password: Optional[str] = None
    ssl: bool = False
    db: int = 0

@dataclass
class AppConfig:
    """Application configuration with environment-specific settings"""
    
    # Environment identification
    environment: Environment
    debug: bool = False
    
    # Database configuration
    database: DatabaseConfig
    redis: RedisConfig
    
    # API configuration
    api_base_url: str
    api_timeout: int = 30
    api_rate_limit: int = 1000
    
    # Security configuration
    jwt_secret_key: str  # From secrets manager
    jwt_expiration_hours: int = 24
    password_salt_rounds: int = 12
    
    # Feature flags
    feature_flags: dict = None
    
    # Monitoring
    sentry_dsn: Optional[str] = None
    datadog_api_key: Optional[str] = None
    
    @classmethod
    def load_from_environment(cls) -> 'AppConfig':
        """Load configuration from environment variables and secrets"""
        env_name = os.getenv('ENVIRONMENT', 'development')
        environment = Environment(env_name)
        
        # Load secrets from appropriate secret manager
        secrets = cls._load_secrets(environment)
        
        return cls(
            environment=environment,
            debug=environment in [Environment.DEVELOPMENT, Environment.TESTING],
            
            database=DatabaseConfig(
                host=os.getenv('DB_HOST', 'localhost'),
                port=int(os.getenv('DB_PORT', '5432')),
                database=os.getenv('DB_NAME', f'myapp_{env_name}'),
                username=os.getenv('DB_USER', 'postgres'),
                password=secrets.get('db_password'),
                connection_pool_size=int(os.getenv('DB_POOL_SIZE', '10')),
            ),
            
            redis=RedisConfig(
                host=os.getenv('REDIS_HOST', 'localhost'),
                port=int(os.getenv('REDIS_PORT', '6379')),
                password=secrets.get('redis_password'),
                ssl=environment == Environment.PRODUCTION,
            ),
            
            api_base_url=os.getenv('API_BASE_URL', f'https://api-{env_name}.company.com'),
            jwt_secret_key=secrets.get('jwt_secret_key'),
            
            feature_flags=cls._load_feature_flags(environment),
            sentry_dsn=secrets.get('sentry_dsn') if environment != Environment.DEVELOPMENT else None,
        )
    
    @staticmethod
    def _load_secrets(environment: Environment) -> dict:
        """Load secrets from environment-appropriate secret store"""
        if environment == Environment.DEVELOPMENT:
            # Use .env file for development
            return {
                'db_password': os.getenv('DB_PASSWORD', 'devpassword'),
                'jwt_secret_key': 'dev-jwt-secret-key',
                'redis_password': None,
            }
        else:
            # Use cloud secret manager for other environments
            from .secrets_manager import SecretsManager
            secrets_manager = SecretsManager(environment.value)
            return secrets_manager.get_all_secrets()
    
    @staticmethod 
    def _load_feature_flags(environment: Environment) -> dict:
        """Load feature flags for environment"""
        # Integration with feature flag service
        base_flags = {
            'new_user_onboarding': True,
            'advanced_analytics': False,
            'beta_features': False,
        }
        
        if environment == Environment.DEVELOPMENT:
            base_flags.update({
                'debug_mode': True,
                'beta_features': True,
            })
        elif environment == Environment.STAGING:
            base_flags.update({
                'beta_features': True,
            })
            
        return base_flags
```

### **🔑 Secrets Management**
```typescript
// TypeScript Secrets Manager
interface SecretsConfig {
  provider: 'aws-secrets-manager' | 'azure-key-vault' | 'hashicorp-vault' | 'local-env';
  region?: string;
  vaultUrl?: string;
  authMethod?: 'iam' | 'service-principal' | 'token';
}

class SecretsManager {
  private provider: SecretsConfig['provider'];
  private client: any;
  
  constructor(private config: SecretsConfig) {
    this.provider = config.provider;
    this.client = this.initializeClient();
  }
  
  private initializeClient() {
    switch (this.provider) {
      case 'aws-secrets-manager':
        const { SecretsManagerClient } = require('@aws-sdk/client-secrets-manager');
        return new SecretsManagerClient({ region: this.config.region });
        
      case 'azure-key-vault':
        const { SecretClient } = require('@azure/keyvault-secrets');
        const { DefaultAzureCredential } = require('@azure/identity');
        return new SecretClient(this.config.vaultUrl!, new DefaultAzureCredential());
        
      case 'hashicorp-vault':
        const vault = require('node-vault');
        return vault({ apiVersion: 'v1', endpoint: this.config.vaultUrl });
        
      case 'local-env':
        return null; // Use process.env directly
        
      default:
        throw new Error(`Unsupported secrets provider: ${this.provider}`);
    }
  }
  
  async getSecret(secretName: string): Promise<string | null> {
    try {
      switch (this.provider) {
        case 'aws-secrets-manager':
          const { GetSecretValueCommand } = require('@aws-sdk/client-secrets-manager');
          const response = await this.client.send(new GetSecretValueCommand({
            SecretId: secretName
          }));
          return response.SecretString;
          
        case 'azure-key-vault':
          const secret = await this.client.getSecret(secretName);
          return secret.value || null;
          
        case 'hashicorp-vault':
          const result = await this.client.read(`secret/data/${secretName}`);
          return result.data.data.value;
          
        case 'local-env':
          return process.env[secretName] || null;
          
        default:
          throw new Error(`Unsupported provider: ${this.provider}`);
      }
    } catch (error) {
      console.error(`Failed to retrieve secret ${secretName}:`, error);
      return null;
    }
  }
  
  async setSecret(secretName: string, secretValue: string): Promise<boolean> {
    if (this.provider === 'local-env') {
      throw new Error('Cannot set secrets in local-env mode');
    }
    
    try {
      switch (this.provider) {
        case 'aws-secrets-manager':
          const { CreateSecretCommand, UpdateSecretCommand } = require('@aws-sdk/client-secrets-manager');
          try {
            await this.client.send(new CreateSecretCommand({
              Name: secretName,
              SecretString: secretValue
            }));
          } catch (error: any) {
            if (error.name === 'ResourceExistsException') {
              await this.client.send(new UpdateSecretCommand({
                SecretId: secretName,
                SecretString: secretValue
              }));
            }
          }
          return true;
          
        case 'azure-key-vault':
          await this.client.setSecret(secretName, secretValue);
          return true;
          
        case 'hashicorp-vault':
          await this.client.write(`secret/data/${secretName}`, {
            data: { value: secretValue }
          });
          return true;
          
        default:
          return false;
      }
    } catch (error) {
      console.error(`Failed to set secret ${secretName}:`, error);
      return false;
    }
  }
}

// Usage example
const secretsManager = new SecretsManager({
  provider: process.env.NODE_ENV === 'production' ? 'aws-secrets-manager' : 'local-env',
  region: 'us-east-1'
});

export { SecretsManager };
```

## 🚀 Deployment Strategies

### **🌊 Blue-Green Deployment**
```yaml
# Kubernetes Blue-Green Deployment
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: myapp-rollout
  namespace: production
spec:
  replicas: 5
  strategy:
    blueGreen:
      # Active service points to blue or green deployment
      activeService: myapp-active
      # Preview service for testing green deployment  
      previewService: myapp-preview
      # Auto-promote after successful checks
      autoPromotionEnabled: false
      # Rollback window
      scaleDownDelaySeconds: 30
      prePromotionAnalysis:
        templates:
          - templateName: success-rate
        args:
          - name: service-name
            value: myapp-preview
      postPromotionAnalysis:
        templates:
          - templateName: success-rate
        args:
          - name: service-name  
            value: myapp-active
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: myapp:latest
          ports:
            - containerPort: 8080
          env:
            - name: ENVIRONMENT
              value: "production"
          readinessProbe:
            httpGet:
              path: /health/ready
              port: 8080
            initialDelaySeconds: 10
            periodSeconds: 5
          livenessProbe:
            httpGet:
              path: /health/live
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 10

---
# Analysis Template for Success Rate
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: success-rate
spec:
  args:
    - name: service-name
  metrics:
    - name: success-rate
      interval: 2m
      successCondition: result[0] >= 0.95
      failureLimit: 3
      provider:
        prometheus:
          address: http://prometheus.monitoring.svc.cluster.local:9090
          query: |
            sum(rate(http_requests_total{service="{{args.service-name}}",status!~"5.."}[2m])) /
            sum(rate(http_requests_total{service="{{args.service-name}}"}[2m]))
```

### **🔄 Canary Deployment**
```yaml
# Canary Deployment Configuration
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: myapp-canary
spec:
  replicas: 10
  strategy:
    canary:
      # Canary service for new version traffic
      canaryService: myapp-canary
      # Stable service for current version traffic  
      stableService: myapp-stable
      steps:
        # Start with 10% traffic to canary
        - setWeight: 10
        - pause: {duration: 2m}
        
        # Increase to 20% with analysis
        - setWeight: 20
        - analysis:
            templates:
              - templateName: canary-success-rate
            args:
              - name: canary-hash
                valueFrom:
                  podTemplateHashValue: Latest
                  
        # Gradual rollout with validation
        - setWeight: 40
        - pause: {duration: 5m}
        - setWeight: 60
        - pause: {duration: 10m}
        - setWeight: 80
        - pause: {duration: 10m}
        
        # Full rollout
        - setWeight: 100
        - pause: {duration: 5m}

---
# Canary Analysis Template
apiVersion: argoproj.io/v1alpha1  
kind: AnalysisTemplate
metadata:
  name: canary-success-rate
spec:
  args:
    - name: canary-hash
  metrics:
    - name: success-rate
      successCondition: result[0] >= 0.95
      interval: 1m
      count: 5
      provider:
        prometheus:
          address: http://prometheus.monitoring.svc.cluster.local:9090
          query: |
            sum(rate(http_requests_total{rollouts_pod_template_hash="{{args.canary-hash}}",status!~"5.."}[2m])) /
            sum(rate(http_requests_total{rollouts_pod_template_hash="{{args.canary-hash}}"}[2m]))
            
    - name: error-rate
      successCondition: result[0] <= 0.05
      interval: 1m
      count: 5
      provider:
        prometheus:
          address: http://prometheus.monitoring.svc.cluster.local:9090
          query: |
            sum(rate(http_requests_total{rollouts_pod_template_hash="{{args.canary-hash}}",status=~"5.."}[2m])) /
            sum(rate(http_requests_total{rollouts_pod_template_hash="{{args.canary-hash}}"}[2m]))
```

## 🔍 Environment Health Monitoring

### **🏥 Health Check Endpoints**
```python
# Multi-layer Health Check Implementation
from dataclasses import dataclass
from typing import Dict, List, Optional
from enum import Enum
import asyncio
import aiohttp
import asyncpg
import aioredis
from datetime import datetime

class HealthStatus(Enum):
    HEALTHY = "healthy"
    DEGRADED = "degraded" 
    UNHEALTHY = "unhealthy"

@dataclass
class ComponentHealth:
    name: str
    status: HealthStatus
    response_time_ms: Optional[float] = None
    error_message: Optional[str] = None
    last_checked: datetime = None
    metadata: Dict = None

class HealthChecker:
    def __init__(self, config: AppConfig):
        self.config = config
        self.checks = {
            'database': self._check_database,
            'redis': self._check_redis,
            'external_api': self._check_external_api,
            'disk_space': self._check_disk_space,
            'memory': self._check_memory,
        }
    
    async def check_all(self) -> Dict[str, ComponentHealth]:
        """Run all health checks concurrently"""
        tasks = []
        for name, check_func in self.checks.items():
            tasks.append(self._run_check(name, check_func))
        
        results = await asyncio.gather(*tasks, return_exceptions=True)
        
        health_report = {}
        for result in results:
            if isinstance(result, ComponentHealth):
                health_report[result.name] = result
            else:
                # Handle exceptions
                health_report['unknown'] = ComponentHealth(
                    name='unknown',
                    status=HealthStatus.UNHEALTHY,
                    error_message=str(result),
                    last_checked=datetime.utcnow()
                )
        
        return health_report
    
    async def _run_check(self, name: str, check_func) -> ComponentHealth:
        """Run individual health check with timing"""
        start_time = datetime.utcnow()
        
        try:
            await check_func()
            end_time = datetime.utcnow()
            response_time = (end_time - start_time).total_seconds() * 1000
            
            return ComponentHealth(
                name=name,
                status=HealthStatus.HEALTHY,
                response_time_ms=response_time,
                last_checked=end_time
            )
            
        except Exception as e:
            end_time = datetime.utcnow()
            response_time = (end_time - start_time).total_seconds() * 1000
            
            return ComponentHealth(
                name=name,
                status=HealthStatus.UNHEALTHY,
                response_time_ms=response_time,
                error_message=str(e),
                last_checked=end_time
            )
    
    async def _check_database(self):
        """Check database connectivity and performance"""
        conn = await asyncpg.connect(
            host=self.config.database.host,
            port=self.config.database.port,
            user=self.config.database.username,
            password=self.config.database.password,
            database=self.config.database.database,
            command_timeout=5
        )
        
        # Test query
        result = await conn.fetchval('SELECT 1')
        if result != 1:
            raise Exception("Database query returned unexpected result")
            
        await conn.close()
    
    async def _check_redis(self):
        """Check Redis connectivity"""
        redis = aioredis.from_url(
            f"redis://{self.config.redis.host}:{self.config.redis.port}",
            password=self.config.redis.password,
            socket_timeout=5
        )
        
        # Test ping
        pong = await redis.ping()
        if not pong:
            raise Exception("Redis ping failed")
            
        await redis.close()
    
    async def _check_external_api(self):
        """Check external API dependencies"""
        timeout = aiohttp.ClientTimeout(total=10)
        
        async with aiohttp.ClientSession(timeout=timeout) as session:
            async with session.get(f"{self.config.api_base_url}/health") as response:
                if response.status != 200:
                    raise Exception(f"External API returned {response.status}")
                    
                data = await response.json()
                if data.get('status') != 'ok':
                    raise Exception("External API reports unhealthy status")

# Health Check API Endpoint
from fastapi import FastAPI, Response, status
from fastapi.responses import JSONResponse

app = FastAPI()

@app.get("/health/live")
async def liveness_check():
    """Kubernetes liveness probe - basic app availability"""
    return {"status": "alive", "timestamp": datetime.utcnow().isoformat()}

@app.get("/health/ready") 
async def readiness_check():
    """Kubernetes readiness probe - full system health"""
    health_checker = HealthChecker(AppConfig.load_from_environment())
    health_report = await health_checker.check_all()
    
    # Determine overall status
    overall_status = HealthStatus.HEALTHY
    for component in health_report.values():
        if component.status == HealthStatus.UNHEALTHY:
            overall_status = HealthStatus.UNHEALTHY
            break
        elif component.status == HealthStatus.DEGRADED:
            overall_status = HealthStatus.DEGRADED
    
    response_data = {
        "status": overall_status.value,
        "timestamp": datetime.utcnow().isoformat(),
        "components": {
            name: {
                "status": comp.status.value,
                "response_time_ms": comp.response_time_ms,
                "error": comp.error_message,
                "last_checked": comp.last_checked.isoformat() if comp.last_checked else None
            }
            for name, comp in health_report.items()
        }
    }
    
    # Return appropriate HTTP status
    if overall_status == HealthStatus.UNHEALTHY:
        return JSONResponse(
            content=response_data, 
            status_code=status.HTTP_503_SERVICE_UNAVAILABLE
        )
    elif overall_status == HealthStatus.DEGRADED:
        return JSONResponse(
            content=response_data,
            status_code=status.HTTP_200_OK,
            headers={"X-Health-Warning": "degraded"}
        )
    else:
        return JSONResponse(content=response_data, status_code=status.HTTP_200_OK)
```

## 🎯 Feature Flags & Environment Controls

### **🚩 Feature Flag Implementation**
```csharp
// C# Feature Flag Service
using Microsoft.Extensions.Configuration;
using System.Collections.Generic;

public interface IFeatureFlagService
{
    Task<bool> IsEnabledAsync(string featureName, string? userId = null);
    Task<T> GetVariantAsync<T>(string featureName, T defaultValue, string? userId = null);
    Task<Dictionary<string, bool>> GetAllFlagsAsync(string? userId = null);
}

public class FeatureFlagService : IFeatureFlagService
{
    private readonly IConfiguration _configuration;
    private readonly ILogger<FeatureFlagService> _logger;
    private readonly string _environment;
    
    // Feature flag providers (LaunchDarkly, Split.io, custom)
    private readonly Dictionary<string, IFeatureFlagProvider> _providers;
    
    public FeatureFlagService(
        IConfiguration configuration,
        ILogger<FeatureFlagService> logger)
    {
        _configuration = configuration;
        _logger = logger;
        _environment = Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT") ?? "Development";
        
        _providers = InitializeProviders();
    }
    
    public async Task<bool> IsEnabledAsync(string featureName, string? userId = null)
    {
        try
        {
            // Check environment-specific overrides first
            var environmentOverride = GetEnvironmentOverride(featureName);
            if (environmentOverride.HasValue)
            {
                _logger.LogDebug("Feature {FeatureName} overridden by environment: {Value}", 
                    featureName, environmentOverride.Value);
                return environmentOverride.Value;
            }
            
            // Check feature flag providers
            foreach (var provider in _providers.Values)
            {
                var result = await provider.IsEnabledAsync(featureName, userId);
                if (result.HasValue)
                {
                    _logger.LogDebug("Feature {FeatureName} resolved by {Provider}: {Value}",
                        featureName, provider.GetType().Name, result.Value);
                    return result.Value;
                }
            }
            
            // Default to false if no provider has the flag
            _logger.LogWarning("Feature {FeatureName} not found in any provider, defaulting to false", 
                featureName);
            return false;
            
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error checking feature flag {FeatureName}, defaulting to false", 
                featureName);
            return false;
        }
    }
    
    private bool? GetEnvironmentOverride(string featureName)
    {
        // Allow environment-specific overrides
        var environmentOverrides = new Dictionary<string, Dictionary<string, bool>>
        {
            ["Development"] = new()
            {
                ["beta_features"] = true,
                ["debug_mode"] = true,
                ["strict_validation"] = false
            },
            ["Staging"] = new()
            {
                ["beta_features"] = true,
                ["load_testing"] = true,
                ["analytics_sampling"] = true
            },
            ["Production"] = new()
            {
                ["beta_features"] = false,
                ["debug_mode"] = false,
                ["strict_validation"] = true
            }
        };
        
        if (environmentOverrides.TryGetValue(_environment, out var overrides))
        {
            overrides.TryGetValue(featureName, out var value);
            return value ? (bool?)value : null;
        }
        
        return null;
    }
}

// Usage in controller
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    private readonly IUserService _userService;
    private readonly IFeatureFlagService _featureFlags;
    
    public UsersController(IUserService userService, IFeatureFlagService featureFlags)
    {
        _userService = userService;
        _featureFlags = featureFlags;
    }
    
    [HttpGet]
    public async Task<IActionResult> GetUsers()
    {
        var users = await _userService.GetUsersAsync();
        
        // Feature-flag controlled enhancements
        if (await _featureFlags.IsEnabledAsync("enhanced_user_profiles"))
        {
            // Load additional user profile data
            users = await _userService.EnrichUserProfiles(users);
        }
        
        if (await _featureFlags.IsEnabledAsync("user_analytics"))
        {
            // Add analytics data
            foreach (var user in users)
            {
                user.Metadata["last_activity"] = await _userService.GetLastActivity(user.Id);
            }
        }
        
        return Ok(users);
    }
}
```

---

**🏗️ Regra de Ouro**: Paridade entre ambientes é fundamental - diferenças de configuração são fonte #1 de bugs em produção.

**🔐 Lembrete**: Secrets nunca devem estar em código - sempre usar secret managers apropriados para cada ambiente.

**🔄 Versão**: 1.0  
**📅 Última atualização**: 2024-12-19  
**👨‍💻 Criado por**: Universal AI-Powered Development Rules System
