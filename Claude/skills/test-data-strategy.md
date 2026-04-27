---
type: skill
name: test-data-strategy
description: Comprehensive framework for test data management including provisioning, anonymization, synthetic data generation, and lifecycle management.
origin: ECC
---

# Test Data Strategy Framework

Strategic approach to managing test data across the development lifecycle, ensuring data quality, privacy compliance, and test reliability.

## Data Provisioning Patterns

### Synthetic Data Generation

```python
# utils/synthetic_data.py
import random
import string
from datetime import datetime, timedelta
from typing import Dict, List, Any
import pandas as pd
import numpy as np

class SyntheticDataGenerator:
    """Generate realistic synthetic data for testing"""
    
    def __init__(self):
        self.first_names = ['James', 'Mary', 'John', 'Patricia', 'Robert', 'Jennifer']
        self.last_names = ['Smith', 'Johnson', 'Williams', 'Jones', 'Brown', 'Davis']
        self.domains = ['example.com', 'test.org', 'demo.net', 'qa.co']
        
    def generate_email(self, first_name: str = None, last_name: str = None) -> str:
        """Generate realistic email address"""
        if not first_name:
            first_name = random.choice(self.first_names)
        if not last_name:
            last_name = random.choice(self.last_names)
            
        separator = random.choice(['.', '_', ''])
        domain = random.choice(self.domains)
        
        # Create variation in email format
        variations = [
            f"{first_name.lower()}{separator}{last_name.lower()}@{domain}",
            f"{first_name[0].lower()}{last_name.lower()}@{domain}",
            f"{first_name.lower()}{random.randint(10,99)}@{domain}"
        ]
        
        return random.choice(variations)
    
    def generate_phone(self) -> str:
        """Generate realistic phone number"""
        return f"+1-{random.randint(200,999)}-{random.randint(100,999)}-{random.randint(1000,9999)}"
    
    def generate_address(self) -> Dict[str, str]:
        """Generate realistic address"""
        streets = ['Main St', 'Oak Ave', 'Pine Rd', 'Maple Dr', 'Cedar Ln']
        cities = ['Springfield', 'Greenville', 'Oakland', 'Portsmouth', 'Lakeside']
        states = ['CA', 'TX', 'NY', 'FL', 'WA']
        
        return {
            'street': f"{random.randint(100,999)} {random.choice(streets)}",
            'city': random.choice(cities),
            'state': random.choice(states),
            'zip_code': f"{random.randint(10000,99999)}"
        }
    
    def generate_user(self, user_type: str = 'standard') -> Dict[str, Any]:
        """Generate complete user profile"""
        first_name = random.choice(self.first_names)
        last_name = random.choice(self.last_names)
        
        user = {
            'first_name': first_name,
            'last_name': last_name,
            'email': self.generate_email(first_name, last_name),
            'phone': self.generate_phone(),
            'date_of_birth': (datetime.now() - timedelta(
                days=random.randint(365*18, 365*80)
            )).strftime('%Y-%m-%d'),
            'created_at': datetime.now().isoformat(),
            'updated_at': datetime.now().isoformat()
        }
        
        # Add user type specific attributes
        if user_type == 'premium':
            user.update({
                'membership_level': 'premium',
                'membership_since': (datetime.now() - timedelta(
                    days=random.randint(30, 365*3)
                )).strftime('%Y-%m-%d'),
                'loyalty_points': random.randint(100, 10000)
            })
        elif user_type == 'business':
            user.update({
                'company_name': f"{random.choice(['Tech', 'Global', 'Innovate', 'Solutions'])} {random.choice(['Inc', 'LLC', 'Co'])}",
                'business_type': random.choice(['corporation', 'llc', 'partnership']),
                'tax_id': f"{random.randint(10,99)}-{random.randint(1000000,9999999)}"
            })
        
        return user
    
    def generate_order(self, user_id: str, status: str = None) -> Dict[str, Any]:
        """Generate realistic order data"""
        products = [
            {'name': 'Premium Subscription', 'price': 99.99, 'category': 'software'},
            {'name': 'Enterprise License', 'price': 499.99, 'category': 'software'},
            {'name': 'Training Course', 'price': 299.99, 'category': 'education'},
            {'name': 'Consulting Package', 'price': 1999.99, 'category': 'services'}
        ]
        
        # Random number of items (1-5)
        num_items = random.randint(1, 5)
        items = []
        total = 0.0
        
        for _ in range(num_items):
            product = random.choice(products)
            quantity = random.randint(1, 3)
            line_total = product['price'] * quantity
            total += line_total
            
            items.append({
                'product_id': f"PROD{random.randint(1000,9999)}",
                'name': product['name'],
                'quantity': quantity,
                'unit_price': product['price'],
                'line_total': round(line_total, 2)
            })
        
        # Randomize status if not provided
        if not status:
            status = random.choices(
                ['pending', 'confirmed', 'shipped', 'delivered', 'cancelled'],
                weights=[10, 40, 30, 15, 5]
            )[0]
        
        return {
            'order_id': f"ORD{datetime.now().strftime('%Y%m')}{random.randint(1000,9999)}",
            'user_id': user_id,
            'items': items,
            'total_amount': round(total, 2),
            'currency': 'USD',
            'status': status,
            'created_at': (datetime.now() - timedelta(
                hours=random.randint(0, 72),
                minutes=random.randint(0, 59)
            )).isoformat(),
            'updated_at': datetime.now().isoformat()
        }
    
    def generate_data_frame(self, schema: Dict[str, str], count: int) -> pd.DataFrame:
        """Generate pandas DataFrame with specified schema"""
        data = {}
        
        for field, field_type in schema.items():
            if field_type == 'string':
                data[field] = [self._generate_random_string() for _ in range(count)]
            elif field_type == 'email':
                data[field] = [self.generate_email() for _ in range(count)]
            elif field_type == 'phone':
                data[field] = [self.generate_phone() for _ in range(count)]
            elif field_type == 'integer':
                data[field] = [random.randint(1, 1000) for _ in range(count)]
            elif field_type == 'float':
                data[field] = [round(random.uniform(1.0, 1000.0), 2) for _ in range(count)]
            elif field_type == 'date':
                start_date = datetime.now() - timedelta(days=365)
                data[field] = [
                    (start_date + timedelta(days=random.randint(0, 365))).strftime('%Y-%m-%d')
                    for _ in range(count)
                ]
            elif field_type == 'boolean':
                data[field] = [random.choice([True, False]) for _ in range(count)]
            
        return pd.DataFrame(data)
    
    def _generate_random_string(self, length: int = 8) -> str:
        """Generate random string of specified length"""
        return ''.join(random.choices(string.ascii_letters + string.digits, k=length))

# Usage example
if __name__ == "__main__":
    generator = SyntheticDataGenerator()
    
    # Generate 100 users
    users = [generator.generate_user('standard') for _ in range(100)]
    users += [generator.generate_user('premium') for _ in range(20)]
    users += [generator.generate_user('business') for _ in range(10)]
    
    # Generate orders for users
    orders = []
    for user in users:
        # Each user has 0-3 orders
        num_orders = random.randint(0, 3)
        for _ in range(num_orders):
            orders.append(generator.generate_order(user['email']))
    
    print(f"Generated {len(users)} users and {len(orders)} orders")
```

