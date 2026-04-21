---
type: skill
name: test-environment-management
description: Framework for managing test environments, configuration, data provisioning, isolation, and lifecycle automation.
origin: ECC
---

# Test Environment Management

Use this skill to design, provision, configure, isolate, monitor, and retire test environments that reflect production behavior without compromising reliability or safety.

## When to use this skill

- Defining environment tiers for development, staging, integration, end-to-end, or performance testing
- Standardizing configuration, secrets handling, and data provisioning across environments
- Automating lifecycle tasks such as setup, reset, teardown, and drift detection
- Improving environment isolation, observability, and reproducibility in CI/CD pipelines

## Core areas

- Environment configuration and hierarchy
- Test data provisioning and reset strategies
- Isolation and parallel test execution
- CI workflow integration
- Cloud provisioning and ephemeral environments
- Monitoring, health checks, and incident response

## Recommended reference file split

Keep this primary skill file concise and move detailed reference material into companion files such as:

- `references/configuration.md`
- `references/test-data.md`
- `references/isolation.md`
- `references/ci-workflow.md`
- `references/cloud-provisioning.md`
- `references/monitoring.md`

## Quick guidance

1. Start from a shared baseline configuration.
2. Apply environment-specific overrides and secrets securely.
3. Provision deterministic test data and clear teardown rules.
4. Isolate tests by tenant, namespace, account, or ephemeral stack.
5. Add health checks, observability, and drift detection.
6. Prefer automation for creation, reset, and cleanup.

## Notes

Detailed examples, large YAML samples, CI workflows, cloud provisioning guidance, and monitoring runbooks should be stored in the reference files listed above rather than embedded in the primary skill document.
  ssl_required: true
  cors:
    allowed_origins: ["${FRONTEND_URL}"]
    allowed_methods: ["GET", "POST", "PUT", "DELETE"]

# Environment-specific override
# environments/staging/overrides.yaml
app:
  environment: "staging"
  feature_flags:
    new_payment_gateway: true
    enhanced_reporting: false

api:
  base_url: "https://api.staging.example.com"
  timeout: 45000

logging:
  level: "debug"
```

## Test Data Strategy

### Data Provisioning Patterns

**Clean Slate Pattern**:
```python
# tests/conftest.py
import pytest
from database import DatabaseClient

def setup_clean_database():
    """Reset database to known state before test suite"""
    db = DatabaseClient()
    db.drop_all_collections()
    db.create_collections()
    db.load_fixtures('fixtures/common.yaml')
    db.load_fixtures(f'fixtures/{test_env}.yaml')
    return db

@ pytest.fixture(scope="session")
def database():
    return setup_clean_database()
```

**Data Seeding Service**:
```javascript
// services/data-seeder.js
const faker = require('faker');
const axios = require('axios');

class DataSeeder {
  constructor(config) {
    this.apiClient = axios.create({
      baseURL: config.apiBaseURL,
      headers: { 'Authorization': `Bearer ${config.accessToken}` }
    });
    this.config = config;
  }

  async seedUsers(count) {
    const users = [];
    for (let i = 0; i < count; i++) {
      const userData = {
        email: faker.internet.email(),
        name: faker.name.findName(),
        age: faker.datatype.number({ min: 18, max: 80 }),
        status: ['active', 'inactive', 'pending'][
          faker.datatype.number({ min: 0, max: 2 })
        ]
      };
      
      const response = await this.apiClient.post('/users', userData);
      users.push(response.data);
    }
    return users;
  }

  async seedOrders(userId, count) {
    const orders = [];
    for (let i = 0; i < count; i++) {
      const orderData = {
        user_id: userId,
        items: this.generateOrderItems(),
        total: faker.commerce.price(),
        status: ['pending', 'confirmed', 'shipped', 'delivered']
          [faker.datatype.number({ min: 0, max: 3 })]
      };
      
      const response = await this.apiClient.post('/orders', orderData);
      orders.push(response.data);
    }
    return orders;
  }
}

