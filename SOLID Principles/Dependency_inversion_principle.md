# Dependency Inversion Principle (DIP)

## Table of Contents
1. [Introduction](#introduction)
2. [Definition and Theory](#definition-and-theory)
3. [Why DIP Matters](#why-dip-matters)
4. [When to Apply DIP](#when-to-apply-dip)
5. [Consequences of Violating DIP](#consequences-of-violating-dip)
6. [Python Examples](#python-examples)
7. [Dependency Injection Patterns](#dependency-injection-patterns)
8. [Benefits of Following DIP](#benefits-of-following-dip)
9. [Common Pitfalls](#common-pitfalls)
10. [Best Practices](#best-practices)
11. [Conclusion](#conclusion)

## Introduction

The Dependency Inversion Principle (DIP) is the fifth and final principle in the SOLID principles of object-oriented design, introduced by Robert C. Martin. It addresses the problem of high-level modules depending on low-level modules, which creates tight coupling and makes systems difficult to maintain, test, and extend.

DIP is often confused with Dependency Injection (DI), but they are related yet distinct concepts. DIP is the principle that guides the design, while DI is one of the techniques used to implement DIP.

## Definition and Theory

### Formal Definition
**"High-level modules should not depend on low-level modules. Both should depend on abstractions. Abstractions should not depend on details. Details should depend on abstractions."**

### Two Core Rules

#### Rule 1: High-level modules should not depend on low-level modules
- **High-level modules**: Modules that contain complex business logic and policy decisions
- **Low-level modules**: Modules that handle basic operations like database access, file I/O, network communication
- **Solution**: Both should depend on abstractions (interfaces, abstract base classes)

#### Rule 2: Abstractions should not depend on details
- **Abstractions**: Interfaces, abstract base classes, protocols
- **Details**: Concrete implementations, specific technologies, frameworks
- **Solution**: Design abstractions that are stable and don't change when implementation details change

### Core Concepts

#### Dependency Direction
Traditional layered architecture has dependencies flowing downward (high-level → low-level). DIP inverts this flow by introducing abstractions.

#### Inversion of Control (IoC)
The principle of inverting the control of dependency creation and management from the dependent class to an external entity.

#### Dependency Injection (DI)
A technique for implementing IoC where dependencies are provided to a class from the outside rather than created internally.

#### Abstractions vs Concretions
Depending on stable abstractions rather than volatile concrete implementations.

## Why DIP Matters

### 1. **Testability**
High-level modules can be tested in isolation by injecting mock implementations of low-level dependencies.

### 2. **Flexibility**
Different implementations can be swapped without changing high-level code.

### 3. **Maintainability**
Changes to low-level modules don't affect high-level business logic.

### 4. **Reusability**
High-level modules can work with different low-level implementations.

### 5. **Parallel Development**
Teams can work on different layers simultaneously by depending on agreed-upon abstractions.

### 6. **Technology Independence**
Business logic becomes independent of specific technologies and frameworks.

## When to Apply DIP

### You Should Apply DIP When:

1. **Building layered architectures**
2. **Working with external dependencies** (databases, web services, file systems)
3. **Creating testable code**
4. **Building frameworks or libraries**
5. **Working in teams with multiple developers**
6. **Requirements are likely to change**
7. **Multiple implementations of the same concept exist**

### Warning Signs That DIP is Violated:

- High-level classes directly instantiate low-level classes
- Business logic mixed with infrastructure concerns
- Difficult to unit test classes in isolation
- Changes to databases or external services require changes to business logic
- Hard-coded dependencies throughout the codebase
- Tight coupling between layers
- Inability to swap implementations easily

## Consequences of Violating DIP

### 1. **Tight Coupling**
High-level modules become tightly coupled to specific low-level implementations.

### 2. **Difficult Testing**
Unit testing becomes challenging because dependencies cannot be easily mocked or stubbed.

### 3. **Reduced Flexibility**
Changing low-level implementations requires changes to high-level modules.

### 4. **Technology Lock-in**
Business logic becomes tied to specific technologies and frameworks.

### 5. **Parallel Development Issues**
Teams cannot work independently because of tight coupling between layers.

### 6. **Maintenance Overhead**
Changes ripple through multiple layers of the application.

## Python Examples

### Example 1: Email Notification System

#### Violating DIP

```python
# BAD: Violates DIP - High-level module depends on low-level modules
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
import requests
import json

class SMTPEmailSender:
    """Low-level module for SMTP email sending"""
    
    def __init__(self, smtp_server: str, port: int, username: str, password: str):
        self.smtp_server = smtp_server
        self.port = port
        self.username = username
        self.password = password
    
    def send_email(self, to: str, subject: str, body: str) -> bool:
        try:
            msg = MIMEMultipart()
            msg['From'] = self.username
            msg['To'] = to
            msg['Subject'] = subject
            msg.attach(MIMEText(body, 'plain'))
            
            server = smtplib.SMTP(self.smtp_server, self.port)
            server.starttls()
            server.login(self.username, self.password)
            server.send_message(msg)
            server.quit()
            return True
        except Exception as e:
            print(f"Failed to send email: {e}")
            return False

class SlackNotifier:
    """Low-level module for Slack notifications"""
    
    def __init__(self, webhook_url: str):
        self.webhook_url = webhook_url
    
    def send_message(self, channel: str, message: str) -> bool:
        try:
            payload = {
                'channel': channel,
                'text': message
            }
            response = requests.post(self.webhook_url, json=payload)
            return response.status_code == 200
        except Exception as e:
            print(f"Failed to send Slack message: {e}")
            return False

class UserRegistrationService:
    """High-level module that directly depends on low-level modules"""
    
    def __init__(self):
        # Problem: Direct dependency on concrete implementations
        self.email_sender = SMTPEmailSender(
            "smtp.gmail.com", 587, "user@gmail.com", "password"
        )
        self.slack_notifier = SlackNotifier("https://hooks.slack.com/webhook")
    
    def register_user(self, user_data: dict) -> bool:
        """High-level business logic mixed with low-level concerns"""
        
        # Validate user data
        if not self._validate_user_data(user_data):
            return False
        
        # Save user to database (more tight coupling)
        user_id = self._save_user_to_database(user_data)
        if not user_id:
            return False
        
        # Problem: Business logic directly calls low-level modules
        # Send welcome email
        welcome_subject = "Welcome to our platform!"
        welcome_body = f"Hello {user_data['name']}, welcome to our platform!"
        
        if not self.email_sender.send_email(
            user_data['email'], welcome_subject, welcome_body
        ):
            print("Failed to send welcome email")
            # What should we do here? Rollback user creation?
        
        # Notify admin via Slack
        admin_message = f"New user registered: {user_data['name']} ({user_data['email']})"
        
        if not self.slack_notifier.send_message("#admin", admin_message):
            print("Failed to notify admin")
        
        return True
    
    def _validate_user_data(self, user_data: dict) -> bool:
        # Business logic for validation
        required_fields = ['name', 'email', 'password']
        return all(field in user_data for field in required_fields)
    
    def _save_user_to_database(self, user_data: dict) -> int:
        # Problem: More tight coupling to database implementation
        print(f"Saving user {user_data['name']} to database")
        return 12345  # Simulated user ID

# Usage - difficult to test and inflexible
def main():
    registration_service = UserRegistrationService()
    
    user_data = {
        'name': 'John Doe',
        'email': 'john@example.com',
        'password': 'secret123'
    }
    
    # Problem: Cannot easily test this without actually sending emails and Slack messages
    # Problem: Cannot easily switch to different email or notification providers
    # Problem: Cannot easily test failure scenarios
    
    success = registration_service.register_user(user_data)
    print(f"Registration successful: {success}")

if __name__ == "__main__":
    main()
```

#### Following DIP

```python
# GOOD: Following DIP - Depend on abstractions
from abc import ABC, abstractmethod
from typing import Dict, Any, Optional
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
import requests

# Abstractions (high-level modules depend on these)
class EmailSender(ABC):
    """Abstraction for email sending"""
    
    @abstractmethod
    def send_email(self, to: str, subject: str, body: str) -> bool:
        pass

class NotificationSender(ABC):
    """Abstraction for notifications"""
    
    @abstractmethod
    def send_notification(self, recipient: str, message: str) -> bool:
        pass

class UserRepository(ABC):
    """Abstraction for user data persistence"""
    
    @abstractmethod
    def save_user(self, user_data: Dict[str, Any]) -> Optional[int]:
        pass
    
    @abstractmethod
    def get_user_by_email(self, email: str) -> Optional[Dict[str, Any]]:
        pass

# Low-level modules (concrete implementations)
class SMTPEmailSender(EmailSender):
    """Concrete implementation for SMTP email sending"""
    
    def __init__(self, smtp_server: str, port: int, username: str, password: str):
        self.smtp_server = smtp_server
        self.port = port
        self.username = username
        self.password = password
    
    def send_email(self, to: str, subject: str, body: str) -> bool:
        try:
            msg = MIMEMultipart()
            msg['From'] = self.username
            msg['To'] = to
            msg['Subject'] = subject
            msg.attach(MIMEText(body, 'plain'))
            
            server = smtplib.SMTP(self.smtp_server, self.port)
            server.starttls()
            server.login(self.username, self.password)
            server.send_message(msg)
            server.quit()
            return True
        except Exception as e:
            print(f"SMTP failed to send email: {e}")
            return False

class SendGridEmailSender(EmailSender):
    """Alternative email implementation using SendGrid"""
    
    def __init__(self, api_key: str):
        self.api_key = api_key
    
    def send_email(self, to: str, subject: str, body: str) -> bool:
        # Simulated SendGrid implementation
        print(f"SendGrid: Sending email to {to} with subject '{subject}'")
        return True

class SlackNotificationSender(NotificationSender):
    """Concrete implementation for Slack notifications"""
    
    def __init__(self, webhook_url: str, default_channel: str = "#general"):
        self.webhook_url = webhook_url
        self.default_channel = default_channel
    
    def send_notification(self, recipient: str, message: str) -> bool:
        try:
            payload = {
                'channel': recipient or self.default_channel,
                'text': message
            }
            response = requests.post(self.webhook_url, json=payload)
            return response.status_code == 200
        except Exception as e:
            print(f"Slack failed to send notification: {e}")
            return False

class DiscordNotificationSender(NotificationSender):
    """Alternative notification implementation using Discord"""
    
    def __init__(self, webhook_url: str):
        self.webhook_url = webhook_url
    
    def send_notification(self, recipient: str, message: str) -> bool:
        # Simulated Discord implementation
        print(f"Discord: Sending notification to {recipient}: {message}")
        return True

class DatabaseUserRepository(UserRepository):
    """Concrete implementation for database user storage"""
    
    def __init__(self, connection_string: str):
        self.connection_string = connection_string
        self.users = {}  # Simulated database
        self.next_id = 1
    
    def save_user(self, user_data: Dict[str, Any]) -> Optional[int]:
        try:
            user_id = self.next_id
            self.users[user_id] = user_data.copy()
            self.next_id += 1
            print(f"Database: Saved user {user_data['name']} with ID {user_id}")
            return user_id
        except Exception as e:
            print(f"Database error: {e}")
            return None
    
    def get_user_by_email(self, email: str) -> Optional[Dict[str, Any]]:
        for user_data in self.users.values():
            if user_data.get('email') == email:
                return user_data
        return None

class InMemoryUserRepository(UserRepository):
    """Alternative implementation for testing or caching"""
    
    def __init__(self):
        self.users = {}
        self.next_id = 1
    
    def save_user(self, user_data: Dict[str, Any]) -> Optional[int]:
        user_id = self.next_id
        self.users[user_id] = user_data.copy()
        self.next_id += 1
        print(f"Memory: Saved user {user_data['name']} with ID {user_id}")
        return user_id
    
    def get_user_by_email(self, email: str) -> Optional[Dict[str, Any]]:
        for user_data in self.users.values():
            if user_data.get('email') == email:
                return user_data
        return None

# High-level module (business logic)
class UserRegistrationService:
    """High-level module that depends on abstractions"""
    
    def __init__(
        self, 
        user_repository: UserRepository,
        email_sender: EmailSender,
        notification_sender: NotificationSender
    ):
        # Dependency injection - dependencies provided from outside
        self.user_repository = user_repository
        self.email_sender = email_sender
        self.notification_sender = notification_sender
    
    def register_user(self, user_data: Dict[str, Any]) -> bool:
        """Pure business logic without infrastructure concerns"""
        
        # Validate user data
        if not self._validate_user_data(user_data):
            return False
        
        # Check if user already exists
        existing_user = self.user_repository.get_user_by_email(user_data['email'])
        if existing_user:
            print(f"User with email {user_data['email']} already exists")
            return False
        
        # Save user
        user_id = self.user_repository.save_user(user_data)
        if not user_id:
            print("Failed to save user")
            return False
        
        # Send welcome email
        welcome_success = self._send_welcome_email(user_data)
        
        # Notify administrators
        admin_success = self._notify_admin(user_data)
        
        # Business decision: Registration succeeds even if notifications fail
        if not welcome_success:
            print("Warning: Failed to send welcome email")
        
        if not admin_success:
            print("Warning: Failed to notify admin")
        
        return True
    
    def _validate_user_data(self, user_data: Dict[str, Any]) -> bool:
        """Business logic for user validation"""
        required_fields = ['name', 'email', 'password']
        
        # Check required fields
        if not all(field in user_data for field in required_fields):
            print("Missing required fields")
            return False
        
        # Validate email format
        email = user_data['email']
        if '@' not in email or '.' not in email:
            print("Invalid email format")
            return False
        
        # Validate password strength
        password = user_data['password']
        if len(password) < 8:
            print("Password must be at least 8 characters")
            return False
        
        return True
    
    def _send_welcome_email(self, user_data: Dict[str, Any]) -> bool:
        """Send welcome email to new user"""
        subject = "Welcome to our platform!"
        body = f"""
        Hello {user_data['name']},
        
        Welcome to our platform! We're excited to have you on board.
        
        Best regards,
        The Team
        """
        
        return self.email_sender.send_email(
            user_data['email'], subject, body.strip()
        )
    
    def _notify_admin(self, user_data: Dict[str, Any]) -> bool:
        """Notify administrators about new user registration"""
        message = f"New user registered: {user_data['name']} ({user_data['email']})"
        return self.notification_sender.send_notification("#admin", message)

# Composition Root (where dependencies are wired together)
class ApplicationContainer:
    """Dependency injection container"""
    
    def __init__(self, config: Dict[str, Any]):
        self.config = config
        self._email_sender = None
        self._notification_sender = None
        self._user_repository = None
        self._registration_service = None
    
    def get_email_sender(self) -> EmailSender:
        if self._email_sender is None:
            email_config = self.config.get('email', {})
            provider = email_config.get('provider', 'smtp')
            
            if provider == 'smtp':
                self._email_sender = SMTPEmailSender(
                    email_config['smtp_server'],
                    email_config['port'],
                    email_config['username'],
                    email_config['password']
                )
            elif provider == 'sendgrid':
                self._email_sender = SendGridEmailSender(
                    email_config['api_key']
                )
            else:
                raise ValueError(f"Unknown email provider: {provider}")
        
        return self._email_sender
    
    def get_notification_sender(self) -> NotificationSender:
        if self._notification_sender is None:
            notification_config = self.config.get('notifications', {})
            provider = notification_config.get('provider', 'slack')
            
            if provider == 'slack':
                self._notification_sender = SlackNotificationSender(
                    notification_config['webhook_url']
                )
            elif provider == 'discord':
                self._notification_sender = DiscordNotificationSender(
                    notification_config['webhook_url']
                )
            else:
                raise ValueError(f"Unknown notification provider: {provider}")
        
        return self._notification_sender
    
    def get_user_repository(self) -> UserRepository:
        if self._user_repository is None:
            db_config = self.config.get('database', {})
            provider = db_config.get('provider', 'database')
            
            if provider == 'database':
                self._user_repository = DatabaseUserRepository(
                    db_config['connection_string']
                )
            elif provider == 'memory':
                self._user_repository = InMemoryUserRepository()
            else:
                raise ValueError(f"Unknown database provider: {provider}")
        
        return self._user_repository
    
    def get_registration_service(self) -> UserRegistrationService:
        if self._registration_service is None:
            self._registration_service = UserRegistrationService(
                self.get_user_repository(),
                self.get_email_sender(),
                self.get_notification_sender()
            )
        
        return self._registration_service

# Mock implementations for testing
class MockEmailSender(EmailSender):
    """Mock implementation for testing"""
    
    def __init__(self):
        self.sent_emails = []
    
    def send_email(self, to: str, subject: str, body: str) -> bool:
        self.sent_emails.append({
            'to': to,
            'subject': subject,
            'body': body
        })
        print(f"Mock: Email sent to {to}")
        return True

class MockNotificationSender(NotificationSender):
    """Mock implementation for testing"""
    
    def __init__(self):
        self.sent_notifications = []
    
    def send_notification(self, recipient: str, message: str) -> bool:
        self.sent_notifications.append({
            'recipient': recipient,
            'message': message
        })
        print(f"Mock: Notification sent to {recipient}")
        return True

# Usage and Testing
def main():
    # Production configuration
    production_config = {
        'email': {
            'provider': 'smtp',
            'smtp_server': 'smtp.gmail.com',
            'port': 587,
            'username': 'user@gmail.com',
            'password': 'password'
        },
        'notifications': {
            'provider': 'slack',
            'webhook_url': 'https://hooks.slack.com/webhook'
        },
        'database': {
            'provider': 'database',
            'connection_string': 'postgresql://localhost/mydb'
        }
    }
    
    # Create application container
    container = ApplicationContainer(production_config)
    registration_service = container.get_registration_service()
    
    # Use the service
    user_data = {
        'name': 'John Doe',
        'email': 'john@example.com',
        'password': 'secret123'
    }
    
    success = registration_service.register_user(user_data)
    print(f"Registration successful: {success}")

def test_registration_service():
    """Example of how easy testing becomes with DI"""
    
    # Create mock dependencies
    user_repository = InMemoryUserRepository()
    email_sender = MockEmailSender()
    notification_sender = MockNotificationSender()
    
    # Inject mocks into service
    service = UserRegistrationService(
        user_repository, email_sender, notification_sender
    )
    
    # Test user registration
    user_data = {
        'name': 'Test User',
        'email': 'test@example.com',
        'password': 'testpass123'
    }
    
    success = service.register_user(user_data)
    
    # Verify behavior
    assert success == True
    assert len(email_sender.sent_emails) == 1
    assert len(notification_sender.sent_notifications) == 1
    assert email_sender.sent_emails[0]['to'] == 'test@example.com'
    
    print("All tests passed!")

if __name__ == "__main__":
    print("=== Production Usage ===")
    main()
    
    print("\n=== Testing ===")
    test_registration_service()
```

### Example 2: Order Processing System

#### Violating DIP

```python
# BAD: Violates DIP - Tight coupling to specific implementations
import sqlite3
import requests
import json
from datetime import datetime

class SQLiteDatabase:
    """Low-level database implementation"""
    
    def __init__(self, db_path: str):
        self.db_path = db_path
        self.connection = sqlite3.connect(db_path, check_same_thread=False)
        self._create_tables()
    
    def _create_tables(self):
        cursor = self.connection.cursor()
        cursor.execute('''
            CREATE TABLE IF NOT EXISTS orders (
                id INTEGER PRIMARY KEY,
                customer_id INTEGER,
                total REAL,
                status TEXT,
                created_at TEXT
            )
        ''')
        self.connection.commit()
    
    def save_order(self, order_data: dict) -> int:
        cursor = self.connection.cursor()
        cursor.execute('''
            INSERT INTO orders (customer_id, total, status, created_at)
            VALUES (?, ?, ?, ?)
        ''', (
            order_data['customer_id'],
            order_data['total'],
            order_data['status'],
            order_data['created_at']
        ))
        self.connection.commit()
        return cursor.lastrowid

class StripePaymentProcessor:
    """Low-level payment processing implementation"""
    
    def __init__(self, api_key: str):
        self.api_key = api_key
    
    def process_payment(self, amount: float, card_token: str) -> dict:
        # Simulated Stripe API call
        print(f"Processing ${amount} payment with Stripe")
        return {
            'success': True,
            'transaction_id': 'ch_1234567890',
            'amount': amount
        }

class SMSNotificationService:
    """Low-level SMS notification implementation"""
    
    def __init__(self, api_key: str, base_url: str):
        self.api_key = api_key
        self.base_url = base_url
    
    def send_sms(self, phone: str, message: str) -> bool:
        # Simulated SMS API call
        print(f"Sending SMS to {phone}: {message}")
        return True

class OrderService:
    """High-level business logic tightly coupled to low-level modules"""
    
    def __init__(self):
        # Problem: Direct dependencies on concrete implementations
        self.database = SQLiteDatabase("orders.db")
        self.payment_processor = StripePaymentProcessor("sk_test_123")
        self.sms_service = SMSNotificationService("sms_key_123", "https://api.sms.com")
    
    def process_order(self, order_data: dict) -> bool:
        """Business logic mixed with infrastructure concerns"""
        
        try:
            # Validate order
            if not self._validate_order(order_data):
                return False
            
            # Process payment directly with Stripe
            payment_result = self.payment_processor.process_payment(
                order_data['total'],
                order_data['card_token']
            )
            
            if not payment_result['success']:
                print("Payment failed")
                return False
            
            # Save order directly to SQLite
            order_record = {
                'customer_id': order_data['customer_id'],
                'total': order_data['total'],
                'status': 'confirmed',
                'created_at': datetime.now().isoformat()
            }
            
            order_id = self.database.save_order(order_record)
            
            # Send SMS notification directly
            sms_message = f"Order #{order_id} confirmed. Total: ${order_data['total']}"
            self.sms_service.send_sms(order_data['phone'], sms_message)
            
            return True
            
        except Exception as e:
            print(f"Order processing failed: {e}")
            return False
    
    def _validate_order(self, order_data: dict) -> bool:
        required_fields = ['customer_id', 'total', 'card_token', 'phone']
        return all(field in order_data for field in required_fields)

# Usage - difficult to test and inflexible
def main():
    order_service = OrderService()
    
    order_data = {
        'customer_id': 123,
        'total': 99.99,
        'card_token': 'tok_visa',
        'phone': '+1234567890'
    }
    
    # Problems:
    # - Cannot test without actual database, Stripe, and SMS service
    # - Cannot easily switch payment providers
    # - Cannot easily change database or notification methods
    # - Business logic is mixed with infrastructure concerns
    
    success = order_service.process_order(order_data)
    print(f"Order processing result: {success}")

if __name__ == "__main__":
    main()
```

#### Following DIP

```python
# GOOD: Following DIP - Depend on abstractions
from abc import ABC, abstractmethod
from typing import Dict, Any, Optional, List
from datetime import datetime
from enum import Enum
import sqlite3
import requests

# Domain models
class OrderStatus(Enum):
    PENDING = "pending"
    CONFIRMED = "confirmed"
    CANCELLED = "cancelled"
    SHIPPED = "shipped"

class Order:
    def __init__(self, customer_id: int, total: float, items: List[Dict[str, Any]]):
        self.id: Optional[int] = None
        self.customer_id = customer_id
        self.total = total
        self.items = items
        self.status = OrderStatus.PENDING
        self.created_at = datetime.now()
        self.payment_id: Optional[str] = None

class PaymentResult:
    def __init__(self, success: bool, transaction_id: str = None, error_message: str = None):
        self.success = success
        self.transaction_id = transaction_id
        self.error_message = error_message

# Abstractions (interfaces)
class OrderRepository(ABC):
    """Abstraction for order persistence"""
    
    @abstractmethod
    def save_order(self, order: Order) -> int:
        pass
    
    @abstractmethod
    def get_order(self, order_id: int) -> Optional[Order]:
        pass
    
    @abstractmethod
    def update_order_status(self, order_id: int, status: OrderStatus) -> bool:
        pass

class PaymentProcessor(ABC):
    """Abstraction for payment processing"""
    
    @abstractmethod
    def process_payment(self, amount: float, payment_details: Dict[str, Any]) -> PaymentResult:
        pass
    
    @abstractmethod
    def refund_payment(self, transaction_id: str, amount: float) -> PaymentResult:
        pass

class NotificationService(ABC):
    """Abstraction for notifications"""
    
    @abstractmethod
    def send_order_confirmation(self, order: Order, contact_info: Dict[str, str]) -> bool:
        pass
    
    @abstractmethod
    def send_order_update(self, order: Order, contact_info: Dict[str, str]) -> bool:
        pass

class InventoryService(ABC):
    """Abstraction for inventory management"""
    
    @abstractmethod
    def reserve_items(self, items: List[Dict[str, Any]]) -> bool:
        pass
    
    @abstractmethod
    def release_items(self, items: List[Dict[str, Any]]) -> bool:
        pass

# Concrete implementations (low-level modules)
class SQLiteOrderRepository(OrderRepository):
    """SQLite implementation of order storage"""
    
    def __init__(self, db_path: str):
        self.db_path = db_path
        self.connection = sqlite3.connect(db_path, check_same_thread=False)
        self._create_tables()
    
    def _create_tables(self):
        cursor = self.connection.cursor()
        cursor.execute('''
            CREATE TABLE IF NOT EXISTS orders (
                id INTEGER PRIMARY KEY,
                customer_id INTEGER,
                total REAL,
                status TEXT,
                created_at TEXT,
                payment_id TEXT
            )
        ''')
        self.connection.commit()
    
    def save_order(self, order: Order) -> int:
        cursor = self.connection.cursor()
        cursor.execute('''
            INSERT INTO orders (customer_id, total, status, created_at, payment_id)
            VALUES (?, ?, ?, ?, ?)
        ''', (
            order.customer_id,
            order.total,
            order.status.value,
            order.created_at.isoformat(),
            order.payment_id
        ))
        self.connection.commit()
        order.id = cursor.lastrowid
        return order.id
    
    def get_order(self, order_id: int) -> Optional[Order]:
        cursor = self.connection.cursor()
        cursor.execute('SELECT * FROM orders WHERE id = ?', (order_id,))
        row = cursor.fetchone()
        
        if row:
            order = Order(row[1], row[2], [])  # customer_id, total, empty items
            order.id = row[0]
            order.status = OrderStatus(row[3])
            order.created_at = datetime.fromisoformat(row[4])
            order.payment_id = row[5]
            return order
        return None
    
    def update_order_status(self, order_id: int, status: OrderStatus) -> bool:
        cursor = self.connection.cursor()
        cursor.execute(
            'UPDATE orders SET status = ? WHERE id = ?',
            (status.value, order_id)
        )
        self.connection.commit()
        return cursor.rowcount > 0

class InMemoryOrderRepository(OrderRepository):
    """In-memory implementation for testing"""
    
    def __init__(self):
        self.orders: Dict[int, Order] = {}
        self.next_id = 1
    
    def save_order(self, order: Order) -> int:
        order.id = self.next_id
        self.orders[order.id] = order
        self.next_id += 1
        return order.id
    
    def get_order(self, order_id: int) -> Optional[Order]:
        return self.orders.get(order_id)
    
    def update_order_status(self, order_id: int, status: OrderStatus) -> bool:
        if order_id in self.orders:
            self.orders[order_id].status = status
            return True
        return False

class StripePaymentProcessor(PaymentProcessor):
    """Stripe implementation of payment processing"""
    
    def __init__(self, api_key: str):
        self.api_key = api_key
    
    def process_payment(self, amount: float, payment_details: Dict[str, Any]) -> PaymentResult:
        try:
            # Simulated Stripe API call
            print(f"Stripe: Processing ${amount} payment")
            
            # Simulate success/failure based on card token
            if payment_details.get('card_token') == 'tok_fail':
                return PaymentResult(False, error_message="Card declined")
            
            transaction_id = f"ch_{hash(str(amount) + str(datetime.now()))}"
            return PaymentResult(True, transaction_id)
            
        except Exception as e:
            return PaymentResult(False, error_message=str(e))
    
    def refund_payment(self, transaction_id: str, amount: float) -> PaymentResult:
        print(f"Stripe: Refunding ${amount} for transaction {transaction_id}")
        return PaymentResult(True, f"re_{transaction_id}")

class PayPalPaymentProcessor(PaymentProcessor):
    """PayPal implementation of payment processing"""
    
    def __init__(self, client_id: str, client_secret: str):
        self.client_id = client_id
        self.client_secret = client_secret
    
    def process_payment(self, amount: float, payment_details: Dict[str, Any]) -> PaymentResult:
        try:
            print(f"PayPal: Processing ${amount} payment")
            transaction_id = f"PAY-{hash(str(amount) + str(datetime.now()))}"
            return PaymentResult(True, transaction_id)
        except Exception as e:
            return PaymentResult(False, error_message=str(e))
    
    def refund_payment(self, transaction_id: str, amount: float) -> PaymentResult:
        print(f"PayPal: Refunding ${amount} for transaction {transaction_id}")
        return PaymentResult(True, f"REF-{transaction_id}")

class EmailNotificationService(NotificationService):
    """Email-based notification implementation"""
    
    def __init__(self, smtp_config: Dict[str, str]):
        self.smtp_config = smtp_config
    
    def send_order_confirmation(self, order: Order, contact_info: Dict[str, str]) -> bool:
        email = contact_info.get('email')
        if not email:
            return False
        
        print(f"Email: Sending order confirmation for order #{order.id} to {email}")
        print(f"  Order total: ${order.total}")
        print(f"  Status: {order.status.value}")
        return True
    
    def send_order_update(self, order: Order, contact_info: Dict[str, str]) -> bool:
        email = contact_info.get('email')
        if not email:
            return False
        
        print(f"Email: Sending order update for order #{order.id} to {email}")
        print(f"  New status: {order.status.value}")
        return True

class SMSNotificationService(NotificationService):
    """SMS-based notification implementation"""
    
    def __init__(self, api_config: Dict[str, str]):
        self.api_config = api_config
    
    def send_order_confirmation(self, order: Order, contact_info: Dict[str, str]) -> bool:
        phone = contact_info.get('phone')
        if not phone:
            return False
        
        message = f"Order #{order.id} confirmed. Total: ${order.total}"
        print(f"SMS: Sending to {phone}: {message}")
        return True
    
    def send_order_update(self, order: Order, contact_info: Dict[str, str]) -> bool:
        phone = contact_info.get('phone')
        if not phone:
            return False
        
        message = f"Order #{order.id} status updated to: {order.status.value}"
        print(f"SMS: Sending to {phone}: {message}")
        return True

class SimpleInventoryService(InventoryService):
    """Simple inventory management implementation"""
    
    def __init__(self):
        self.inventory = {
            'item1': 100,
            'item2': 50,
            'item3': 25
        }
    
    def reserve_items(self, items: List[Dict[str, Any]]) -> bool:
        # Check availability first
        for item in items:
            item_id = item['id']
            quantity = item['quantity']
            if self.inventory.get(item_id, 0) < quantity:
                print(f"Insufficient inventory for {item_id}")
                return False
        
        # Reserve items
        for item in items:
            item_id = item['id']
            quantity = item['quantity']
            self.inventory[item_id] -= quantity
            print(f"Reserved {quantity} units of {item_id}")
        
        return True
    
    def release_items(self, items: List[Dict[str, Any]]) -> bool:
        for item in items:
            item_id = item['id']
            quantity = item['quantity']
            self.inventory[item_id] = self.inventory.get(item_id, 0) + quantity
            print(f"Released {quantity} units of {item_id}")
        
        return True

# High-level business logic
class OrderService:
    """High-level order processing service that depends on abstractions"""
    
    def __init__(
        self,
        order_repository: OrderRepository,
        payment_processor: PaymentProcessor,
        notification_service: NotificationService,
        inventory_service: InventoryService
    ):
        self.order_repository = order_repository
        self.payment_processor = payment_processor
        self.notification_service = notification_service
        self.inventory_service = inventory_service
    
    def process_order(self, order_data: Dict[str, Any]) -> Dict[str, Any]:
        """Pure business logic for order processing"""
        
        try:
            # Validate order data
            if not self._validate_order_data(order_data):
                return {'success': False, 'error': 'Invalid order data'}
            
            # Create order object
            order = Order(
                customer_id=order_data['customer_id'],
                total=order_data['total'],
                items=order_data['items']
            )
            
            # Reserve inventory
            if not self.inventory_service.reserve_items(order.items):
                return {'success': False, 'error': 'Insufficient inventory'}
            
            try:
                # Process payment
                payment_result = self.payment_processor.process_payment(
                    order.total,
                    order_data['payment_details']
                )
                
                if not payment_result.success:
                    # Release reserved inventory on payment failure
                    self.inventory_service.release_items(order.items)
                    return {
                        'success': False, 
                        'error': f'Payment failed: {payment_result.error_message}'
                    }
                
                # Update order with payment info
                order.payment_id = payment_result.transaction_id
                order.status = OrderStatus.CONFIRMED
                
                # Save order
                order_id = self.order_repository.save_order(order)
                
                # Send confirmation
                self.notification_service.send_order_confirmation(
                    order, 
                    order_data['contact_info']
                )
                
                return {
                    'success': True,
                    'order_id': order_id,
                    'transaction_id': payment_result.transaction_id
                }
                
            except Exception as e:
                # Release inventory on any error after reservation
                self.inventory_service.release_items(order.items)
                raise e
                
        except Exception as e:
            return {'success': False, 'error': str(e)}
    
    def cancel_order(self, order_id: int) -> Dict[str, Any]:
        """Cancel an order and handle refunds"""
        
        order = self.order_repository.get_order(order_id)
        if not order:
            return {'success': False, 'error': 'Order not found'}
        
        if order.status != OrderStatus.CONFIRMED:
            return {'success': False, 'error': 'Order cannot be cancelled'}
        
        try:
            # Process refund
            if order.payment_id:
                refund_result = self.payment_processor.refund_payment(
                    order.payment_id, 
                    order.total
                )
                
                if not refund_result.success:
                    return {
                        'success': False, 
                        'error': f'Refund failed: {refund_result.error_message}'
                    }
            
            # Release inventory
            self.inventory_service.release_items(order.items)
            
            # Update order status
            self.order_repository.update_order_status(order_id, OrderStatus.CANCELLED)
            
            # Send notification (would need contact info in real implementation)
            # self.notification_service.send_order_update(order, contact_info)
            
            return {
                'success': True,
                'refund_id': refund_result.transaction_id if order.payment_id else None
            }
            
        except Exception as e:
            return {'success': False, 'error': str(e)}
    
    def _validate_order_data(self, order_data: Dict[str, Any]) -> bool:
        """Validate order data"""
        required_fields = ['customer_id', 'total', 'items', 'payment_details', 'contact_info']
        
        if not all(field in order_data for field in required_fields):
            return False
        
        if order_data['total'] <= 0:
            return False
        
        if not order_data['items']:
            return False
        
        return True

# Dependency injection container
class OrderProcessingContainer:
    """Container for managing dependencies"""
    
    def __init__(self, config: Dict[str, Any]):
        self.config = config
        self._instances = {}
    
    def get_order_repository(self) -> OrderRepository:
        if 'order_repository' not in self._instances:
            db_config = self.config.get('database', {})
            provider = db_config.get('provider', 'sqlite')
            
            if provider == 'sqlite':
                self._instances['order_repository'] = SQLiteOrderRepository(
                    db_config.get('path', 'orders.db')
                )
            elif provider == 'memory':
                self._instances['order_repository'] = InMemoryOrderRepository()
            else:
                raise ValueError(f"Unknown database provider: {provider}")
        
        return self._instances['order_repository']
    
    def get_payment_processor(self) -> PaymentProcessor:
        if 'payment_processor' not in self._instances:
            payment_config = self.config.get('payment', {})
            provider = payment_config.get('provider', 'stripe')
            
            if provider == 'stripe':
                self._instances['payment_processor'] = StripePaymentProcessor(
                    payment_config.get('api_key')
                )
            elif provider == 'paypal':
                self._instances['payment_processor'] = PayPalPaymentProcessor(
                    payment_config.get('client_id'),
                    payment_config.get('client_secret')
                )
            else:
                raise ValueError(f"Unknown payment provider: {provider}")
        
        return self._instances['payment_processor']
    
    def get_notification_service(self) -> NotificationService:
        if 'notification_service' not in self._instances:
            notification_config = self.config.get('notifications', {})
            provider = notification_config.get('provider', 'email')
            
            if provider == 'email':
                self._instances['notification_service'] = EmailNotificationService(
                    notification_config.get('smtp', {})
                )
            elif provider == 'sms':
                self._instances['notification_service'] = SMSNotificationService(
                    notification_config.get('api', {})
                )
            else:
                raise ValueError(f"Unknown notification provider: {provider}")
        
        return self._instances['notification_service']
    
    def get_inventory_service(self) -> InventoryService:
        if 'inventory_service' not in self._instances:
            self._instances['inventory_service'] = SimpleInventoryService()
        
        return self._instances['inventory_service']
    
    def get_order_service(self) -> OrderService:
        if 'order_service' not in self._instances:
            self._instances['order_service'] = OrderService(
                self.get_order_repository(),
                self.get_payment_processor(),
                self.get_notification_service(),
                self.get_inventory_service()
            )
        
        return self._instances['order_service']

# Mock implementations for testing
class MockPaymentProcessor(PaymentProcessor):
    def __init__(self, should_succeed: bool = True):
        self.should_succeed = should_succeed
        self.processed_payments = []
        self.refunded_payments = []
    
    def process_payment(self, amount: float, payment_details: Dict[str, Any]) -> PaymentResult:
        self.processed_payments.append({'amount': amount, 'details': payment_details})
        
        if self.should_succeed:
            return PaymentResult(True, f"mock_tx_{len(self.processed_payments)}")
        else:
            return PaymentResult(False, error_message="Mock payment failure")
    
    def refund_payment(self, transaction_id: str, amount: float) -> PaymentResult:
        self.refunded_payments.append({'transaction_id': transaction_id, 'amount': amount})
        return PaymentResult(True, f"mock_refund_{len(self.refunded_payments)}")

class MockNotificationService(NotificationService):
    def __init__(self):
        self.sent_confirmations = []
        self.sent_updates = []
    
    def send_order_confirmation(self, order: Order, contact_info: Dict[str, str]) -> bool:
        self.sent_confirmations.append({'order_id': order.id, 'contact_info': contact_info})
        return True
    
    def send_order_update(self, order: Order, contact_info: Dict[str, str]) -> bool:
        self.sent_updates.append({'order_id': order.id, 'contact_info': contact_info})
        return True

# Usage and testing
def main():
    """Production usage example"""
    
    config = {
        'database': {
            'provider': 'sqlite',
            'path': 'orders.db'
        },
        'payment': {
            'provider': 'stripe',
            'api_key': 'sk_test_123'
        },
        'notifications': {
            'provider': 'email',
            'smtp': {
                'server': 'smtp.gmail.com',
                'port': 587,
                'username': 'user@gmail.com',
                'password': 'password'
            }
        }
    }
    
    container = OrderProcessingContainer(config)
    order_service = container.get_order_service()
    
    order_data = {
        'customer_id': 123,
        'total': 99.99,
        'items': [
            {'id': 'item1', 'quantity': 2, 'price': 49.995}
        ],
        'payment_details': {
            'card_token': 'tok_visa'
        },
        'contact_info': {
            'email': 'customer@example.com',
            'phone': '+1234567890'
        }
    }
    
    result = order_service.process_order(order_data)
    print(f"Order processing result: {result}")

def test_order_processing():
    """Testing example with mocked dependencies"""
    
    # Create mock dependencies
    order_repository = InMemoryOrderRepository()
    payment_processor = MockPaymentProcessor(should_succeed=True)
    notification_service = MockNotificationService()
    inventory_service = SimpleInventoryService()
    
    # Inject dependencies
    order_service = OrderService(
        order_repository,
        payment_processor,
        notification_service,
        inventory_service
    )
    
    # Test successful order processing
    order_data = {
        'customer_id': 123,
        'total': 99.99,
        'items': [
            {'id': 'item1', 'quantity': 1, 'price': 99.99}
        ],
        'payment_details': {
            'card_token': 'tok_visa'
        },
        'contact_info': {
            'email': 'test@example.com'
        }
    }
    
    result = order_service.process_order(order_data)
    
    # Verify results
    assert result['success'] == True
    assert 'order_id' in result
    assert len(payment_processor.processed_payments) == 1
    assert len(notification_service.sent_confirmations) == 1
    
    # Test order cancellation
    order_id = result['order_id']
    cancel_result = order_service.cancel_order(order_id)
    
    assert cancel_result['success'] == True
    assert len(payment_processor.refunded_payments) == 1
    
    print("All tests passed!")

def test_payment_failure():
    """Test payment failure scenario"""
    
    # Create dependencies with failing payment processor
    order_repository = InMemoryOrderRepository()
    payment_processor = MockPaymentProcessor(should_succeed=False)
    notification_service = MockNotificationService()
    inventory_service = SimpleInventoryService()
    
    order_service = OrderService(
        order_repository,
        payment_processor,
        notification_service,
        inventory_service
    )
    
    order_data = {
        'customer_id': 123,
        'total': 99.99,
        'items': [
            {'id': 'item1', 'quantity': 1, 'price': 99.99}
        ],
        'payment_details': {
            'card_token': 'tok_fail'
        },
        'contact_info': {
            'email': 'test@example.com'
        }
    }
    
    result = order_service.process_order(order_data)
    
    # Verify failure handling
    assert result['success'] == False
    assert 'Payment failed' in result['error']
    assert len(notification_service.sent_confirmations) == 0
    
    # Verify inventory was released
    assert inventory_service.inventory['item1'] == 100  # Should be back to original
    
    print("Payment failure test passed!")

if __name__ == "__main__":
    print("=== Production Usage ===")
    main()
    
    print("\n=== Testing Successful Order ===")
    test_order_processing()
    
    print("\n=== Testing Payment Failure ===")
    test_payment_failure()
```

### Example 3: File Processing System

#### Violating DIP

```python
# BAD: Violates DIP - File processing system with tight coupling
import os
import json
import csv
import xml.etree.ElementTree as ET
from typing import Dict, Any, List

class LocalFileSystem:
    """Low-level file system operations"""
    
    def read_file(self, file_path: str) -> str:
        with open(file_path, 'r', encoding='utf-8') as file:
            return file.read()
    
    def write_file(self, file_path: str, content: str) -> bool:
        try:
            with open(file_path, 'w', encoding='utf-8') as file:
                file.write(content)
            return True
        except Exception as e:
            print(f"Error writing file: {e}")
            return False
    
    def file_exists(self, file_path: str) -> bool:
        return os.path.exists(file_path)

class JSONParser:
    """Low-level JSON parsing"""
    
    def parse(self, content: str) -> Dict[str, Any]:
        return json.loads(content)
    
    def serialize(self, data: Dict[str, Any]) -> str:
        return json.dumps(data, indent=2)

class CSVParser:
    """Low-level CSV parsing"""
    
    def parse(self, content: str) -> List[Dict[str, Any]]:
        lines = content.strip().split('\n')
        if not lines:
            return []
        
        headers = lines[0].split(',')
        result = []
        
        for line in lines[1:]:
            values = line.split(',')
            record = dict(zip(headers, values))
            result.append(record)
        
        return result
    
    def serialize(self, data: List[Dict[str, Any]]) -> str:
        if not data:
            return ""
        
        headers = list(data[0].keys())
        lines = [','.join(headers)]
        
        for record in data:
            values = [str(record.get(header, '')) for header in headers]
            lines.append(','.join(values))
        
        return '\n'.join(lines)

class EmailSender:
    """Low-level email sending"""
    
    def __init__(self, smtp_server: str, username: str, password: str):
        self.smtp_server = smtp_server
        self.username = username
        self.password = password
    
    def send_email(self, to: str, subject: str, body: str) -> bool:
        print(f"Sending email to {to}: {subject}")
        return True

class FileProcessingService:
    """High-level service tightly coupled to low-level implementations"""
    
    def __init__(self):
        # Problem: Direct dependencies on concrete implementations
        self.file_system = LocalFileSystem()
        self.json_parser = JSONParser()
        self.csv_parser = CSVParser()
        self.email_sender = EmailSender("smtp.gmail.com", "user@gmail.com", "password")
    
    def process_configuration_file(self, config_path: str) -> bool:
        """Process configuration files - tightly coupled to JSON and local files"""
        
        try:
            # Problem: Directly uses local file system
            if not self.file_system.file_exists(config_path):
                print(f"Configuration file {config_path} not found")
                return False
            
            # Problem: Assumes JSON format
            content = self.file_system.read_file(config_path)
            config = self.json_parser.parse(content)
            
            # Process configuration
            self._apply_configuration(config)
            
            # Problem: Direct email dependency
            self.email_sender.send_email(
                "admin@company.com",
                "Configuration Updated",
                f"Configuration from {config_path} has been processed"
            )
            
            return True
            
        except Exception as e:
            print(f"Error processing configuration: {e}")
            return False
    
    def process_data_export(self, data: List[Dict[str, Any]], output_path: str) -> bool:
        """Export data - tightly coupled to CSV and local files"""
        
        try:
            # Problem: Assumes CSV format and local file system
            csv_content = self.csv_parser.serialize(data)
            
            success = self.file_system.write_file(output_path, csv_content)
            
            if success:
                # Problem: Direct email dependency
                self.email_sender.send_email(
                    "admin@company.com",
                    "Data Export Complete",
                    f"Data has been exported to {output_path}"
                )
            
            return success
            
        except Exception as e:
            print(f"Error exporting data: {e}")
            return False
    
    def _apply_configuration(self, config: Dict[str, Any]) -> None:
        print(f"Applying configuration: {config}")

# Usage - inflexible and hard to test
def main():
    processor = FileProcessingService()
    
    # Problems:
    # - Cannot easily test without creating actual files
    # - Cannot easily switch to cloud storage or different file formats
    # - Cannot easily mock email sending for testing
    # - Cannot easily change notification methods
    
    # Process configuration
    processor.process_configuration_file("config.json")
    
    # Export data
    sample_data = [
        {"name": "John", "age": "30", "city": "New York"},
        {"name": "Jane", "age": "25", "city": "San Francisco"}
    ]
    processor.process_data_export(sample_data, "export.csv")

if __name__ == "__main__":
    main()
```

#### Following DIP

```python
# GOOD: Following DIP - Depend on abstractions
from abc import ABC, abstractmethod
from typing import Dict, Any, List, Optional
import os
import json
import csv
import xml.etree.ElementTree as ET
from datetime import datetime

# Abstractions for file operations
class FileStorage(ABC):
    """Abstraction for file storage operations"""
    
    @abstractmethod
    def read_file(self, file_path: str) -> str:
        pass
    
    @abstractmethod
    def write_file(self, file_path: str, content: str) -> bool:
        pass
    
    @abstractmethod
    def file_exists(self, file_path: str) -> bool:
        pass
    
    @abstractmethod
    def list_files(self, directory: str) -> List[str]:
        pass

class DataParser(ABC):
    """Abstraction for data parsing"""
    
    @abstractmethod
    def parse(self, content: str) -> Any:
        pass
    
    @abstractmethod
    def serialize(self, data: Any) -> str:
        pass
    
    @property
    @abstractmethod
    def supported_extensions(self) -> List[str]:
        pass

class NotificationSender(ABC):
    """Abstraction for sending notifications"""
    
    @abstractmethod
    def send_notification(self, recipient: str, subject: str, message: str) -> bool:
        pass

class Logger(ABC):
    """Abstraction for logging"""
    
    @abstractmethod
    def log_info(self, message: str) -> None:
        pass
    
    @abstractmethod
    def log_error(self, message: str) -> None:
        pass
    
    @abstractmethod
    def log_warning(self, message: str) -> None:
        pass

# Concrete implementations
class LocalFileStorage(FileStorage):
    """Local file system implementation"""
    
    def read_file(self, file_path: str) -> str:
        with open(file_path, 'r', encoding='utf-8') as file:
            return file.read()
    
    def write_file(self, file_path: str, content: str) -> bool:
        try:
            os.makedirs(os.path.dirname(file_path), exist_ok=True)
            with open(file_path, 'w', encoding='utf-8') as file:
                file.write(content)
            return True
        except Exception as e:
            print(f"Error writing file: {e}")
            return False
    
    def file_exists(self, file_path: str) -> bool:
        return os.path.exists(file_path)
    
    def list_files(self, directory: str) -> List[str]:
        if not os.path.exists(directory):
            return []
        return [f for f in os.listdir(directory) if os.path.isfile(os.path.join(directory, f))]

class CloudFileStorage(FileStorage):
    """Cloud storage implementation (simulated)"""
    
    def __init__(self, bucket_name: str, access_key: str):
        self.bucket_name = bucket_name
        self.access_key = access_key
        self.files = {}  # Simulated cloud storage
    
    def read_file(self, file_path: str) -> str:
        if file_path in self.files:
            print(f"Cloud: Reading file {file_path}")
            return self.files[file_path]
        raise FileNotFoundError(f"File {file_path} not found in cloud storage")
    
    def write_file(self, file_path: str, content: str) -> bool:
        try:
            self.files[file_path] = content
            print(f"Cloud: Wrote file {file_path}")
            return True
        except Exception as e:
            print(f"Cloud error: {e}")
            return False
    
    def file_exists(self, file_path: str) -> bool:
        return file_path in self.files
    
    def list_files(self, directory: str) -> List[str]:
        return [f for f in self.files.keys() if f.startswith(directory)]

class JSONDataParser(DataParser):
    """JSON parsing implementation"""
    
    def parse(self, content: str) -> Dict[str, Any]:
        return json.loads(content)
    
    def serialize(self, data: Dict[str, Any]) -> str:
        return json.dumps(data, indent=2)
    
    @property
    def supported_extensions(self) -> List[str]:
        return ['.json']

class CSVDataParser(DataParser):
    """CSV parsing implementation"""
    
    def parse(self, content: str) -> List[Dict[str, Any]]:
        lines = content.strip().split('\n')
        if not lines:
            return []
        
        headers = [h.strip() for h in lines[0].split(',')]
        result = []
        
        for line in lines[1:]:
            values = [v.strip() for v in line.split(',')]
            record = dict(zip(headers, values))
            result.append(record)
        
        return result
    
    def serialize(self, data: List[Dict[str, Any]]) -> str:
        if not data:
            return ""
        
        headers = list(data[0].keys())
        lines = [','.join(headers)]
        
        for record in data:
            values = [str(record.get(header, '')) for header in headers]
            lines.append(','.join(values))
        
        return '\n'.join(lines)
    
    @property
    def supported_extensions(self) -> List[str]:
        return ['.csv']

class XMLDataParser(DataParser):
    """XML parsing implementation"""
    
    def parse(self, content: str) -> Dict[str, Any]:
        root = ET.fromstring(content)
        return self._xml_to_dict(root)
    
    def serialize(self, data: Dict[str, Any]) -> str:
        root = self._dict_to_xml(data, 'root')
        return ET.tostring(root, encoding='unicode')
    
    def _xml_to_dict(self, element) -> Dict[str, Any]:
        result = {}
        for child in element:
            if len(child) == 0:
                result[child.tag] = child.text
            else:
                result[child.tag] = self._xml_to_dict(child)
        return result
    
    def _dict_to_xml(self, data: Dict[str, Any], root_tag: str):
        root = ET.Element(root_tag)
        for key, value in data.items():
            child = ET.SubElement(root, key)
            if isinstance(value, dict):
                child.extend([self._dict_to_xml(value, subkey) for subkey in value])
            else:
                child.text = str(value)
        return root
    
    @property
    def supported_extensions(self) -> List[str]:
        return ['.xml']

class EmailNotificationSender(NotificationSender):
    """Email notification implementation"""
    
    def __init__(self, smtp_config: Dict[str, str]):
        self.smtp_config = smtp_config
    
    def send_notification(self, recipient: str, subject: str, message: str) -> bool:
        print(f"Email: Sending to {recipient}")
        print(f"  Subject: {subject}")
        print(f"  Message: {message}")
        return True

class SlackNotificationSender(NotificationSender):
    """Slack notification implementation"""
    
    def __init__(self, webhook_url: str):
        self.webhook_url = webhook_url
    
    def send_notification(self, recipient: str, subject: str, message: str) -> bool:
        print(f"Slack: Sending to {recipient}")
        print(f"  {subject}: {message}")
        return True

class ConsoleLogger(Logger):
    """Console logging implementation"""
    
    def log_info(self, message: str) -> None:
        print(f"[INFO] {datetime.now().isoformat()}: {message}")
    
    def log_error(self, message: str) -> None:
        print(f"[ERROR] {datetime.now().isoformat()}: {message}")
    
    def log_warning(self, message: str) -> None:
        print(f"[WARNING] {datetime.now().isoformat()}: {message}")

class FileLogger(Logger):
    """File logging implementation"""
    
    def __init__(self, log_file: str):
        self.log_file = log_file
    
    def _write_log(self, level: str, message: str) -> None:
        log_entry = f"[{level}] {datetime.now().isoformat()}: {message}\n"
        with open(self.log_file, 'a', encoding='utf-8') as f:
            f.write(log_entry)
    
    def log_info(self, message: str) -> None:
        self._write_log("INFO", message)
    
    def log_error(self, message: str) -> None:
        self._write_log("ERROR", message)
    
    def log_warning(self, message: str) -> None:
        self._write_log("WARNING", message)

# High-level business logic
class FileProcessingService:
    """High-level file processing service that depends on abstractions"""
    
    def __init__(
        self,
        file_storage: FileStorage,
        parsers: Dict[str, DataParser],
        notification_sender: NotificationSender,
        logger: Logger
    ):
        self.file_storage = file_storage
        self.parsers = parsers
        self.notification_sender = notification_sender
        self.logger = logger
    
    def process_configuration_file(self, config_path: str, admin_email: str) -> bool:
        """Process configuration files using injected dependencies"""
        
        try:
            self.logger.log_info(f"Processing configuration file: {config_path}")
            
            # Check if file exists
            if not self.file_storage.file_exists(config_path):
                error_msg = f"Configuration file {config_path} not found"
                self.logger.log_error(error_msg)
                return False
            
            # Read file content
            content = self.file_storage.read_file(config_path)
            
            # Determine parser based on file extension
            parser = self._get_parser_for_file(config_path)
            if not parser:
                error_msg = f"No parser available for file: {config_path}"
                self.logger.log_error(error_msg)
                return False
            
            # Parse configuration
            config = parser.parse(content)
            
            # Apply configuration
            self._apply_configuration(config)
            
            # Send notification
            success_msg = f"Configuration from {config_path} has been processed successfully"
            self.notification_sender.send_notification(
                admin_email,
                "Configuration Updated",
                success_msg
            )
            
            self.logger.log_info("Configuration processing completed successfully")
            return True
            
        except Exception as e:
            error_msg = f"Error processing configuration: {str(e)}"
            self.logger.log_error(error_msg)
            return False
    
    def export_data(
        self, 
        data: List[Dict[str, Any]], 
        output_path: str, 
        admin_email: str
    ) -> bool:
        """Export data using injected dependencies"""
        
        try:
            self.logger.log_info(f"Exporting data to: {output_path}")
            
            # Determine parser based on file extension
            parser = self._get_parser_for_file(output_path)
            if not parser:
                error_msg = f"No parser available for output file: {output_path}"
                self.logger.log_error(error_msg)
                return False
            
            # Serialize data
            content = parser.serialize(data)
            
            # Write file
            success = self.file_storage.write_file(output_path, content)
            
            if success:
                success_msg = f"Data has been exported to {output_path}"
                self.notification_sender.send_notification(
                    admin_email,
                    "Data Export Complete",
                    success_msg
                )
                self.logger.log_info("Data export completed successfully")
            else:
                error_msg = "Failed to write export file"
                self.logger.log_error(error_msg)
            
            return success
            
        except Exception as e:
            error_msg = f"Error exporting data: {str(e)}"
            self.logger.log_error(error_msg)
            return False
    
    def batch_process_files(
        self, 
        directory: str, 
        admin_email: str,
        file_pattern: Optional[str] = None
    ) -> Dict[str, bool]:
        """Process multiple files in a directory"""
        
        self.logger.log_info(f"Starting batch processing of directory: {directory}")
        
        results = {}
        
        try:
            # List files in directory
            files = self.file_storage.list_files(directory)
            
            # Filter files if pattern provided
            if file_pattern:
                files = [f for f in files if file_pattern in f]
            
            self.logger.log_info(f"Found {len(files)} files to process")
            
            # Process each file
            for file_name in files:
                file_path = os.path.join(directory, file_name)
                
                if self._can_process_file(file_path):
                    success = self.process_configuration_file(file_path, admin_email)
                    results[file_name] = success
                else:
                    self.logger.log_warning(f"Skipping unsupported file: {file_name}")
                    results[file_name] = False
            
            # Send summary notification
            successful = sum(1 for success in results.values() if success)
            total = len(results)
            
            summary_msg = f"Batch processing complete: {successful}/{total} files processed successfully"
            self.notification_sender.send_notification(
                admin_email,
                "Batch Processing Complete",
                summary_msg
            )
            
            self.logger.log_info(summary_msg)
            
        except Exception as e:
            error_msg = f"Error during batch processing: {str(e)}"
            self.logger.log_error(error_msg)
        
        return results
    
    def _get_parser_for_file(self, file_path: str) -> Optional[DataParser]:
        """Get appropriate parser based on file extension"""
        _, ext = os.path.splitext(file_path.lower())
        
        for parser in self.parsers.values():
            if ext in parser.supported_extensions:
                return parser
        
        return None
    
    def _can_process_file(self, file_path: str) -> bool:
        """Check if file can be processed"""
        return self._get_parser_for_file(file_path) is not None
    
    def _apply_configuration(self, config: Any) -> None:
        """Apply configuration - pure business logic"""
        self.logger.log_info(f"Applying configuration with {len(config) if hasattr(config, '__len__') else 'unknown'} settings")
        # Business logic for applying configuration

# Dependency injection container
class FileProcessingContainer:
    """Container for managing file processing dependencies"""
    
    def __init__(self, config: Dict[str, Any]):
        self.config = config
        self._instances = {}
    
    def get_file_storage(self) -> FileStorage:
        if 'file_storage' not in self._instances:
            storage_config = self.config.get('storage', {})
            provider = storage_config.get('provider', 'local')
            
            if provider == 'local':
                self._instances['file_storage'] = LocalFileStorage()
            elif provider == 'cloud':
                self._instances['file_storage'] = CloudFileStorage(
                    storage_config.get('bucket_name'),
                    storage_config.get('access_key')
                )
            else:
                raise ValueError(f"Unknown storage provider: {provider}")
        
        return self._instances['file_storage']
    
    def get_parsers(self) -> Dict[str, DataParser]:
        if 'parsers' not in self._instances:
            self._instances['parsers'] = {
                'json': JSONDataParser(),
                'csv': CSVDataParser(),
                'xml': XMLDataParser()
            }
        
        return self._instances['parsers']
    
    def get_notification_sender(self) -> NotificationSender:
        if 'notification_sender' not in self._instances:
            notification_config = self.config.get('notifications', {})
            provider = notification_config.get('provider', 'email')
            
            if provider == 'email':
                self._instances['notification_sender'] = EmailNotificationSender(
                    notification_config.get('smtp', {})
                )
            elif provider == 'slack':
                self._instances['notification_sender'] = SlackNotificationSender(
                    notification_config.get('webhook_url')
                )
            else:
                raise ValueError(f"Unknown notification provider: {provider}")
        
        return self._instances['notification_sender']
    
    def get_logger(self) -> Logger:
        if 'logger' not in self._instances:
            logging_config = self.config.get('logging', {})
            provider = logging_config.get('provider', 'console')
            
            if provider == 'console':
                self._instances['logger'] = ConsoleLogger()
            elif provider == 'file':
                self._instances['logger'] = FileLogger(
                    logging_config.get('file_path', 'app.log')
                )
            else:
                raise ValueError(f"Unknown logging provider: {provider}")
        
        return self._instances['logger']
    
    def get_file_processing_service(self) -> FileProcessingService:
        if 'file_processing_service' not in self._instances:
            self._instances['file_processing_service'] = FileProcessingService(
                self.get_file_storage(),
                self.get_parsers(),
                self.get_notification_sender(),
                self.get_logger()
            )
        
        return self._instances['file_processing_service']

# Mock implementations for testing
class InMemoryFileStorage(FileStorage):
    """In-memory file storage for testing"""
    
    def __init__(self):
        self.files = {}
    
    def read_file(self, file_path: str) -> str:
        if file_path not in self.files:
            raise FileNotFoundError(f"File {file_path} not found")
        return self.files[file_path]
    
    def write_file(self, file_path: str, content: str) -> bool:
        self.files[file_path] = content
        return True
    
    def file_exists(self, file_path: str) -> bool:
        return file_path in self.files
    
    def list_files(self, directory: str) -> List[str]:
        return [f for f in self.files.keys() if f.startswith(directory)]

class MockNotificationSender(NotificationSender):
    """Mock notification sender for testing"""
    
    def __init__(self):
        self.sent_notifications = []
    
    def send_notification(self, recipient: str, subject: str, message: str) -> bool:
        self.sent_notifications.append({
            'recipient': recipient,
            'subject': subject,
            'message': message
        })
        return True

class MockLogger(Logger):
    """Mock logger for testing"""
    
    def __init__(self):
        self.logs = []
    
    def log_info(self, message: str) -> None:
        self.logs.append(('INFO', message))
    
    def log_error(self, message: str) -> None:
        self.logs.append(('ERROR', message))
    
    def log_warning(self, message: str) -> None:
        self.logs.append(('WARNING', message))

# Usage and testing
def main():
    """Production usage example"""
    
    config = {
        'storage': {
            'provider': 'local'
        },
        'notifications': {
            'provider': 'email',
            'smtp': {
                'server': 'smtp.gmail.com',
                'username': 'user@gmail.com',
                'password': 'password'
            }
        },
        'logging': {
            'provider': 'console'
        }
    }
    
    container = FileProcessingContainer(config)
    service = container.get_file_processing_service()
    
    # Create a sample configuration file
    sample_config = {'database_url': 'localhost:5432', 'debug': True}
    file_storage = container.get_file_storage()
    json_parser = container.get_parsers()['json']
    
    config_content = json_parser.serialize(sample_config)
    file_storage.write_file('config.json', config_content)
    
    # Process configuration
    success = service.process_configuration_file('config.json', 'admin@company.com')
    print(f"Configuration processing result: {success}")
    
    # Export sample data
    sample_data = [
        {'name': 'John', 'age': '30', 'city': 'New York'},
        {'name': 'Jane', 'age': '25', 'city': 'San Francisco'}
    ]
    
    export_success = service.export_data(sample_data, 'export.csv', 'admin@company.com')
    print(f"Data export result: {export_success}")

def test_file_processing():
    """Testing example with mocked dependencies"""
    
    # Create mock dependencies
    file_storage = InMemoryFileStorage()
    parsers = {
        'json': JSONDataParser(),
        'csv': CSVDataParser()
    }
    notification_sender = MockNotificationSender()
    logger = MockLogger()
    
    # Create service with injected dependencies
    service = FileProcessingService(file_storage, parsers, notification_sender, logger)
    
    # Setup test data
    test_config = {'setting1': 'value1', 'setting2': 'value2'}
    config_content = json.dumps(test_config)
    file_storage.write_file('test_config.json', config_content)
    
    # Test configuration processing
    success = service.process_configuration_file('test_config.json', 'test@example.com')
    
    # Verify results
    assert success == True
    assert len(notification_sender.sent_notifications) == 1
    assert notification_sender.sent_notifications[0]['recipient'] == 'test@example.com'
    assert len([log for log in logger.logs if log[0] == 'INFO']) > 0
    
    # Test data export
    test_data = [{'col1': 'val1', 'col2': 'val2'}]
    export_success = service.export_data(test_data, 'test_export.csv', 'test@example.com')
    
    assert export_success == True
    assert file_storage.file_exists('test_export.csv')
    assert len(notification_sender.sent_notifications) == 2
    
    print("All file processing tests passed!")

if __name__ == "__main__":
    print("=== Production Usage ===")
    main()
    
    print("\n=== Testing ===")
    test_file_processing()
```

## Dependency Injection Patterns

### 1. Constructor Injection
Dependencies are provided through the constructor.

```python
class OrderService:
    def __init__(self, repository: OrderRepository, payment: PaymentProcessor):
        self.repository = repository
        self.payment = payment
```

### 2. Property Injection
Dependencies are set through properties after object creation.

```python
class OrderService:
    def __init__(self):
        self.repository = None
        self.payment = None
    
    def set_repository(self, repository: OrderRepository):
        self.repository = repository
    
    def set_payment_processor(self, payment: PaymentProcessor):
        self.payment = payment
```

### 3. Method Injection
Dependencies are passed to specific methods that need them.

```python
class OrderService:
    def process_order(self, order_data: dict, payment_processor: PaymentProcessor):
        # Use payment_processor for this specific operation
        pass
```

### 4. Service Locator Pattern
```python
class ServiceLocator:
    _services = {}
    
    @classmethod
    def register(cls, interface, implementation):
        cls._services[interface] = implementation
    
    @classmethod
    def get(cls, interface):
        return cls._services.get(interface)

class OrderService:
    def __init__(self):
        self.repository = ServiceLocator.get(OrderRepository)
        self.payment = ServiceLocator.get(PaymentProcessor)
```

### 5. Dependency Injection Container
```python
class DIContainer:
    def __init__(self):
        self._services = {}
        self._singletons = {}
    
    def register_transient(self, interface, implementation):
        self._services[interface] = ('transient', implementation)
    
    def register_singleton(self, interface, implementation):
        self._services[interface] = ('singleton', implementation)
    
    def resolve(self, interface):
        if interface not in self._services:
            raise ValueError(f"Service {interface} not registered")
        
        service_type, implementation = self._services[interface]
        
        if service_type == 'singleton':
            if interface not in self._singletons:
                self._singletons[interface] = implementation()
            return self._singletons[interface]
        else:
            return implementation()
```

## Benefits of Following DIP

### 1. **Improved Testability**
Dependencies can be easily mocked or stubbed for unit testing.

### 2. **Increased Flexibility**
Different implementations can be swapped without changing high-level code.

### 3. **Better Maintainability**
Changes to low-level modules don't affect high-level business logic.

### 4. **Enhanced Reusability**
High-level modules can work with different low-level implementations.

### 5. **Parallel Development**
Teams can work on different layers simultaneously.

### 6. **Technology Independence**
Business logic becomes independent of specific technologies.

### 7. **Reduced Coupling**
Components become loosely coupled and more modular.

### 8. **Easier Configuration**
System behavior can be changed through configuration rather than code changes.

## Common Pitfalls

### 1. **Over-Abstraction**
Creating interfaces for every small dependency, leading to unnecessary complexity.

### 2. **Leaky Abstractions**
Abstractions that expose implementation details defeat the purpose of DIP.

### 3. **Service Locator Anti-Pattern**
Using service locator pattern instead of proper dependency injection.

### 4. **New Keyword Violations**
Using `new` keyword inside classes instead of injecting dependencies.

### 5. **Temporal Coupling**
Dependencies that must be set in a specific order.

### 6. **God Container**
Creating a single container that manages too many dependencies.

### 7. **Circular Dependencies**
Creating dependency cycles that are difficult to resolve.

### 8. **Ignoring Composition Root**
Not having a single place where all dependencies are wired together.

## Best Practices

### 1. **Use Constructor Injection**
Prefer constructor injection for required dependencies.

### 2. **Create Composition Root**
Have a single place where all dependencies are wired together.

### 3. **Keep Interfaces Focused**
Follow Interface Segregation Principle when designing abstractions.

### 4. **Use Factories for Complex Creation**
Use factory patterns when object creation is complex.

### 5. **Avoid Service Locator**
Prefer dependency injection over service locator pattern.

### 6. **Design Stable Abstractions**
Create abstractions that are unlikely to change.

### 7. **Use Dependency Injection Containers**
Use DI containers for complex applications with many dependencies.

### 8. **Document Dependencies**
Clearly document what dependencies a class requires.

### 9. **Test with Mocks**
Always test with mock implementations to verify DIP compliance.

### 10. **Gradual Refactoring**
Introduce DIP gradually when refactoring existing code.

## Measuring DIP Compliance

### Code Metrics to Watch:
- **Concrete Dependencies**: Number of concrete classes referenced by high-level modules
- **New Keywords**: Usage of `new` keyword inside classes
- **Import Statements**: High-level modules importing low-level modules
- **Test Coverage**: Ability to test classes in isolation

### Questions to Ask:
1. Can I test this class without involving external dependencies?
2. Can I easily swap implementations without changing high-level code?
3. Are my abstractions stable and unlikely to change?
4. Do my high-level modules import low-level modules?
5. Can different teams work on different layers independently?

### DIP Validation Checklist:
- [ ] High-level modules depend on abstractions, not concretions
- [ ] Abstractions don't depend on implementation details
- [ ] Dependencies are injected from the outside
- [ ] Classes can be easily unit tested in isolation
- [ ] Different implementations can be swapped without code changes
- [ ] Business logic is separated from infrastructure concerns
- [ ] There's a clear composition root where dependencies are wired

## Conclusion

The Dependency Inversion Principle is fundamental to creating maintainable, testable, and flexible software architectures. By inverting the traditional dependency flow and depending on abstractions rather than concretions, you create systems that are resilient to change and easy to extend.

DIP enables you to separate business logic from infrastructure concerns, making your code more modular and easier to understand. It facilitates testing by allowing dependencies to be easily mocked, and it supports parallel development by allowing teams to work against agreed-upon interfaces.

The key to successful DIP implementation is to identify the right abstractions and establish clear boundaries between high-level policy and low-level implementation details. Start by identifying areas where your code is tightly coupled to specific technologies or frameworks, then gradually introduce abstractions and dependency injection.

Remember that DIP is not just about using interfaces - it's about designing your architecture so that high-level business logic doesn't depend on low-level implementation details. This creates systems that are more maintainable, testable, and adaptable to changing requirements.

When properly applied, DIP leads to architectures that are easier to understand, test, and modify, ultimately resulting in more robust and flexible software systems.