### Data Masking and Anonymization

```python
# utils/data_anonymizer.py
import re
import hashlib
from typing import Dict, Any, List
import pandas as pd

class DataAnonymizer:
    """Anonymize sensitive data for testing purposes"""
    
    def __init__(self, salt: str = "test_salt"):
        self.salt = salt
        
    def anonymize_email(self, email: str) -> str:
        """Hash email address while preserving domain"""
        if not email or '@' not in email:
            return email
            
        local_part, domain = email.split('@')
        # Hash the local part and append domain
        hashed = hashlib.sha256(f"{local_part}{self.salt}".encode()).hexdigest()[:8]
        return f"anon_{hashed}@{domain}"
    
    def anonymize_phone(self, phone: str) -> str:
        """Mask phone number, preserve country code and last 4 digits"""
        if not phone:
            return phone
            
        # Extract digits
        digits = re.sub(r'\D', '', phone)
        
        if len(digits) == 10:  # US number
            return f"+1-XXX-XXX-{digits[-4:] }"
        elif len(digits) > 10:
            country_code = digits[:-10]
            return f"+{country_code}-XXX-XXX-{digits[-4:] }"
        else:
            return "+X-XXX-XXX-XXXX"  # Generic mask
    
    def anonymize_address(self, address: Dict[str, str]) -> Dict[str, str]:
        """Anonymize address information"""
        if not address:
            return address
            
        # Keep city and state but randomize street and zip
        return {
            'street': f"{random.randint(100, 999)} Main St",
            'city': address.get('city', 'City'),
            'state': address.get('state', 'ST'),
            'zip_code': f"{random.randint(10000, 99999)}"
        }
    
    def anonymize_name(self, first_name: str, last_name: str) -> Dict[str, str]:
        """Anonymize person name"""
        # Use consistent hashing for same names
        first_hash = hashlib.md5(f"{first_name}{self.salt}".encode()).hexdigest()[0]
        last_hash = hashlib.md5(f"{last_name}{self.salt}".encode()).hexdigest()[0]
        
        # Map to generic names based on hash
        first_names = ['User', 'Client', 'Customer', 'Member', 'Account']
        last_names = ['One', 'Two', 'Three', 'Four', 'Five']
        
        return {
            'first_name': first_names[ord(first_hash) % len(first_names)],
            'last_name': last_names[ord(last_hash) % len(last_names)]
        }
    
    def anonymize_user(self, user: Dict[str, Any]) -> Dict[str, Any]:
        """Anonymize complete user profile"""
        if not user:
            return user
            
        # Create deep copy
        anonymized = user.copy()
        
        # Anonymize sensitive fields
        if 'email' in anonymized:
            anonymized['email'] = self.anonymize_email(anonymized['email'])
            
        if 'phone' in anonymized:
            anonymized['phone'] = self.anonymize_phone(anonymized['phone'])
            
        if 'first_name' in anonymized and 'last_name' in anonymized:
            names = self.anonymize_name(anonymized['first_name'], anonymized['last_name'])
            anonymized['first_name'] = names['first_name']
            anonymized['last_name'] = names['last_name']
            
        if 'address' in anonymized:
            anonymized['address'] = self.anonymize_address(anonymized['address'])
            
        # Remove or mask other sensitive fields
        sensitive_fields = ['ssn', 'national_id', 'credit_card', 'bank_account']
        for field in sensitive_fields:
            if field in anonymized:
                anonymized[field] = 'REDACTED'
                
        # Add anonymization metadata
        anonymized['anonymized_at'] = datetime.now().isoformat()
        anonymized['original_user_id'] = anonymized.get('user_id')
        anonymized['user_id'] = f"ANON_{hashlib.md5(f"{anonymized.get('email', '')}{self.salt}".encode()).hexdigest()[:12]}"
        
        return anonymized
    
    def anonymize_dataset(self, df: pd.DataFrame, config: Dict[str, str]) -> pd.DataFrame:
        """Anonymize pandas DataFrame according to configuration"""
        df_anon = df.copy()
        
        for column, method in config.items():
            if column not in df_anon.columns:
                continue
                
            if method == 'email':
                df_anon[column] = df_anon[column].apply(self.anonymize_email)
            elif method == 'phone':
                df_anon[column] = df_anon[column].apply(self.anonymize_phone)
            elif method == 'name':
                # For name columns, we need to handle first and last separately
                if 'first' in column.lower():
                    df_anon[column] = df_anon.apply(
                        lambda row: self.anonymize_name(row[column], row.get(column.replace('first', 'last'), ''))['first_name'], 
                        axis=1
                    )
                elif 'last' in column.lower():
                    df_anon[column] = df_anon.apply(
                        lambda row: self.anonymize_name(row.get(column.replace('last', 'first'), ''), row[column])['last_name'], 
                        axis=1
                    )
            elif method == 'address':
                # For address, we assume a JSON/Dict column
                df_anon[column] = df_anon[column].apply(self.anonymize_address)
            elif method == 'id':
                # For IDs, hash them
                df_anon[column] = df_anon[column].apply(
                    lambda x: f"ANON_{hashlib.md5(f"{x}{self.salt}".encode()).hexdigest()[:8]}" if pd.notna(x) else x
                )
            
        return df_anon
    
    def generate_test_subset(self, source_data: List[Dict[str, Any]], target_count: int, 
                           distribution: Dict[str, float] = None) -> List[Dict[str, Any]]:
        """Generate representative test subset from production data"""
        if distribution:
            # Stratified sampling based on distribution
            result = []
            for category, ratio in distribution.items():
                category_items = [item for item in source_data if item.get('category') == category]
                sample_count = int(target_count * ratio)
                result.extend(random.sample(category_items, min(sample_count, len(category_items))))
            return result
        else:
            # Random sampling
            return random.sample(source_data, min(target_count, len(source_data)))

# Usage example
if __name__ == "__main__":
    anonymizer = DataAnonymizer(salt="my_secret_salt")
    
    # Example user data
    user_data = {
        'user_id': 'USR12345',
        'first_name': 'John',
        'last_name': 'Smith',
        'email': 'john.smith@example.com',
        'phone': '+1-555-123-4567',
        'address': {
            'street': '123 Main St',
            'city': 'Springfield',
            'state': 'IL',
            'zip_code': '62701'
        },
        'ssn': '123-45-6789',
        'created_at': '2025-01-15T10:30:00Z'
    }
    
    anonymized_user = anonymizer.anonymize_user(user_data)
    print("Original:", user_data)
    print("Anonymized:", anonymized_user)
```