module.exports = DataSeeder;
```

## Test Isolation

### Containerized Environment Spinning

```dockerfile
# Dockerfile.test
FROM node:18-alpine

# Install testing tools
RUN apk add --no-cache \
    chromium \
    firefox \
    tzdata \
    unzip

# Install Playwright browsers
RUN npm install -g @playwright/test && \
    npx playwright install --with-deps chromium firefox

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .

# Create non-root user
RUN addgroup -g 1001 -S playwright && \
    adduser -u 1001 -S playwright -G playwright
USER playwright

CMD ["npm", "run", "test:e2e"]
```

```yaml
# docker-compose.test.yml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile.test
    environment:
      - NODE_ENV=test
      - DATABASE_URL=mongodb://mongo:27017/testdb
      - REDIS_URL=redis://redis:6379
      - API_BASE_URL=http://api:3000
    depends_on:
      - mongo
      - redis
      - api
    networks:
      - test-network

  api:
    image: myapp-api:staging
    environment:
      - MONGODB_URI=mongodb://mongo:27017/staging
      - REDIS_URL=redis://redis:6379
    ports:
      - "3000:3000"
    networks:
      - test-network

  mongo:
    image: mongo:6.0
    volumes:
      - mongo-data:/data/db
    networks:
      - test-network

  redis:
    image: redis:7.0-alpine
    networks:
      - test-network

  proxy:
    image: nginx:alpine
    volumes:
      - ./nginx.test.conf:/etc/nginx/nginx.conf
    ports:
      - "8080:8080"
    depends_on:
      - app
    networks:
      - test-network

volumes:
  mongo-data:

networks:
  test-network:
    driver: bridge
```

### Test-Specific Configuration

```typescript
// utils/test-environment.ts
import fs from 'fs';
import yaml from 'js-yaml';
import { merge } from 'lodash';

interface TestEnvironmentConfig {
  envType: 'integration' | 'e2e' | 'performance';
  timeout: number;
  retries: number;
  headless: boolean;
  captureLogs: boolean;
  videoRecording: boolean;
}

class TestEnvironment {
  private static instance: TestEnvironment;
  private config: any;
  private envConfig: TestEnvironmentConfig;

  private constructor() {
    this.loadConfiguration();
    this.setupEnvironment();
  }

  public static getInstance(): TestEnvironment {
    if (!TestEnvironment.instance) {
      TestEnvironment.instance = new TestEnvironment();
    }
    return TestEnvironment.instance;
  }

  private loadConfiguration() {
    // Load common configuration
    const commonConfig = yaml.load(
      fs.readFileSync('environments/common/config.yaml', 'utf8'));
    
    // Load environment-specific configuration
    const env = process.env.TEST_ENV || 'development';
    const envConfig = yaml.load(
      fs.readFileSync(`environments/${env}/base.yaml`, 'utf8'));
    
    // Load test-specific configuration
    const testType = process.env.TEST_TYPE || 'e2e';
    const testConfig = yaml.load(
      fs.readFileSync(`environments/test-specific/${testType}.yaml`, 'utf8'));
    
    // Deep merge configurations
    this.config = merge({}, commonConfig, envConfig, testConfig);
  }

  private setupEnvironment() {
    // Set environment variables
    process.env.API_BASE_URL = this.config.api.base_url;
    process.env.LOG_LEVEL = this.config.logging.level;
    
    // Configure test framework
    this.envConfig = {
      envType: process.env.TEST_TYPE as any,
      timeout: this.config.api.timeout,
      retries: this.config.api.retry_attempts,
      headless: process.env.HEADLESS !== 'false',
      captureLogs: this.config.logging.audit_trail,
      videoRecording: process.env.VIDEO_RECORDING === 'true'
    };
  }

  public getConfig(): any {
    return this.config;
  }

  public getEnvConfig(): TestEnvironmentConfig {
    return this.envConfig;
  }

  public async setupTestDatabase(): Promise<void> {
    // Setup database state specific to test
    console.log(`Setting up database for ${this.envConfig.envType} tests`);
    // Implementation details...
  }

