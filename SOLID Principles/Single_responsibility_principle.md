# Single Responsibility Principle (SRP)

## Table of Contents
1. [Introduction](#introduction)
2. [Definition and Theory](#definition-and-theory)
3. [Why SRP Matters](#why-srp-matters)
4. [When to Apply SRP](#when-to-apply-srp)
5. [Consequences of Violating SRP](#consequences-of-violating-srp)
6. [Python Examples](#python-examples)
7. [Benefits of Following SRP](#benefits-of-following-srp)
8. [Common Pitfalls](#common-pitfalls)
9. [Best Practices](#best-practices)
10. [Conclusion](#conclusion)

## Introduction

The Single Responsibility Principle (SRP) is the first principle in the SOLID principles of object-oriented design, introduced by Robert C. Martin (Uncle Bob). It's one of the most fundamental and important principles in software engineering that helps create maintainable, scalable, and robust code.

## Definition and Theory

### Formal Definition
**"A class should have only one reason to change."**

### Alternative Definitions
- A class should have only one responsibility
- A class should have only one job
- A class should serve a single actor or stakeholder

### Core Concept
The principle suggests that every module, class, or function should have responsibility over a single part of the functionality provided by the software. This responsibility should be entirely encapsulated by the class, and all services provided by the class should be narrowly aligned with that responsibility.

## Why SRP Matters

### 1. **Maintainability**
Code becomes easier to understand, modify, and debug when each class has a clear, single purpose.

### 2. **Testability**
Classes with single responsibilities are easier to test because they have fewer dependencies and clearer interfaces.

### 3. **Flexibility**
Changes to one responsibility don't affect other unrelated functionalities.

### 4. **Reusability**
Classes with focused responsibilities can be reused in different contexts more easily.

### 5. **Reduced Coupling**
Lower coupling between different parts of the system makes the code more modular.

## When to Apply SRP

### You Should Apply SRP When:

1. **Designing new classes or modules**
2. **Refactoring existing code**
3. **A class is becoming too large or complex**
4. **Multiple reasons for change exist in a single class**
5. **Different stakeholders are interested in different parts of the same class**
6. **Testing becomes difficult due to multiple responsibilities**

### Warning Signs That SRP is Violated:

- Class names with "And", "Or", "Manager", "Handler" that do multiple things
- Methods that don't relate to the main purpose of the class
- Large classes with many methods
- Frequent changes to the same class for different reasons
- Difficulty in writing unit tests
- High coupling with many other classes

## Consequences of Violating SRP

### 1. **Increased Complexity**
Classes become harder to understand and reason about when they handle multiple responsibilities.

### 2. **Difficult Maintenance**
Changes in one area can unexpectedly break functionality in another area.

### 3. **Poor Testability**
Testing becomes complex when a class has multiple responsibilities with different dependencies.

### 4. **Tight Coupling**
Classes become tightly coupled with multiple other classes, making the system rigid.

### 5. **Reduced Reusability**
Classes with multiple responsibilities are harder to reuse in different contexts.

### 6. **Merge Conflicts**
Multiple developers working on the same class for different reasons leads to more conflicts.

## Python Examples

### Example 1: Violating SRP

```python
# BAD: Violates SRP
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email
        self.is_logged_in = False
    
    # Responsibility 1: User data management
    def update_name(self, new_name):
        self.name = new_name
    
    def update_email(self, new_email):
        self.email = new_email
    
    # Responsibility 2: Authentication
    def login(self, password):
        # Authentication logic
        if self._verify_password(password):
            self.is_logged_in = True
            return True
        return False
    
    def logout(self):
        self.is_logged_in = False
    
    def _verify_password(self, password):
        # Password verification logic
        return password == "secret123"
    
    # Responsibility 3: Database operations
    def save_to_database(self):
        # Database saving logic
        print(f"Saving user {self.name} to database")
    
    def load_from_database(self, user_id):
        # Database loading logic
        print(f"Loading user {user_id} from database")
    
    # Responsibility 4: Email notifications
    def send_welcome_email(self):
        print(f"Sending welcome email to {self.email}")
    
    def send_password_reset_email(self):
        print(f"Sending password reset email to {self.email}")
    
    # Responsibility 5: User validation
    def validate_email(self):
        return "@" in self.email and "." in self.email.split("@")[1]
    
    def validate_name(self):
        return len(self.name) > 0 and self.name.isalpha()
```

### Example 1: Following SRP (Refactored)

```python
# GOOD: Following SRP
class User:
    """Responsible only for user data representation"""
    def __init__(self, name, email):
        self.name = name
        self.email = email
    
    def update_name(self, new_name):
        self.name = new_name
    
    def update_email(self, new_email):
        self.email = new_email

class UserAuthentication:
    """Responsible only for user authentication"""
    def __init__(self):
        self.logged_in_users = set()
    
    def login(self, user, password):
        if self._verify_password(password):
            self.logged_in_users.add(user.email)
            return True
        return False
    
    def logout(self, user):
        self.logged_in_users.discard(user.email)
    
    def is_logged_in(self, user):
        return user.email in self.logged_in_users
    
    def _verify_password(self, password):
        return password == "secret123"

class UserRepository:
    """Responsible only for user data persistence"""
    def save(self, user):
        print(f"Saving user {user.name} to database")
    
    def load(self, user_id):
        print(f"Loading user {user_id} from database")
        # Return User object
    
    def delete(self, user_id):
        print(f"Deleting user {user_id} from database")

class EmailService:
    """Responsible only for email operations"""
    def send_welcome_email(self, user):
        print(f"Sending welcome email to {user.email}")
    
    def send_password_reset_email(self, user):
        print(f"Sending password reset email to {user.email}")
    
    def send_notification(self, user, message):
        print(f"Sending notification to {user.email}: {message}")

class UserValidator:
    """Responsible only for user data validation"""
    @staticmethod
    def validate_email(email):
        return "@" in email and "." in email.split("@")[1]
    
    @staticmethod
    def validate_name(name):
        return len(name) > 0 and name.replace(" ", "").isalpha()
    
    @staticmethod
    def validate_user(user):
        return (UserValidator.validate_email(user.email) and 
                UserValidator.validate_name(user.name))

# Usage example
user = User("John Doe", "john@example.com")
auth_service = UserAuthentication()
user_repo = UserRepository()
email_service = EmailService()
validator = UserValidator()

# Validate user data
if validator.validate_user(user):
    # Save user
    user_repo.save(user)
    
    # Send welcome email
    email_service.send_welcome_email(user)
    
    # Login user
    if auth_service.login(user, "secret123"):
        print("User logged in successfully")
```

### Example 2: Report Generation

#### Violating SRP
```python
# BAD: Violates SRP
class ReportGenerator:
    def __init__(self, data):
        self.data = data
    
    # Responsibility 1: Data processing
    def process_data(self):
        processed = []
        for item in self.data:
            processed.append(item.upper())
        return processed
    
    # Responsibility 2: Data formatting
    def format_as_csv(self, data):
        return ",".join(data)
    
    def format_as_json(self, data):
        import json
        return json.dumps(data)
    
    # Responsibility 3: File operations
    def save_to_file(self, content, filename):
        with open(filename, 'w') as f:
            f.write(content)
    
    # Responsibility 4: Email sending
    def email_report(self, content, recipient):
        print(f"Emailing report to {recipient}")
    
    # Responsibility 5: Data validation
    def validate_data(self):
        return all(isinstance(item, str) for item in self.data)
    
    # Main method that uses all responsibilities
    def generate_and_send_report(self, format_type, filename, recipient):
        if not self.validate_data():
            raise ValueError("Invalid data")
        
        processed_data = self.process_data()
        
        if format_type == "csv":
            formatted_data = self.format_as_csv(processed_data)
        elif format_type == "json":
            formatted_data = self.format_as_json(processed_data)
        else:
            raise ValueError("Unsupported format")
        
        self.save_to_file(formatted_data, filename)
        self.email_report(formatted_data, recipient)
```

#### Following SRP
```python
# GOOD: Following SRP
class DataProcessor:
    """Responsible only for processing raw data"""
    @staticmethod
    def process(data):
        return [item.upper() for item in data]

class DataValidator:
    """Responsible only for data validation"""
    @staticmethod
    def validate(data):
        return all(isinstance(item, str) for item in data)

class DataFormatter:
    """Responsible only for formatting data"""
    @staticmethod
    def to_csv(data):
        return ",".join(data)
    
    @staticmethod
    def to_json(data):
        import json
        return json.dumps(data)

class FileManager:
    """Responsible only for file operations"""
    @staticmethod
    def save_to_file(content, filename):
        with open(filename, 'w') as f:
            f.write(content)
    
    @staticmethod
    def read_from_file(filename):
        with open(filename, 'r') as f:
            return f.read()

class EmailSender:
    """Responsible only for sending emails"""
    @staticmethod
    def send_report(content, recipient):
        print(f"Emailing report to {recipient}")
        # Email sending logic here

class ReportService:
    """Orchestrates the report generation process"""
    def __init__(self):
        self.data_processor = DataProcessor()
        self.data_validator = DataValidator()
        self.data_formatter = DataFormatter()
        self.file_manager = FileManager()
        self.email_sender = EmailSender()
    
    def generate_and_send_report(self, data, format_type, filename, recipient):
        # Validate data
        if not self.data_validator.validate(data):
            raise ValueError("Invalid data")
        
        # Process data
        processed_data = self.data_processor.process(data)
        
        # Format data
        if format_type == "csv":
            formatted_data = self.data_formatter.to_csv(processed_data)
        elif format_type == "json":
            formatted_data = self.data_formatter.to_json(processed_data)
        else:
            raise ValueError("Unsupported format")
        
        # Save to file
        self.file_manager.save_to_file(formatted_data, filename)
        
        # Send email
        self.email_sender.send_report(formatted_data, recipient)

# Usage
data = ["hello", "world", "python"]
report_service = ReportService()
report_service.generate_and_send_report(data, "csv", "report.csv", "user@example.com")
```

### Example 3: E-commerce Order Processing

#### Violating SRP
```python
# BAD: Violates SRP
class Order:
    def __init__(self, order_id, items, customer_email):
        self.order_id = order_id
        self.items = items
        self.customer_email = customer_email
        self.total = 0
        self.status = "pending"
    
    # Responsibility 1: Order calculations
    def calculate_total(self):
        self.total = sum(item['price'] * item['quantity'] for item in self.items)
        return self.total
    
    def apply_discount(self, discount_code):
        if discount_code == "SAVE10":
            self.total *= 0.9
    
    # Responsibility 2: Inventory management
    def check_inventory(self):
        for item in self.items:
            print(f"Checking inventory for {item['name']}")
        return True
    
    def update_inventory(self):
        for item in self.items:
            print(f"Updating inventory for {item['name']}")
    
    # Responsibility 3: Payment processing
    def process_payment(self, payment_info):
        print(f"Processing payment of ${self.total}")
        return True
    
    def refund_payment(self):
        print(f"Refunding payment of ${self.total}")
    
    # Responsibility 4: Email notifications
    def send_confirmation_email(self):
        print(f"Sending order confirmation to {self.customer_email}")
    
    def send_shipping_notification(self):
        print(f"Sending shipping notification to {self.customer_email}")
    
    # Responsibility 5: Database operations
    def save_to_database(self):
        print(f"Saving order {self.order_id} to database")
    
    def update_status(self, new_status):
        self.status = new_status
        self.save_to_database()
    
    # Responsibility 6: Logging
    def log_order_activity(self, activity):
        print(f"LOG: Order {self.order_id} - {activity}")
```

#### Following SRP
```python
# GOOD: Following SRP
from abc import ABC, abstractmethod
from typing import List, Dict

class Order:
    """Responsible only for order data representation"""
    def __init__(self, order_id: str, items: List[Dict], customer_email: str):
        self.order_id = order_id
        self.items = items
        self.customer_email = customer_email
        self.total = 0
        self.status = "pending"

class OrderCalculator:
    """Responsible only for order calculations"""
    @staticmethod
    def calculate_total(order: Order) -> float:
        total = sum(item['price'] * item['quantity'] for item in order.items)
        order.total = total
        return total
    
    @staticmethod
    def apply_discount(order: Order, discount_code: str) -> float:
        if discount_code == "SAVE10":
            order.total *= 0.9
        return order.total

class InventoryService:
    """Responsible only for inventory management"""
    def check_availability(self, items: List[Dict]) -> bool:
        for item in items:
            print(f"Checking inventory for {item['name']}")
        return True
    
    def reserve_items(self, items: List[Dict]) -> bool:
        for item in items:
            print(f"Reserving {item['quantity']} of {item['name']}")
        return True
    
    def update_stock(self, items: List[Dict]) -> None:
        for item in items:
            print(f"Updating inventory for {item['name']}")

class PaymentProcessor:
    """Responsible only for payment processing"""
    def process_payment(self, amount: float, payment_info: Dict) -> bool:
        print(f"Processing payment of ${amount}")
        return True
    
    def refund_payment(self, amount: float, transaction_id: str) -> bool:
        print(f"Refunding payment of ${amount}")
        return True

class EmailNotificationService:
    """Responsible only for email notifications"""
    def send_order_confirmation(self, order: Order) -> None:
        print(f"Sending order confirmation to {order.customer_email}")
    
    def send_shipping_notification(self, order: Order) -> None:
        print(f"Sending shipping notification to {order.customer_email}")
    
    def send_cancellation_email(self, order: Order) -> None:
        print(f"Sending cancellation email to {order.customer_email}")

class OrderRepository:
    """Responsible only for order data persistence"""
    def save(self, order: Order) -> None:
        print(f"Saving order {order.order_id} to database")
    
    def update_status(self, order: Order, new_status: str) -> None:
        order.status = new_status
        self.save(order)
    
    def find_by_id(self, order_id: str) -> Order:
        print(f"Finding order {order_id}")
        # Return order from database

class Logger:
    """Responsible only for logging activities"""
    @staticmethod
    def log_activity(order_id: str, activity: str) -> None:
        print(f"LOG: Order {order_id} - {activity}")

class OrderService:
    """Orchestrates order processing workflow"""
    def __init__(self):
        self.calculator = OrderCalculator()
        self.inventory = InventoryService()
        self.payment_processor = PaymentProcessor()
        self.email_service = EmailNotificationService()
        self.repository = OrderRepository()
        self.logger = Logger()
    
    def process_order(self, order: Order, payment_info: Dict, discount_code: str = None) -> bool:
        try:
            # Calculate total
            self.calculator.calculate_total(order)
            
            # Apply discount if provided
            if discount_code:
                self.calculator.apply_discount(order, discount_code)
            
            # Check inventory
            if not self.inventory.check_availability(order.items):
                self.logger.log_activity(order.order_id, "Inventory check failed")
                return False
            
            # Process payment
            if not self.payment_processor.process_payment(order.total, payment_info):
                self.logger.log_activity(order.order_id, "Payment failed")
                return False
            
            # Reserve inventory
            self.inventory.reserve_items(order.items)
            
            # Update order status
            self.repository.update_status(order, "confirmed")
            
            # Send confirmation email
            self.email_service.send_order_confirmation(order)
            
            # Log success
            self.logger.log_activity(order.order_id, "Order processed successfully")
            
            return True
            
        except Exception as e:
            self.logger.log_activity(order.order_id, f"Error processing order: {str(e)}")
            return False

# Usage
order = Order("ORD001", [
    {"name": "Laptop", "price": 999.99, "quantity": 1},
    {"name": "Mouse", "price": 29.99, "quantity": 2}
], "customer@example.com")

payment_info = {"card_number": "1234-5678-9012-3456", "cvv": "123"}

order_service = OrderService()
success = order_service.process_order(order, payment_info, "SAVE10")
```

## Benefits of Following SRP

### 1. **Enhanced Readability**
Code becomes self-documenting when each class has a clear, single purpose.

### 2. **Easier Testing**
Unit tests become simpler and more focused when testing single responsibilities.

### 3. **Better Organization**
Code structure becomes more logical and easier to navigate.

### 4. **Improved Collaboration**
Team members can work on different responsibilities without conflicts.

### 5. **Easier Debugging**
Issues are easier to isolate when responsibilities are separated.

### 6. **Increased Reusability**
Single-purpose classes can be reused across different contexts.

### 7. **Simplified Maintenance**
Changes to one responsibility don't affect others.

## Common Pitfalls

### 1. **Over-Engineering**
Creating too many small classes can sometimes lead to unnecessary complexity.

### 2. **Misunderstanding "Responsibility"**
Confusing methods with responsibilities. A class can have multiple methods that serve the same responsibility.

### 3. **Premature Optimization**
Applying SRP too early in development when requirements are still unclear.

### 4. **God Classes**
Allowing classes to grow too large before refactoring.

### 5. **Ignoring Context**
Not considering the specific domain and requirements when defining responsibilities.

## Best Practices

### 1. **Start with Domain Understanding**
Understand the business domain to identify natural responsibilities.

### 2. **Use Descriptive Names**
Class names should clearly indicate their single responsibility.

### 3. **Apply the "One Reason to Change" Rule**
If you can think of multiple reasons why a class might change, it probably violates SRP.

### 4. **Regular Refactoring**
Continuously refactor code as understanding of the domain improves.

### 5. **Use Composition Over Inheritance**
Compose behavior from multiple single-responsibility classes.

### 6. **Dependency Injection**
Use dependency injection to manage relationships between single-responsibility classes.

### 7. **Interface Segregation**
Define small, focused interfaces that serve specific clients.

## Measuring SRP Compliance

### Code Metrics to Watch:
- **Lines of Code per Class**: Keep classes reasonably sized
- **Number of Methods**: Too many methods might indicate multiple responsibilities
- **Cyclomatic Complexity**: High complexity often correlates with multiple responsibilities
- **Coupling**: High coupling with many other classes is a warning sign
- **Cohesion**: Low cohesion indicates the class is doing unrelated things

### Questions to Ask:
1. What is the primary purpose of this class?
2. How many different reasons could this class change?
3. Are all methods in this class related to its primary purpose?
4. Would this class make sense to someone unfamiliar with the codebase?
5. Can I easily write unit tests for this class?

## Conclusion

The Single Responsibility Principle is fundamental to writing maintainable, testable, and scalable software. While it may seem simple in concept, applying it effectively requires practice and a deep understanding of the domain you're working in.

Remember that SRP is not about having classes with only one method, but about having classes with only one reason to change. By following SRP, you create code that is easier to understand, test, maintain, and extend.

The key is finding the right balance - not every small piece of functionality needs its own class, but when you notice a class handling multiple unrelated concerns, it's time to refactor and separate those responsibilities.

Start by identifying the core responsibilities in your current codebase, then gradually refactor towards single-responsibility classes. Your future self (and your team) will thank you for the cleaner, more maintainable code.