## Data Lifecycle Management

### Test Data Factory Pattern

```typescript
// factories/user.factory.ts
import { User, UserRole, UserStatus } from '../interfaces/user.interface';
import { faker } from '@faker-js/faker';

class UserFactory {
  // Default attributes
  private defaults = {
    role: UserRole.USER,
    status: UserStatus.ACTIVE,
    email_verified: false,
    created_at: new Date(),
    updated_at: new Date()
  };

  /**
   * Create a single user with optional overrides
   */
  public create(overrides: Partial<User> = {}): User {
    const user: User = {
      id: overrides.id || `user_${faker.string.uuid()}`,
      first_name: overrides.first_name || faker.person.firstName(),
      last_name: overrides.last_name || faker.person.lastName(),
      email: overrides.email || faker.internet.email(),
      role: overrides.role || this.defaults.role,
      status: overrides.status || this.defaults.status,
      email_verified: overrides.email_verified ?? this.defaults.email_verified,
      created_at: overrides.created_at || this.defaults.created_at,
      updated_at: overrides.updated_at || this.defaults.updated_at
    };

    // Apply specific scenarios
    if (overrides.scenario === 'verified') {
      user.email_verified = true;
    }

    if (overrides.scenario === 'admin') {
      user.role = UserRole.ADMIN;
    }

    return user;
  }

  /**
   * Create multiple users with the same attributes
   */
  public createMany(count: number, overrides: Partial<User> = {}): User[] {
    return Array.from({ length: count }, () => this.create(overrides));
  }

  /**
   * Create users with specific roles
   */
  public createAdmins(count: number = 1): User[] {
    return this.createMany(count, { 
      role: UserRole.ADMIN,
      scenario: 'admin' 
    });
  }

  public createModerators(count: number = 1): User[] {
    return this.createMany(count, { role: UserRole.MODERATOR });
  }

  public createUsers(count: number = 1): User[] {
    return this.createMany(count, { role: UserRole.USER });
  }

  /**
   * Create users with specific statuses
   */
  public createActiveUsers(count: number = 1): User[] {
    return this.createMany(count, { status: UserStatus.ACTIVE });
  }

  public createInactiveUsers(count: number = 1): User[] {
    return this.createMany(count, { status: UserStatus.INACTIVE });
  }

  public createPendingUsers(count: number = 1): User[] {
    return this.createMany(count, { status: UserStatus.PENDING });
  }

  /**
   * Create verified users
   */
  public createVerifiedUsers(count: number = 1): User[] {
    return this.createMany(count, { 
      email_verified: true,
      scenario: 'verified' 
    });
  }

  /**
   * Create users from specific regions
   */
  public createFromRegion(region: string, count: number = 1): User[] {
    const users = this.createMany(count);
    // In a real implementation, you might integrate with location data
    return users;
  }

  /**
   * Create users with specific attribute patterns
   */
  public createWithPattern(pattern: 'long-names' | 'short-names' | 'common-names', count: number = 1): User[] {
    const users: User[] = [];
    
    for (let i = 0; i < count; i++) {
      let firstName: string, lastName: string;
      
      switch (pattern) {
        case 'long-names':
          firstName = this.generateLongName();
          lastName = this.generateLongName();
          break;
        case 'short-names':
          firstName = this.generateShortName();
          lastName = this.generateShortName();
          break;
        case 'common-names':
        default:
          firstName = faker.person.commonFirstName();
          lastName = faker.person.commonLastName();
      }
      
      users.push(this.create({ firstName, lastName }));
    }
    
    return users;
  }

  private generateLongName(): string {
    // Generate longer names (8-15 characters)
    const length = faker.number.int({ min: 8, max: 15 });
    return faker.string.alpha({ length, casing: 'title' });
  }

  private generateShortName(): string {
    // Generate shorter names (2-4 characters)
    const length = faker.number.int({ min: 2, max: 4 });
    return faker.string.alpha({ length, casing: 'title' });
  }
}

// Export singleton instance
export const userFactory = new UserFactory();

// Usage examples
/*
import { userFactory } from './factories/user.factory';

// Create single user
const user = userFactory.create();
const admin = userFactory.create({ role: UserRole.ADMIN });

// Create multiple users
const users = userFactory.createMany(10);
const activeUsers = userFactory.createActiveUsers(5);

// Create specific scenarios
const verifiedUsers = userFactory.createVerifiedUsers(3);
const longNameUsers = userFactory.createWithPattern('long-names', 2);
*/
```