  public async teardown(): Promise<void> {
    // Cleanup resources
    console.log('Tearing down test environment');
    // Implementation details...
  }
}

export default TestEnvironment;
```

## Environment Lifecycle Automation

### CI/CD Pipeline Integration

```yaml
# .github/workflows/test-environment.yml
name: Test Environment Management

on:
  workflow_dispatch:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  setup-test-environment:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        test-type: [integration, e2e, performance]
    environment: ${{ matrix.test-type }}

    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '18'
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Decrypt secrets
      run: |
        echo "${{ secrets.ENV_SECRETS }}" > secrets.enc
        openssl aes-256-cbc -d -in secrets.enc -out environments/secrets.yaml -k "$SECRETS_KEY"
      env:
        SECRETS_KEY: ${{ secrets.SECRETS_KEY }}

    - name: Start test environment
      run: |
        export TEST_ENV=${{ matrix.test-type }}
        export TEST_TYPE=${{ matrix.test-type }}
        docker-compose -f docker-compose.test.yml up -d

    - name: Wait for services to be ready
      run: |
        # Wait for API to be responsive
        timeout 120 bash -c 'until curl -s http://localhost:3000/health; do sleep 5; done' || exit 1
        
        # Wait for database to be ready
        timeout 60 bash -c 'until docker exec mongo mongosh --eval "db.adminCommand('ping')'"; do sleep 5; done' || exit 1

    - name: Run data provisioning
      run: |
        export TEST_ENV=${{ matrix.test-type }}
        npm run data:seed -- --count=50 --type=users
        npm run data:seed -- --count=100 --type=orders

    - name: Run tests
      run: |
        export TEST_ENV=${{ matrix.test-type }}
        export VIDEO_RECORDING=true
        npm run test:${{ matrix.test-type }}

    - name: Collect artifacts
      uses: actions/upload-artifact@v4
      if: always()
      with:
        name: test-results-${{ matrix.test-type }}
        path: |
          test-results/
          videos/
          logs/
        retention-days: 30

    - name: Teardown environment
      if: always()
      run: |
        docker-compose -f docker-compose.test.yml down
        docker system prune -f
```

### Dynamic Environment Provisioning

```python
# scripts/provision-environment.py
import boto3
import yaml
import subprocess
import time
from typing import Dict, Any