```typescript
// factories/order.factory.ts
import { Order, OrderItem, OrderStatus } from '../interfaces/order.interface';
import { Product } from '../interfaces/product.interface';
import { userFactory } from './user.factory';
import { faker } from '@faker-js/faker';

class OrderFactory {
  private defaults = {
    status: OrderStatus.PENDING,
    currency: 'USD',
    created_at: new Date(),
    updated_at: new Date()
  };

  // Predefined products for consistent test data
  private products: Product[] = [
    { 
      id: 'prod-software', 
      name: 'Software Subscription', 
      price: 99.99, 
      category: 'software' 
    },
    { 
      id: 'prod-hardware', 
      name: 'Hardware Device', 
      price: 299.99, 
      category: 'hardware' 
    },
    { 
      id: 'prod-service', 
      name: 'Consulting Service', 
      price: 499.99, 
      category: 'service' 
    }
  ];

  /**
   * Create a single order with optional overrides
   */
  public create(overrides: Partial<Order> = {}): Order {
    // Get or create user
    const userId = overrides.user_id || userFactory.create().id;
    
    // Create order items
    const items = this.createOrderItems(overrides.items_count || 1);
    
    // Calculate total
    const total = items.reduce((sum, item) => sum + item.line_total, 0);

    const order: Order = {
      id: overrides.id || `order_${faker.string.uuid()}`,
      user_id: userId,
      items: overrides.items || items,
      total_amount: overrides.total_amount || total,
      currency: overrides.currency || this.defaults.currency,
      status: overrides.status || this.defaults.status,
      created_at: overrides.created_at || this.defaults.created_at,
      updated_at: overrides.updated_at || this.defaults.updated_at
    };

    // Apply scenarios
    if (overrides.scenario === 'high-value') {
      order.total_amount = faker.number.float({ min: 1000, max: 5000, precision: 2 });
    }

    if (overrides.scenario === 'international') {
      order.currency = this.getRandomCurrency();
    }

    return order;
  }

  /**
   * Create multiple orders
   */
  public createMany(count: number, overrides: Partial<Order> = {}): Order[] {
    return Array.from({ length: count }, () => this.create(overrides));
  }

  /**
   * Create orders with specific statuses
   */
  public createPendingOrders(count: number = 1): Order[] {
    return this.createMany(count, { status: OrderStatus.PENDING });
  }

  public createConfirmedOrders(count: number = 1): Order[] {
    return this.createMany(count, { status: OrderStatus.CONFIRMED });
  }

  public createShippedOrders(count: number = 1): Order[] {
    return this.createMany(count, { status: OrderStatus.SHIPPED });
  }

  public createDeliveredOrders(count: number = 1): Order[] {
    return this.createMany(count, { status: OrderStatus.DELIVERED });
  }

  public createCancelledOrders(count: number = 1): Order[] {
    return this.createMany(count, { status: OrderStatus.CANCELLED });
  }

  /**
   * Create high-value orders
   */
  public createHighValueOrders(count: number = 1): Order[] {
    return this.createMany(count, { 
      scenario: 'high-value',
      status: OrderStatus.CONFIRMED 
    });
  }

  /**
   * Create international orders
   */
  public createInternationalOrders(count: number = 1): Order[] {
    return this.createMany(count, { 
      scenario: 'international',
      status: OrderStatus.CONFIRMED 
    });
  }

  /**
   * Create orders for specific users
   */
  public createForUser(userId: string, count: number = 1): Order[] {
    return this.createMany(count, { user_id: userId });
  }

  /**
   * Create orders with specific item counts
   */
  public createWithItemCount(itemCount: number, count: number = 1): Order[] {
    return this.createMany(count, { items_count: itemCount });
  }

  /**
   * Create orders from specific categories
   */
  public createFromCategory(category: string, count: number = 1): Order[] {
    const orders = this.createMany(count);
    // Modify items to match category
    orders.forEach(order => {
      order.items = this.createOrderItems(1, category);
      order.total_amount = order.items.reduce((sum, item) => sum + item.line_total, 0);
    });
    return orders;
  }

  /**
   * Create order items
   */
  private createOrderItems(count: number, category?: string): OrderItem[] {
    const items: OrderItem[] = [];
    
    for (let i = 0; i < count; i++) {
      // Filter products by category if specified
      const availableProducts = category 
        ? this.products.filter(p => p.category === category)
        : this.products;
      
      // Randomly select product
      const product = faker.helpers.arrayElement(availableProducts);
      
      // Random quantity (1-3)
      const quantity = faker.number.int({ min: 1, max: 3 });
      
      // Calculate line total
      const lineTotal = product.price * quantity;
      
      items.push({
        product_id: product.id,
        name: product.name,
        quantity,
        unit_price: product.price,
        line_total: Number(lineTotal.toFixed(2))
      });
    }
    
    return items;
  }

  /**
   * Get random currency for international orders
   */
  private getRandomCurrency(): string {
    return faker.helpers.arrayElement(['EUR', 'GBP', 'JPY', 'CAD', 'AUD']);
  }
}

// Export singleton instance
export const orderFactory = new OrderFactory();

// Usage examples
/*
import { orderFactory } from './factories/order.factory';

// Create single order
const order = orderFactory.create();

// Create multiple orders
const orders = orderFactory.createMany(5);

// Create specific scenarios
const highValueOrders = orderFactory.createHighValueOrders(3);
const internationalOrders = orderFactory.createInternationalOrders(2);

// Create orders with status variations
const pendingOrders = orderFactory.createPendingOrders(1);
const confirmedOrders = orderFactory.createConfirmedOrders(2);
const shippedOrders = orderFactory.createShippedOrders(1);
const deliveredOrders = orderFactory.createDeliveredOrders(1);
*/
```

## Data Quality Validation

### Schema Validation and Constraints

```python
# validators/data_validator.py
from typing import Dict, Any, List, Optional
import jsonschema
from jsonschema import validate
import pandas as pd
from datetime import datetime
import re

# JSON Schema definitions
USER_SCHEMA = {
    "type": "object",
    "required": ["user_id", "first_name", "last_name", "email", "created_at"],
    "properties": {
        "user_id": {"type": "string", "pattern": "^USR[0-9]{5}$"},
        "first_name": {"type": "string", "minLength": 1, "maxLength": 50},
        "last_name": {"type": "string", "minLength": 1, "maxLength": 50},
        "email": {"type": "string", "format": "email"},
        "phone": {"type": "string", "pattern": "^\\+?[1-9][0-9]{1,14}$"},
        "date_of_birth": {"type": "string", "format": "date"},
        "status": {
            "type": "string",
            "enum": ["active", "inactive", "pending", "suspended"]
        },
        "created_at": {"type": "string", "format": "date-time"},
        "updated_at": {"type": "string", "format": "date-time"}
    }
}

ORDER_SCHEMA = {
    "type": "object",
    "required": ["order_id", "user_id", "items", "total_amount", "status", "created_at"],
    "properties": {
        "order_id": {"type": "string", "pattern": "^ORD[0-9]{8}[0-9]{4}$"},
        "user_id": {"type": "string", "pattern": "^USR[0-9]{5}$"},
        "items": {
            "type": "array",
            "minItems": 1,
            "items": {
                "type": "object",
                "required": ["product_id", "quantity", "unit_price", "line_total"],
                "properties": {
                    "product_id": {"type": "string"},
                    "name": {"type": "string"},
                    "quantity": {"type": "integer", "minimum": 1},
                    "unit_price": {"type": "number", "minimum": 0},
                    "line_total": {"type": "number", "minimum": 0}
                }
            }
        },
        "total_amount": {"type": "number", "minimum": 0},
        "currency": {"type": "string", "maxLength": 3, "default": "USD"},
        "status": {
            "type": "string",
            "enum": ["pending", "confirmed", "shipped", "delivered", "cancelled"]
        },
        "created_at": {"type": "string", "format": "date-time"},
        "updated_at": {"type": "string", "format": "date-time"}
    }
}

class DataValidator:
    """Validate data against defined schemas and business rules"""
    
    def __init__(self):
        self.schemas = {
            'user': USER_SCHEMA,
            'order': ORDER_SCHEMA
        }
    
    def validate_schema(self, data: Dict[str, Any], schema_type: str) -> Dict[str, Any]:
        """Validate data against JSON schema"""
        if schema_type not in self.schemas:
            raise ValueError(f"Unknown schema type: {schema_type}")
        
        try:
            validate(instance=data, schema=self.schemas[schema_type])
            return {"valid": True, "errors": []}
        except jsonschema.exceptions.ValidationError as e:
            return {
                "valid": False,
                "errors": [str(e)]
            }
    
    def validate_user(self, user: Dict[str, Any]) -> Dict[str, Any]:
        """Validate user data with business rules"""
        # Schema validation
        schema_result = self.validate_schema(user, 'user')
        if not schema_result['valid']:
            return schema_result
        
        errors = []
        
        # Business rule validation
        now = datetime.now()
        
        # Validate date of birth
        if 'date_of_birth' in user and user['date_of_birth']:
            dob = datetime.strptime(user['date_of_birth'], '%Y-%m-%d')
            age = (now - dob).days // 365
            if age < 13:
                errors.append("User must be at least 13 years old")
            if age > 120:
                errors.append("User age seems unrealistic")
        
        # Validate email format beyond JSON schema
        if 'email' in user:
            # Check for disposable email domains
            disposable_domains = ['tempmail.com', 'mailinator.com', '10minutemail.com']
            domain = user['email'].split('@')[1].lower()
            if domain in disposable_domains:
                errors.append("Disposable email domains are not allowed")
        
        # Validate phone number format
        if 'phone' in user and user['phone']:
            digits = re.sub(r'\D', '', user['phone'])
            if len(digits) < 10:
                errors.append("Phone number must have at least 10 digits")
        
        # Validate name formatting
        if 'first_name' in user and user['first_name']:
            if not user['first_name'][0].isupper():
                errors.append("First name should be capitalized")
        
        if 'last_name' in user and user['last_name']:
            if not user['last_name'][0].isupper():
                errors.append("Last name should be capitalized")
        
        return {
            "valid": len(errors) == 0,
            "errors": errors
        }
    
    def validate_order(self, order: Dict[str, Any]) -> Dict[str, Any]:
        """Validate order data with business rules"""
        # Schema validation
        schema_result = self.validate_schema(order, 'order')
        if not schema_result['valid']:
            return schema_result
        
        errors = []
        
        # Business rule validation
        
        # Validate total amount matches items sum
        expected_total = sum(item['line_total'] for item in order['items'])
        if abs(order['total_amount'] - expected_total) > 0.01:  # Allow for floating point errors
            errors.append(f"Total amount {order['total_amount']} does not match sum of items {expected_total}")
        
        # Validate item quantities and prices
        for i, item in enumerate(order['items']):
            if item['quantity'] <= 0:
                errors.append(f"Item {i} has invalid quantity {item['quantity']}")
            
            if item['unit_price'] < 0:
                errors.append(f"Item {i} has negative unit price {item['unit_price']}")
            
            expected_line_total = item['quantity'] * item['unit_price']
            if abs(item['line_total'] - expected_line_total) > 0.01:
                errors.append(f"Item {i} line total {item['line_total']} does not match quantity * unit price {expected_line_total}")
        
        # Validate status transitions
        if 'status' in order and 'previous_status' in order:
            valid_transitions = {
                'pending': ['confirmed', 'cancelled'],
                'confirmed': ['shipped', 'cancelled'],
                'shipped': ['delivered', 'cancelled'],
                'delivered': [],
                'cancelled': []
            }
            
            if order['previous_status'] not in valid_transitions:
                errors.append(f"Unknown previous status {order['previous_status']}")
            elif order['status'] not in valid_transitions[order['previous_status']]:
                errors.append(f"Invalid status transition from {order['previous_status']} to {order['status']}")
        
        # Validate timestamps
        if 'created_at' in order and 'updated_at' in order:
            created = datetime.fromisoformat(order['created_at'].replace('Z', '+00:00'))
            updated = datetime.fromisoformat(order['updated_at'].replace('Z', '+00:00'))
            if updated < created:
                errors.append("Updated timestamp cannot be before created timestamp")
        
        return {
            "valid": len(errors) == 0,
            "errors": errors
        }
    
    def validate_dataset(self, df: pd.DataFrame, entity_type: str) -> Dict[str, Any]:
        """Validate entire dataset for data quality"""
        if entity_type == 'user':
            validator_func = self.validate_user
        elif entity_type == 'order':
            validator_func = self.validate_order
        else:
            raise ValueError(f"Unsupported entity type: {entity_type}")
        
        results = {
            'total_records': len(df),
            'valid_records': 0,
            'invalid_records': 0,
            'errors': [],
            'field_completeness': {},
            'summary': {}
        }
        
        # Check field completeness
        for column in df.columns:
            completeness = df[column].notna().mean()
            results['field_completeness'][column] = round(completeness * 100, 2)
        
        # Validate each record
        for idx, row in df.iterrows():
            validation = validator_func(row.to_dict())
            
            if validation['valid']:
                results['valid_records'] += 1
            else:
                results['invalid_records'] += 1
                for error in validation['errors']:
                    results['errors'].append(f"Record {idx}: {error}")
        
        # Generate summary
        results['summary'] = {
            'validity_rate': round(results['valid_records'] / results['total_records'] * 100, 2) if results['total_records'] > 0 else 0,
            'completeness_rate': round(sum(results['field_completeness'].values()) / len(results['field_completeness']) if results['field_completeness'] else 0, 2)
        }
        
        return results

# Usage example
if __name__ == "__main__":
    validator = DataValidator()
    
    # Test user validation
    test_user = {
        'user_id': 'USR12345',
        'first_name': 'John',
        'last_name': 'Smith',
        'email': 'john.smith@example.com',
        'phone': '+1-555-123-4567',
        'date_of_birth': '1990-01-15',
        'status': 'active',
        'created_at': '2025-01-15T10:30:00Z',
        'updated_at': '2025-01-15T10:30:00Z'
    }
    
    result = validator.validate_user(test_user)
    print("User validation:", result)
    
    # Test order validation
    test_order = {
        'order_id': 'ORD202501151234',
        'user_id': 'USR12345',
        'items': [
            {
                'product_id': 'PROD001',
                'name': 'Test Product',
                'quantity': 2,
                'unit_price': 25.00,
                'line_total': 50.00
            }
        ],
        'total_amount': 50.00,
        'status': 'pending',
        'created_at': '2025-01-15T10:30:00Z',
        'updated_at': '2025-01-15T10:30:00Z'
    }
    
    result = validator.validate_order(test_order)
    print("Order validation:", result)
```