class CloudEnvironmentProvisioner:
    def __init__(self, config_file: str):
        self.config = self.load_config(config_file)
        self.ec2 = boto3.client('ec2')
        self.rds = boto3.client('rds')
        self.elasticache = boto3.client('elasticache')

    def load_config(self, config_file: str) -> Dict[str, Any]:
        with open(config_file, 'r') as f:
            return yaml.safe_load(f)

    def create_vpc(self, name: str) -> str:
        """Create isolated VPC for test environment"""
        vpc = self.ec2.create_vpc(
            CidrBlock='10.0.0.0/16',
            TagSpecifications=[
                {
                    'ResourceType': 'vpc',
                    'Tags': [
                        {'Key': 'Name', 'Value': name},
                        {'Key': 'Environment', 'Value': 'test'}
                    ]
                }
            ]
        )
        vpc_id = vpc['Vpc']['VpcId']
        
        # Enable DNS hostnames
        self.ec2.modify_vpc_attribute(
            VpcId=vpc_id,
            EnableDnsHostnames={'Value': True}
        )
        
        return vpc_id

    def create_test_environment(self, env_name: str, env_type: str) -> Dict[str, str]:
        """Provision complete test environment in cloud"""
        print(f"Provisioning {env_type} environment: {env_name}")
        
        # Create VPC
        vpc_id = self.create_vpc(env_name)
        
        # Create subnets
        public_subnet = self.ec2.create_subnet(
            VpcId=vpc_id,
            CidrBlock='10.0.1.0/24',
            AvailabilityZone='us-west-2a'
        )
        
        private_subnet = self.ec2.create_subnet(
            VpcId=vpc_id,
            CidrBlock='10.0.2.0/24',
            AvailabilityZone='us-west-2a'
        )
        
        # Create security groups
        api_sg = self.create_security_group(
            vpc_id, f"{env_name}-api-sg", "API Security Group"
        )
        
        db_sg = self.create_security_group(
            vpc_id, f"{env_name}-db-sg", "Database Security Group"
        )
        
        # Authorize security group rules
        self.ec2.authorize_security_group_ingress(
            GroupId=api_sg,
            IpPermissions=[
                {
                    'IpProtocol': 'tcp',
                    'FromPort': 3000,
                    'ToPort': 3000,
                    'IpRanges': [{'CidrIp': '0.0.0.0/0'}]
                }
            ]
        )
        
        self.ec2.authorize_security_group_ingress(
            GroupId=db_sg,
            IpPermissions=[
                {
                    'IpProtocol': 'tcp',
                    'FromPort': 27017,
                    'ToPort': 27017,
                    'UserIdGroupPairs': [{'GroupId': api_sg}]
                }
            ]
        )
        
        # Create RDS MongoDB instance
        db_instance = self.rds.create_db_instance(
            DBInstanceIdentifier=f"{env_name.lower()}-db",
            DBInstanceClass='db.t3.small',
            Engine='mongodb',
            MasterUsername=self.config['database']['username'],
            MasterUserPassword=self.config['database']['password'],
            AllocatedStorage=20,
            VpcSecurityGroupIds=[db_sg],
            DBSubnetGroupName=self.create_db_subnet_group(vpc_id, private_subnet['Subnet']['SubnetId'])
        )
        
        # Create ElastiCache Redis cluster
        cache_cluster = self.elasticache.create_cache_cluster(
            CacheClusterId=f"{env_name.lower()}-cache",
            NumCacheNodes=1,
            CacheNodeType='cache.t3.micro',
            Engine='redis',
            VpcSecurityGroupIds=[api_sg],
            SubnetGroupName=self.create_cache_subnet_group(vpc_id, private_subnet['Subnet']['SubnetId'])
        )
        
        # Wait for resources to be available
        print("Waiting for database to be available...")
        waiter = self.rds.get_waiter('db_instance_available')
        waiter.wait(DBInstanceIdentifier=f"{env_name.lower()}-db")
        
        # Deploy application using ECS or EC2
        app_endpoint = self.deploy_application(
            env_name, api_sg, public_subnet['Subnet']['SubnetId']
        )
        
        # Return environment details
        return {
            'environment_name': env_name,
            'vpc_id': vpc_id,
            'api_endpoint': app_endpoint,
            'database_endpoint': db_instance['DBInstance']['Endpoint']['Address'],
            'cache_endpoint': cache_cluster['CacheCluster']['ConfigurationEndpoint']['Address'],
            'status': 'provisioned'
        }
    
    def create_security_group(self, vpc_id: str, name: str, description: str) -> str:
        sg = self.ec2.create_security_group(
            GroupName=name,
            Description=description,
            VpcId=vpc_id
        )
        return sg['GroupId']
    
    def create_db_subnet_group(self, vpc_id: str, subnet_id: str) -> str:
        group_name = f"{vpc_id}-db-subnet"
        self.rds.create_db_subnet_group(
            DBSubnetGroupName=group_name,
            DBSubnetGroupDescription='DB Subnet Group',
            SubnetIds=[subnet_id]
        )
        return group_name
    
    def create_cache_subnet_group(self, vpc_id: str, subnet_id: str) -> str:
        group_name = f"{vpc_id}-cache-subnet"
        self.elasticache.create_cache_subnet_group(
            CacheSubnetGroupName=group_name,
            CacheSubnetGroupDescription='Cache Subnet Group',
            SubnetIds=[subnet_id]
        )
        return group_name
    
    def deploy_application(self, env_name: str, security_group: str, subnet_id: str) -> str:
        # Deploy application using ECS, EC2, or EKS
        # Return application endpoint
        pass