### Data Quality Dashboard

```python
# dashboards/data_quality.py
import plotly.express as px
import plotly.graph_objects as go
from plotly.subplots import make_subplots
import pandas as pd
import numpy as np
from datetime import datetime, timedelta
import warnings
warnings.filterwarnings('ignore')

class DataQualityDashboard:
    """Generate interactive data quality dashboards"""
    
    def __init__(self):
        self.colors = {
            'primary': '#1f77b4',
            'secondary': '#ff7f0e',
            'success': '#2ca02c',
            'danger': '#d62728',
            'warning': '#ffbb78',
            'info': '#17becf'
        }
    
    def create_overview_dashboard(self, validation_results: Dict[str, Any], 
                                 time_series_data: pd.DataFrame = None) -> go.Figure:
        """Create comprehensive data quality overview dashboard"""
        fig = make_subplots(
            rows=3, cols=2,
            subplot_titles=[
                'Data Validity', 'Field Completeness',
                'Error Distribution', 'Validation Trends',
                'Top Data Issues', 'Completeness by Field'
            ],
            specs=[
                [{"type": "domain"}, {"type": "bar"}],
                [{"type": "pie"}, {"type": "scatter"}],
                [{"type": "table"}, {"type": "bar"}]
            ],
            vertical_spacing=0.1,
            horizontal_spacing=0.1
        )
        
        # 1. Data Validity (Donut chart)
        validity_data = {
            'labels': ['Valid', 'Invalid'],
            'values': [
                validation_results['valid_records'], 
                validation_results['invalid_records']
            ]
        }
        
        fig.add_trace(
            go.Pie(
                labels=validity_data['labels'],
                values=validity_data['values'],
                hole=0.4,
                marker_colors=[self.colors['success'], self.colors['danger']],
                textinfo='percent+label',
                textposition='outside'
            ),
            row=1, col=1
        )
        
        # 2. Field Completeness (Bar chart)
        completeness_data = list(validation_results['field_completeness'].items())
        completeness_data.sort(key=lambda x: x[1], reverse=True)
        
        fig.add_trace(
            go.Bar(
                x=[item[0] for item in completeness_data],
                y=[item[1] for item in completeness_data],
                marker_color=[
                    self.colors['success'] if item[1] >= 95 else 
                    self.colors['warning'] if item[1] >= 80 else 
                    self.colors['danger'] 
                    for item in completeness_data
                ],
                text=[f"{item[1]:.1f}%" for item in completeness_data],
                textposition='auto'
            ),
            row=1, col=2
        )
        
        # 3. Error Distribution (Pie chart)
        # Extract error types from error messages
        error_types = {}
        for error in validation_results['errors']:
            # Extract main error type from message
            if 'email' in error.lower():
                error_type = 'Email Issues'
            elif 'phone' in error.lower():
                error_type = 'Phone Issues'
            elif 'date' in error.lower():
                error_type = 'Date Issues'
            elif 'schema' in error.lower():
                error_type = 'Schema Issues'
            elif 'business' in error.lower():
                error_type = 'Business Rule Issues'
            else:
                error_type = 'Other Issues'
            
            error_types[error_type] = error_types.get(error_type, 0) + 1
        
        fig.add_trace(
            go.Pie(
                labels=list(error_types.keys()),
                values=list(error_types.values()),
                marker_colors=[self.colors['danger'], self.colors['warning'], 
                             self.colors['info'], self.colors['secondary'], 
                             self.colors['primary']],
                textinfo='percent+label',
                textposition='outside'
            ),
            row=2, col=1
        )
        
        # 4. Validation Trends (Line chart)
        if time_series_data is not None and not time_series_data.empty:
            # Assume time_series_data has columns: date, valid_count, invalid_count, completeness_rate
            fig.add_trace(
                go.Scatter(
                    x=time_series_data['date'],
                    y=time_series_data['validity_rate'],
                    mode='lines+markers',
                    name='Validity Rate',
                    line=dict(color=self.colors['success'], width=3)
                ),
                row=2, col=2
            )
            
            fig.add_trace(
                go.Scatter(
                    x=time_series_data['date'],
                    y=time_series_data['completeness_rate'],
                    mode='lines+markers',
                    name='Completeness Rate',
                    line=dict(color=self.colors['primary'], width=3, dash='dot')
                ),
                row=2, col=2
            )
        
        # 5. Top Data Issues (Table)
        error_series = pd.Series(validation_results['errors'])
        top_issues = error_series.value_counts().head(5)
        
        fig.add_trace(
            go.Table(
                header=dict(
                    values=['Issue', 'Count'],
                    fill_color='paleturquoise',
                    align='left'
                ),
                cells=dict(
                    values=[top_issues.index, top_issues.values],
                    fill_color='lavender',
                    align='left'
                )
            ),
            row=3, col=1
        )
        
        # 6. Completeness by Field (Horizontal Bar)
        fig.add_trace(
            go.Bar(
                y=[item[0] for item in completeness_data],
                x=[item[1] for item in completeness_data],
                orientation='h',
                marker_color=[
                    self.colors['success'] if item[1] >= 95 else 
                    self.colors['warning'] if item[1] >= 80 else 
                    self.colors['danger'] 
                    for item in completeness_data
                ],
                text=[f"{item[1]:.1f}%" for item in completeness_data],
                textposition='auto'
            ),
            row=3, col=2
        )
        
        # Update layout
        fig.update_layout(
            title={
                'text': 'Data Quality Dashboard',
                'x': 0.5,
                'xanchor': 'center',
                'font': {'size': 24}
            },
            height=1200,
            showlegend=False,
            margin=dict(l=50, r=50, t=80, b=50)
        )
        
        # Update subplot titles font size
        for annotation in fig['layout']['annotations']:
            annotation['font'] = dict(size=14)
        
        return fig
    
    def create_field_analysis_dashboard(self, df: pd.DataFrame, field_name: str) -> go.Figure:
        """Create detailed analysis dashboard for a specific field"""
        # Get field data, handling potential missing field
        if field_name not in df.columns:
            raise ValueError(f"Field '{field_name}' not found in dataframe")
            
        field_data = df[field_name].copy()
        
        # Create subplots
        fig = make_subplots(
            rows=2, cols=2,
            subplot_titles=[
                f'{field_name} Distribution',
                f'{field_name} Completeness Over Time',
                f'{field_name} Completeness by Category',
                f'{field_name} Top Values'
            ],
            specs=[[{"type": "histogram"}, {"type": "scatter"}],
                   ["", {"type": "bar"}]],
            vertical_spacing=0.15,
            horizontal_spacing=0.1
        )
        
        # Remove empty subplot (row=2, col=1)
        fig.layout['xaxis3'].update(showticklabels=False, showgrid=False, zeroline=False)
        fig.layout['yaxis3'].update(visible=False)
        
        # 1. Distribution (Histogram)
        # For numeric fields
        if pd.api.types.is_numeric_dtype(field_data):
            fig.add_trace(
                go.Histogram(
                    x=field_data.dropna(),
                    nbinsx=50,
                    marker_color=self.colors['primary']
                ),
                row=1, col=1
            )
            
            # Add summary statistics
            summary_text = (
                f"Mean: {field_data.mean():.2f}<br>"
                f"Std: {field_data.std():.2f}<br>"
                f"Min: {field_data.min():.2f}<br>"
                f"Max: {field_data.max():.2f}<br>"
                f"Nulls: {field_data.isna().sum()} ({field_data.isna().mean()*100:.1f}%)")
            
            fig.add_annotation(
                xref="x domain", yref="y domain",
                x=0.02, y=0.98,
                text=summary_text,
                showarrow=False,
                bgcolor="white",
                bordercolor="black",
                borderwidth=1,
                opacity=0.8,
                align="left"
            )
        
        # For categorical fields
        else:
            # Convert to string and handle nulls
            field_data = field_data.astype(str)
            field_data = field_data.replace('nan', 'Null')
            
            # Get top 10 values
            value_counts = field_data.value_counts().head(10)
            
            fig.add_trace(
                go.Bar(
                    x=value_counts.index,
                    y=value_counts.values,
                    marker_color=self.colors['primary']
                ),
                row=1, col=1
            )
            
            # Add summary
            summary_text = (
                f"Unique values: {field_data.nunique()}<br>"
                f"Total records: {len(field_data)}<br>"
                f"Nulls: {field_data.isin(['Null', '']).sum()} ({(field_data.isin(['Null', ''])).mean()*100:.1f}%)")
            
            fig.add_annotation(
                xref="x domain", yref="y domain",
                x=0.02, y=0.98,
                text=summary_text,
                showarrow=False,
                bgcolor="white",
                bordercolor="black",
                borderwidth=1,
                opacity=0.8,
                align="left"
            )
        
        # 2. Completeness Over Time (if date column exists)
        date_columns = [col for col in df.columns if 'date' in col.lower() or 'time' in col.lower()]
        if date_columns:
            date_col = date_columns[0]  # Use first date column
            df_sorted = df.sort_values(date_col)
            
            # Convert date if needed
            if not pd.api.types.is_datetime64_any_dtype(df_sorted[date_col]):
                df_sorted[date_col] = pd.to_datetime(df_sorted[date_col])
                
            # Calculate completeness over time (rolling 7-day window)
            df_sorted = df_sorted.set_index(date_col)
            completeness_rolling = df_sorted[field_name].notna().rolling('7D').mean()
            
            fig.add_trace(
                go.Scatter(
                    x=completeness_rolling.index,
                    y=completeness_rolling.values * 100,
                    mode='lines',
                    line=dict(color=self.colors['success'], width=3),
                    name='Completeness'
                ),
                row=1, col=2
            )
            
            # Add horizontal line for overall completeness
            overall_completeness = df[field_name].notna().mean() * 100
            fig.add_hline(
                y=overall_completeness,
                line_dash="dot",
                line_color=self.colors['primary'],
                annotation_text=f"Overall: {overall_completeness:.1f}%",
                annotation_position="top left",
                row=1, col=2
            )
        
        # 3. Completeness by Category (if categorical columns exist)
        # Find potential category columns (low cardinality categorical)
        categorical_cols = []
        for col in df.columns:
            if col != field_name and col != date_col:
                if (df[col].dtype == 'object' and 
                    df[col].nunique() <= 10 and 
                    df[col].nunique() > 1):
                    categorical_cols.append(col)
        
        if categorical_cols:
            category_col = categorical_cols[0]  # Use first suitable category column
            completeness_by_category = df.groupby(category_col)[field_name].apply(
                lambda x: x.notna().mean() * 100
            ).round(1)
            
            fig.add_trace(
                go.Bar(
                    x=completeness_by_category.index,
                    y=completeness_by_category.values,
                    marker_color=[
                        self.colors['success'] if val >= 95 else 
                        self.colors['warning'] if val >= 80 else 
                        self.colors['danger'] 
                        for val in completeness_by_category.values
                    ],
                    text=[f"{val:.1f}%" for val in completeness_by_category.values],
                    textposition='auto'
                ),
                row=2, col=2
            )
            
            # Update subplot title to show the category
            fig.layout.annotations[2].text = f"{field_name} Completeness by {category_col.title()}"
        
        # 4. Top Values (for categorical fields)
        if not pd.api.types.is_numeric_dtype(field_data):
            # Get top 10 values
            top_values = field_data.value_counts().head(10)
            
            fig.add_trace(
                go.Bar(
                    x=top_values.values,
                    y=top_values.index,
                    orientation='h',
                    marker_color=self.colors['primary']
                ),
                row=2, col=2
            )
            
            # Update subplot title
            fig.layout.annotations[3].text = f"Top {field_name} Values"
        
        # Update layout
        fig.update_layout(
            title={
                'text': f'Detailed Analysis: {field_name}',
                'x': 0.5,
                'xanchor': 'center',
                'font': {'size': 20}
            },
            height=800,
            showlegend=False,
            margin=dict(l=50, r=50, t=80, b=50)
        )
        
        # Update subplot titles font size
        for annotation in fig['layout']['annotations']:
            annotation['font'] = dict(size=12)
        
        return fig
    
    def export_dashboard(self, fig: go.Figure, filepath: str, format: str = 'html') -> None:
        """Export dashboard to file"""
        if format == 'html':
            fig.write_html(filepath)
        elif format == 'png':
            fig.write_image(filepath, width=1200, height=800)
        elif format == 'pdf':
            fig.write_image(filepath)
        else:
            raise ValueError(f"Unsupported format: {format}")

# Usage example
if __name__ == "__main__":
    # Create sample data for demonstration
    np.random.seed(42)
    dates = pd.date_range('2025-01-01', '2025-03-31', freq='D')
    
    time_series_data = pd.DataFrame({
        'date': dates,
        'validity_rate': np.random.normal(95, 5, len(dates)).clip(80, 100),
        'completeness_rate': np.random.normal(92, 8, len(dates)).clip(70, 100)
    })
    
    # Create validation results
    validation_results = {
        'total_records': 1000,
        'valid_records': 920,
        'invalid_records': 80,
        'field_completeness': {
            'user_id': 100.0,
            'first_name': 98.5,
            'last_name': 98.0,
            'email': 96.2,
            'phone': 85.3,
            'date_of_birth': 92.1,
            'status': 100.0,
            'created_at': 100.0,
            'updated_at': 100.0
        },
        'errors': [
            'User 10: Invalid email format',
            'User 15: Phone number too short',
            'User 23: Date of birth in future',
            'User 34: First name not capitalized',
            'User 45: Disposable email domain',
            'User 10: Invalid email format',
            'User 15: Phone number too short',
            'User 23: Date of birth in future'
        ]
    }
    
    # Create dashboard
    dashboard = DataQualityDashboard()
    fig = dashboard.create_overview_dashboard(validation_results, time_series_data)
    fig.show()
    
    # Export to HTML
    dashboard.export_dashboard(fig, 'data_quality_dashboard.html', 'html')
```