# Usage
if __name__ == "__main__":
    provisioner = CloudEnvironmentProvisioner('config/environment-config.yaml')
    env_details = provisioner.create_test_environment('e2e-test-001', 'e2e')
    print(f"Environment provisioned: {env_details['api_endpoint']}")
    
    # Save environment details for teardown
    with open(f"environments/{env_details['environment_name']}.json", 'w') as f:
        json.dump(env_details, f, indent=2)
```

## Environment Monitoring and Telemetry

### Health Checks and Readiness Probes

```yaml
# kubernetes/test-environment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-api
  namespace: test
spec:
  replicas: 2
  selector:
    matchLabels:
      app: test-api
  template:
    metadata:
      labels:
        app: test-api
    spec:
      containers:
      - name: api
        image: myapp-api:staging
        env:
        - name: NODE_ENV
          value: test
        - name: DATABASE_URL
          value: mongodb://test-db:27017/testdb
        ports:
        - containerPort: 3000
        readinessProbe:
          httpGet:
            path: /health/readiness
            port: 3000
          initialDelaySeconds: 10
          periodSeconds: 5
          timeoutSeconds: 3
          successThreshold: 1
          failureThreshold: 3
        livenessProbe:
          httpGet:
            path: /health/liveness
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          successThreshold: 1
          failureThreshold: 3
        startupProbe:
          httpGet:
            path: /health/startup
            port: 3000
          failureThreshold: 30
          periodSeconds: 10

---
apiVersion: v1
kind: Service
metadata:
  name: test-api
  namespace: test
spec:
  selector:
    app: test-api
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: ClusterIP
```

### Environment Teardown Strategy

```javascript
// utils/environment-teardown.js
const { exec } = require('child_process');
const fs = require('fs');
const path = require('path');

async function teardownEnvironment() {
  const envName = process.env.ENV_NAME || 'test';
  const deploymentType = process.env.DEPLOYMENT_TYPE || 'docker';
  
  console.log(`Tearing down ${envName} environment...`);
  
  try {
    if (deploymentType === 'docker') {
      // Docker Compose teardown
      await executeCommand('docker-compose -f docker-compose.test.yml down --remove-orphans');
      await executeCommand('docker network prune -f');
      await executeCommand('docker volume prune -f');
    } else if (deploymentType === 'kubernetes') {
      // Kubernetes teardown
      await executeCommand('kubectl delete -f kubernetes/test-environment.yaml --namespace test');
      await executeCommand('kubectl delete namespace test');
    } else if (deploymentType === 'cloud') {
      // Cloud infrastructure teardown
      const envDetailsPath = path.join('environments', `${envName}.json`);
      if (fs.existsSync(envDetailsPath)) {
        const envDetails = JSON.parse(fs.readFileSync(envDetailsPath, 'utf8'));
        await teardownCloudEnvironment(envDetails);
        fs.unlinkSync(envDetailsPath);
      }
    }
    
    // Cleanup local artifacts
    await executeCommand('rm -rf test-results/ videos/ logs/');
    
    console.log('Environment teardown completed successfully');
    
  } catch (error) {
    console.error('Error during environment teardown:', error);
    throw error;
  }
}

function executeCommand(command) {
  return new Promise((resolve, reject) => {
    exec(command, (error, stdout, stderr) => {
      if (error) {
        reject(error);
        return;
      }
      if (stderr) {
        console.error('Command stderr:', stderr);
      }
      resolve(stdout);
    });
  });
}

async function teardownCloudEnvironment(envDetails) {
  // Implementation for tearing down cloud resources
  // Remove VPC, EC2 instances, RDS, ElastiCache, etc.
  console.log('Tearing down cloud environment...');
  // Cloud provider specific teardown logic
}

// Register cleanup on process exit
process.on('SIGINT', async () => {
  await teardownEnvironment();
  process.exit(0);
});

process.on('SIGTERM', async () => {
  await teardownEnvironment();
  process.exit(0);
});

module.exports = { teardownEnvironment };
