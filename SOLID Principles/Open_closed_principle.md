# Open/Closed Principle (OCP)

## Table of Contents
1. [Introduction](#introduction)
2. [Definition and Theory](#definition-and-theory)
3. [Why OCP Matters](#why-ocp-matters)
4. [When to Apply OCP](#when-to-apply-ocp)
5. [Consequences of Violating OCP](#consequences-of-violating-ocp)
6. [Python Examples](#python-examples)
7. [Design Patterns Supporting OCP](#design-patterns-supporting-ocp)
8. [Benefits of Following OCP](#benefits-of-following-ocp)
9. [Common Pitfalls](#common-pitfalls)
10. [Best Practices](#best-practices)
11. [Conclusion](#conclusion)

## Introduction

The Open/Closed Principle (OCP) is the second principle in the SOLID principles of object-oriented design, introduced by Bertrand Meyer in 1988 and later refined by Robert C. Martin. It's a fundamental principle that helps create flexible and maintainable software architectures by promoting extensibility without modification.

## Definition and Theory

### Formal Definition
**"Software entities (classes, modules, functions, etc.) should be open for extension but closed for modification."**

### Core Concepts

#### Open for Extension
- You should be able to add new functionality to existing code
- New features can be implemented without changing existing code
- The system can grow and evolve over time

#### Closed for Modification
- Existing code should not be modified when adding new features
- Once a module is developed and tested, it should remain unchanged
- Changes to existing code can introduce bugs and break existing functionality

### The Abstraction Key
The principle relies heavily on abstraction. By depending on abstractions (interfaces, abstract classes) rather than concrete implementations, we can extend behavior through new implementations without modifying existing code.

## Why OCP Matters

### 1. **Stability**
Existing, tested code remains unchanged, reducing the risk of introducing bugs.

### 2. **Maintainability**
New features can be added without touching existing code, making maintenance easier.

### 3. **Flexibility**
The system becomes more flexible and adaptable to changing requirements.

### 4. **Reduced Risk**
Changes don't affect existing functionality, reducing the risk of breaking working features.

### 5. **Team Productivity**
Multiple developers can work on extending the system without interfering with each other.

### 6. **Version Control Benefits**
Fewer merge conflicts and cleaner commit histories when adding new features.

## When to Apply OCP

### You Should Apply OCP When:

1. **You anticipate future extensions**
2. **Requirements are likely to change frequently**
3. **Multiple variants of similar functionality exist**
4. **You're building a framework or library**
5. **Working in a team environment**
6. **The cost of change is high**

### Warning Signs That OCP is Violated:

- Frequent modifications to existing classes when adding new features
- Long chains of if-else or switch statements based on type
- Code that needs to be recompiled when new features are added
- Classes that change for multiple reasons
- Tight coupling between modules
- Fear of modifying existing code due to potential side effects

## Consequences of Violating OCP

### 1. **Ripple Effects**
Changes in one part of the system require changes in multiple other parts.

### 2. **Increased Testing Burden**
Every modification requires retesting of existing functionality.

### 3. **Higher Risk of Bugs**
Modifying existing code can introduce new bugs in previously working features.

### 4. **Reduced Productivity**
Developers spend more time understanding and modifying existing code rather than building new features.

### 5. **Tight Coupling**
Systems become tightly coupled, making them difficult to maintain and extend.

### 6. **Deployment Complexity**
Changes require redeployment of entire modules rather than just new components.

## Python Examples

### Example 1: Shape Area Calculator

#### Violating OCP

```python
# BAD: Violates OCP
class AreaCalculator:
    def calculate_area(self, shapes):
        total_area = 0
        for shape in shapes:
            if shape.type == "rectangle":
                total_area += shape.width * shape.height
            elif shape.type == "circle":
                total_area += 3.14159 * shape.radius * shape.radius
            elif shape.type == "triangle":
                total_area += 0.5 * shape.base * shape.height
            # Problem: Adding a new shape requires modifying this method
            # What if we want to add square, pentagon, hexagon, etc.?
        return total_area

class Rectangle:
    def __init__(self, width, height):
        self.type = "rectangle"
        self.width = width
        self.height = height

class Circle:
    def __init__(self, radius):
        self.type = "circle"
        self.radius = radius

class Triangle:
    def __init__(self, base, height):
        self.type = "triangle"
        self.base = base
        self.height = height

# Usage
shapes = [
    Rectangle(5, 10),
    Circle(7),
    Triangle(4, 6)
]

calculator = AreaCalculator()
total_area = calculator.calculate_area(shapes)
print(f"Total area: {total_area}")
```

#### Following OCP

```python
# GOOD: Following OCP
from abc import ABC, abstractmethod
import math

class Shape(ABC):
    """Abstract base class defining the interface for all shapes"""
    
    @abstractmethod
    def calculate_area(self):
        pass

class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height
    
    def calculate_area(self):
        return self.width * self.height

class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius
    
    def calculate_area(self):
        return math.pi * self.radius * self.radius

class Triangle(Shape):
    def __init__(self, base, height):
        self.base = base
        self.height = height
    
    def calculate_area(self):
        return 0.5 * self.base * self.height

# Easy to extend without modifying existing code
class Square(Shape):
    def __init__(self, side):
        self.side = side
    
    def calculate_area(self):
        return self.side * self.side

class Pentagon(Shape):
    def __init__(self, side):
        self.side = side
    
    def calculate_area(self):
        # Regular pentagon area formula
        return (1/4) * math.sqrt(25 + 10 * math.sqrt(5)) * self.side * self.side

class Ellipse(Shape):
    def __init__(self, major_axis, minor_axis):
        self.major_axis = major_axis
        self.minor_axis = minor_axis
    
    def calculate_area(self):
        return math.pi * self.major_axis * self.minor_axis

class AreaCalculator:
    """Calculator remains unchanged regardless of new shapes"""
    def calculate_total_area(self, shapes):
        return sum(shape.calculate_area() for shape in shapes)
    
    def calculate_average_area(self, shapes):
        if not shapes:
            return 0
        return self.calculate_total_area(shapes) / len(shapes)
    
    def get_largest_area(self, shapes):
        if not shapes:
            return 0
        return max(shape.calculate_area() for shape in shapes)

# Usage - Calculator works with any shape without modification
shapes = [
    Rectangle(5, 10),
    Circle(7),
    Triangle(4, 6),
    Square(8),
    Pentagon(5),
    Ellipse(3, 2)
]

calculator = AreaCalculator()
print(f"Total area: {calculator.calculate_total_area(shapes)}")
print(f"Average area: {calculator.calculate_average_area(shapes)}")
print(f"Largest area: {calculator.get_largest_area(shapes)}")
```

### Example 2: Payment Processing System

#### Violating OCP

```python
# BAD: Violates OCP
class PaymentProcessor:
    def process_payment(self, payment_method, amount, details):
        if payment_method == "credit_card":
            return self._process_credit_card(amount, details)
        elif payment_method == "paypal":
            return self._process_paypal(amount, details)
        elif payment_method == "bank_transfer":
            return self._process_bank_transfer(amount, details)
        elif payment_method == "crypto":
            return self._process_crypto(amount, details)
        # Problem: Adding new payment methods requires modifying this method
        else:
            raise ValueError(f"Unsupported payment method: {payment_method}")
    
    def _process_credit_card(self, amount, details):
        print(f"Processing ${amount} via Credit Card")
        # Credit card processing logic
        return {"status": "success", "transaction_id": "CC123456"}
    
    def _process_paypal(self, amount, details):
        print(f"Processing ${amount} via PayPal")
        # PayPal processing logic
        return {"status": "success", "transaction_id": "PP789012"}
    
    def _process_bank_transfer(self, amount, details):
        print(f"Processing ${amount} via Bank Transfer")
        # Bank transfer processing logic
        return {"status": "success", "transaction_id": "BT345678"}
    
    def _process_crypto(self, amount, details):
        print(f"Processing ${amount} via Cryptocurrency")
        # Crypto processing logic
        return {"status": "success", "transaction_id": "CR901234"}

# Usage
processor = PaymentProcessor()
result = processor.process_payment("credit_card", 100.0, {"card_number": "1234"})
print(result)
```

#### Following OCP

```python
# GOOD: Following OCP
from abc import ABC, abstractmethod
from typing import Dict, Any
import uuid

class PaymentMethod(ABC):
    """Abstract base class for all payment methods"""
    
    @abstractmethod
    def process_payment(self, amount: float, details: Dict[str, Any]) -> Dict[str, Any]:
        pass
    
    @abstractmethod
    def validate_details(self, details: Dict[str, Any]) -> bool:
        pass

class CreditCardPayment(PaymentMethod):
    def validate_details(self, details: Dict[str, Any]) -> bool:
        required_fields = ["card_number", "expiry_date", "cvv"]
        return all(field in details for field in required_fields)
    
    def process_payment(self, amount: float, details: Dict[str, Any]) -> Dict[str, Any]:
        if not self.validate_details(details):
            return {"status": "error", "message": "Invalid credit card details"}
        
        print(f"Processing ${amount} via Credit Card")
        # Credit card processing logic
        transaction_id = f"CC{uuid.uuid4().hex[:8]}"
        return {
            "status": "success",
            "transaction_id": transaction_id,
            "method": "credit_card",
            "amount": amount
        }

class PayPalPayment(PaymentMethod):
    def validate_details(self, details: Dict[str, Any]) -> bool:
        return "email" in details and "password" in details
    
    def process_payment(self, amount: float, details: Dict[str, Any]) -> Dict[str, Any]:
        if not self.validate_details(details):
            return {"status": "error", "message": "Invalid PayPal credentials"}
        
        print(f"Processing ${amount} via PayPal")
        # PayPal processing logic
        transaction_id = f"PP{uuid.uuid4().hex[:8]}"
        return {
            "status": "success",
            "transaction_id": transaction_id,
            "method": "paypal",
            "amount": amount
        }

class BankTransferPayment(PaymentMethod):
    def validate_details(self, details: Dict[str, Any]) -> bool:
        required_fields = ["account_number", "routing_number"]
        return all(field in details for field in required_fields)
    
    def process_payment(self, amount: float, details: Dict[str, Any]) -> Dict[str, Any]:
        if not self.validate_details(details):
            return {"status": "error", "message": "Invalid bank details"}
        
        print(f"Processing ${amount} via Bank Transfer")
        # Bank transfer processing logic
        transaction_id = f"BT{uuid.uuid4().hex[:8]}"
        return {
            "status": "success",
            "transaction_id": transaction_id,
            "method": "bank_transfer",
            "amount": amount
        }

# Easy to extend with new payment methods
class CryptoPayment(PaymentMethod):
    def validate_details(self, details: Dict[str, Any]) -> bool:
        return "wallet_address" in details and "private_key" in details
    
    def process_payment(self, amount: float, details: Dict[str, Any]) -> Dict[str, Any]:
        if not self.validate_details(details):
            return {"status": "error", "message": "Invalid crypto wallet details"}
        
        print(f"Processing ${amount} via Cryptocurrency")
        # Crypto processing logic
        transaction_id = f"CR{uuid.uuid4().hex[:8]}"
        return {
            "status": "success",
            "transaction_id": transaction_id,
            "method": "cryptocurrency",
            "amount": amount
        }

class ApplePayPayment(PaymentMethod):
    def validate_details(self, details: Dict[str, Any]) -> bool:
        return "touch_id" in details or "face_id" in details
    
    def process_payment(self, amount: float, details: Dict[str, Any]) -> Dict[str, Any]:
        if not self.validate_details(details):
            return {"status": "error", "message": "Biometric authentication required"}
        
        print(f"Processing ${amount} via Apple Pay")
        transaction_id = f"AP{uuid.uuid4().hex[:8]}"
        return {
            "status": "success",
            "transaction_id": transaction_id,
            "method": "apple_pay",
            "amount": amount
        }

class GooglePayPayment(PaymentMethod):
    def validate_details(self, details: Dict[str, Any]) -> bool:
        return "google_account" in details and "device_id" in details
    
    def process_payment(self, amount: float, details: Dict[str, Any]) -> Dict[str, Any]:
        if not self.validate_details(details):
            return {"status": "error", "message": "Invalid Google Pay setup"}
        
        print(f"Processing ${amount} via Google Pay")
        transaction_id = f"GP{uuid.uuid4().hex[:8]}"
        return {
            "status": "success",
            "transaction_id": transaction_id,
            "method": "google_pay",
            "amount": amount
        }

class PaymentProcessor:
    """Processor remains unchanged regardless of new payment methods"""
    def __init__(self):
        self.payment_methods: Dict[str, PaymentMethod] = {}
    
    def register_payment_method(self, name: str, payment_method: PaymentMethod):
        """Register a new payment method"""
        self.payment_methods[name] = payment_method
    
    def process_payment(self, method_name: str, amount: float, details: Dict[str, Any]) -> Dict[str, Any]:
        if method_name not in self.payment_methods:
            return {"status": "error", "message": f"Unsupported payment method: {method_name}"}
        
        payment_method = self.payment_methods[method_name]
        return payment_method.process_payment(amount, details)
    
    def get_available_methods(self) -> list:
        return list(self.payment_methods.keys())

# Usage
processor = PaymentProcessor()

# Register payment methods
processor.register_payment_method("credit_card", CreditCardPayment())
processor.register_payment_method("paypal", PayPalPayment())
processor.register_payment_method("bank_transfer", BankTransferPayment())
processor.register_payment_method("crypto", CryptoPayment())
processor.register_payment_method("apple_pay", ApplePayPayment())
processor.register_payment_method("google_pay", GooglePayPayment())

# Process payments
credit_card_details = {
    "card_number": "1234-5678-9012-3456",
    "expiry_date": "12/25",
    "cvv": "123"
}

result = processor.process_payment("credit_card", 100.0, credit_card_details)
print(result)

print(f"Available payment methods: {processor.get_available_methods()}")
```

### Example 3: Notification System

#### Violating OCP

```python
# BAD: Violates OCP
class NotificationService:
    def send_notification(self, notification_type, message, recipient):
        if notification_type == "email":
            self._send_email(message, recipient)
        elif notification_type == "sms":
            self._send_sms(message, recipient)
        elif notification_type == "push":
            self._send_push_notification(message, recipient)
        elif notification_type == "slack":
            self._send_slack_message(message, recipient)
        # Problem: Adding new notification types requires modifying this method
        else:
            raise ValueError(f"Unsupported notification type: {notification_type}")
    
    def _send_email(self, message, recipient):
        print(f"Sending email to {recipient}: {message}")
    
    def _send_sms(self, message, recipient):
        print(f"Sending SMS to {recipient}: {message}")
    
    def _send_push_notification(self, message, recipient):
        print(f"Sending push notification to {recipient}: {message}")
    
    def _send_slack_message(self, message, recipient):
        print(f"Sending Slack message to {recipient}: {message}")

# Usage
service = NotificationService()
service.send_notification("email", "Hello World!", "user@example.com")
```

#### Following OCP

```python
# GOOD: Following OCP
from abc import ABC, abstractmethod
from typing import Dict, Any, List
from datetime import datetime

class NotificationChannel(ABC):
    """Abstract base class for notification channels"""
    
    @abstractmethod
    def send(self, message: str, recipient: str, metadata: Dict[str, Any] = None) -> bool:
        pass
    
    @abstractmethod
    def validate_recipient(self, recipient: str) -> bool:
        pass
    
    @property
    @abstractmethod
    def channel_name(self) -> str:
        pass

class EmailNotification(NotificationChannel):
    @property
    def channel_name(self) -> str:
        return "email"
    
    def validate_recipient(self, recipient: str) -> bool:
        return "@" in recipient and "." in recipient.split("@")[1]
    
    def send(self, message: str, recipient: str, metadata: Dict[str, Any] = None) -> bool:
        if not self.validate_recipient(recipient):
            print(f"Invalid email address: {recipient}")
            return False
        
        subject = metadata.get("subject", "Notification") if metadata else "Notification"
        print(f"📧 Email sent to {recipient}")
        print(f"Subject: {subject}")
        print(f"Message: {message}")
        return True

class SMSNotification(NotificationChannel):
    @property
    def channel_name(self) -> str:
        return "sms"
    
    def validate_recipient(self, recipient: str) -> bool:
        # Simple phone number validation
        return recipient.replace("+", "").replace("-", "").replace(" ", "").isdigit()
    
    def send(self, message: str, recipient: str, metadata: Dict[str, Any] = None) -> bool:
        if not self.validate_recipient(recipient):
            print(f"Invalid phone number: {recipient}")
            return False
        
        # SMS has character limit
        if len(message) > 160:
            message = message[:157] + "..."
        
        print(f"📱 SMS sent to {recipient}: {message}")
        return True

class PushNotification(NotificationChannel):
    @property
    def channel_name(self) -> str:
        return "push"
    
    def validate_recipient(self, recipient: str) -> bool:
        # Device token validation (simplified)
        return len(recipient) > 20
    
    def send(self, message: str, recipient: str, metadata: Dict[str, Any] = None) -> bool:
        if not self.validate_recipient(recipient):
            print(f"Invalid device token: {recipient}")
            return False
        
        title = metadata.get("title", "App Notification") if metadata else "App Notification"
        print(f"🔔 Push notification sent to device {recipient[:10]}...")
        print(f"Title: {title}")
        print(f"Message: {message}")
        return True

class SlackNotification(NotificationChannel):
    @property
    def channel_name(self) -> str:
        return "slack"
    
    def validate_recipient(self, recipient: str) -> bool:
        return recipient.startswith("#") or recipient.startswith("@")
    
    def send(self, message: str, recipient: str, metadata: Dict[str, Any] = None) -> bool:
        if not self.validate_recipient(recipient):
            print(f"Invalid Slack recipient: {recipient}")
            return False
        
        print(f"💬 Slack message sent to {recipient}: {message}")
        return True

# Easy to extend with new notification types
class DiscordNotification(NotificationChannel):
    @property
    def channel_name(self) -> str:
        return "discord"
    
    def validate_recipient(self, recipient: str) -> bool:
        return "#" in recipient or recipient.isdigit()
    
    def send(self, message: str, recipient: str, metadata: Dict[str, Any] = None) -> bool:
        if not self.validate_recipient(recipient):
            print(f"Invalid Discord recipient: {recipient}")
            return False
        
        print(f"🎮 Discord message sent to {recipient}: {message}")
        return True

class TelegramNotification(NotificationChannel):
    @property
    def channel_name(self) -> str:
        return "telegram"
    
    def validate_recipient(self, recipient: str) -> bool:
        return recipient.startswith("@") or recipient.isdigit()
    
    def send(self, message: str, recipient: str, metadata: Dict[str, Any] = None) -> bool:
        if not self.validate_recipient(recipient):
            print(f"Invalid Telegram recipient: {recipient}")
            return False
        
        print(f"✈️ Telegram message sent to {recipient}: {message}")
        return True

class WhatsAppNotification(NotificationChannel):
    @property
    def channel_name(self) -> str:
        return "whatsapp"
    
    def validate_recipient(self, recipient: str) -> bool:
        return recipient.replace("+", "").replace("-", "").replace(" ", "").isdigit()
    
    def send(self, message: str, recipient: str, metadata: Dict[str, Any] = None) -> bool:
        if not self.validate_recipient(recipient):
            print(f"Invalid WhatsApp number: {recipient}")
            return False
        
        print(f"💚 WhatsApp message sent to {recipient}: {message}")
        return True

class NotificationService:
    """Service remains unchanged regardless of new notification channels"""
    def __init__(self):
        self.channels: Dict[str, NotificationChannel] = {}
        self.notification_log: List[Dict[str, Any]] = []
    
    def register_channel(self, channel: NotificationChannel):
        """Register a new notification channel"""
        self.channels[channel.channel_name] = channel
    
    def send_notification(self, channel_name: str, message: str, recipient: str, metadata: Dict[str, Any] = None) -> bool:
        if channel_name not in self.channels:
            print(f"Unsupported notification channel: {channel_name}")
            return False
        
        channel = self.channels[channel_name]
        success = channel.send(message, recipient, metadata)
        
        # Log the notification
        self.notification_log.append({
            "timestamp": datetime.now(),
            "channel": channel_name,
            "recipient": recipient,
            "message": message,
            "success": success
        })
        
        return success
    
    def send_bulk_notification(self, channel_name: str, message: str, recipients: List[str], metadata: Dict[str, Any] = None) -> Dict[str, int]:
        results = {"success": 0, "failed": 0}
        
        for recipient in recipients:
            if self.send_notification(channel_name, message, recipient, metadata):
                results["success"] += 1
            else:
                results["failed"] += 1
        
        return results
    
    def send_multi_channel_notification(self, channels: List[str], message: str, recipients: Dict[str, str]) -> Dict[str, bool]:
        """Send notification across multiple channels"""
        results = {}
        
        for channel_name in channels:
            if channel_name in recipients:
                recipient = recipients[channel_name]
                results[channel_name] = self.send_notification(channel_name, message, recipient)
        
        return results
    
    def get_available_channels(self) -> List[str]:
        return list(self.channels.keys())
    
    def get_notification_history(self, limit: int = 10) -> List[Dict[str, Any]]:
        return self.notification_log[-limit:]

# Usage
service = NotificationService()

# Register all notification channels
service.register_channel(EmailNotification())
service.register_channel(SMSNotification())
service.register_channel(PushNotification())
service.register_channel(SlackNotification())
service.register_channel(DiscordNotification())
service.register_channel(TelegramNotification())
service.register_channel(WhatsAppNotification())

# Send single notifications
service.send_notification("email", "Welcome to our service!", "user@example.com", {"subject": "Welcome!"})
service.send_notification("sms", "Your verification code is 123456", "+1234567890")
service.send_notification("slack", "Deployment completed successfully", "#general")

# Send bulk notifications
recipients = ["user1@example.com", "user2@example.com", "invalid-email"]
results = service.send_bulk_notification("email", "Newsletter", recipients)
print(f"Bulk email results: {results}")

# Send multi-channel notification
multi_recipients = {
    "email": "user@example.com",
    "sms": "+1234567890",
    "slack": "#alerts"
}
multi_results = service.send_multi_channel_notification(
    ["email", "sms", "slack"], 
    "Critical system alert!", 
    multi_recipients
)
print(f"Multi-channel results: {multi_results}")

print(f"Available channels: {service.get_available_channels()}")
print(f"Recent notifications: {len(service.get_notification_history())}")
```

### Example 4: Discount Calculation System

#### Violating OCP

```python
# BAD: Violates OCP
class DiscountCalculator:
    def calculate_discount(self, customer_type, order_amount, discount_code=None):
        discount = 0
        
        # Customer type discounts
        if customer_type == "regular":
            discount = 0
        elif customer_type == "premium":
            discount = order_amount * 0.10  # 10% discount
        elif customer_type == "vip":
            discount = order_amount * 0.20  # 20% discount
        elif customer_type == "employee":
            discount = order_amount * 0.30  # 30% discount
        
        # Discount code handling
        if discount_code:
            if discount_code == "SAVE10":
                discount += order_amount * 0.10
            elif discount_code == "SAVE20":
                discount += order_amount * 0.20
            elif discount_code == "FIRSTTIME":
                discount += order_amount * 0.15
            elif discount_code == "LOYALTY":
                discount += order_amount * 0.25
        
        # Seasonal discounts (hardcoded dates)
        from datetime import datetime
        today = datetime.now()
        if today.month == 12:  # December
            discount += order_amount * 0.05  # Holiday discount
        elif today.month == 7:  # July
            discount += order_amount * 0.03  # Summer discount
        
        return min(discount, order_amount * 0.50)  # Max 50% discount

# Usage
calculator = DiscountCalculator()
discount = calculator.calculate_discount("premium", 100, "SAVE10")
print(f"Discount: ${discount}")
```

#### Following OCP

```python
# GOOD: Following OCP
from abc import ABC, abstractmethod
from typing import Dict, Any, List
from datetime import datetime
from enum import Enum

class DiscountType(Enum):
    PERCENTAGE = "percentage"
    FIXED_AMOUNT = "fixed_amount"

class Discount(ABC):
    """Abstract base class for all discount types"""
    
    def __init__(self, name: str, description: str):
        self.name = name
        self.description = description
    
    @abstractmethod
    def calculate_discount(self, order_amount: float, customer_data: Dict[str, Any]) -> float:
        pass
    
    @abstractmethod
    def is_applicable(self, customer_data: Dict[str, Any]) -> bool:
        pass
    
    @property
    @abstractmethod
    def discount_type(self) -> DiscountType:
        pass

class CustomerTierDiscount(Discount):
    def __init__(self, tier: str, percentage: float):
        super().__init__(f"{tier.title()} Customer Discount", f"{percentage*100}% discount for {tier} customers")
        self.tier = tier
        self.percentage = percentage
    
    @property
    def discount_type(self) -> DiscountType:
        return DiscountType.PERCENTAGE
    
    def is_applicable(self, customer_data: Dict[str, Any]) -> bool:
        return customer_data.get("tier") == self.tier
    
    def calculate_discount(self, order_amount: float, customer_data: Dict[str, Any]) -> float:
        if self.is_applicable(customer_data):
            return order_amount * self.percentage
        return 0

class PromoCodeDiscount(Discount):
    def __init__(self, code: str, percentage: float, description: str = None):
        desc = description or f"{percentage*100}% discount with code {code}"
        super().__init__(f"Promo Code: {code}", desc)
        self.code = code
        self.percentage = percentage
    
    @property
    def discount_type(self) -> DiscountType:
        return DiscountType.PERCENTAGE
    
    def is_applicable(self, customer_data: Dict[str, Any]) -> bool:
        return customer_data.get("promo_code") == self.code
    
    def calculate_discount(self, order_amount: float, customer_data: Dict[str, Any]) -> float:
        if self.is_applicable(customer_data):
            return order_amount * self.percentage
        return 0

class SeasonalDiscount(Discount):
    def __init__(self, name: str, months: List[int], percentage: float):
        super().__init__(name, f"{percentage*100}% seasonal discount")
        self.months = months
        self.percentage = percentage
    
    @property
    def discount_type(self) -> DiscountType:
        return DiscountType.PERCENTAGE
    
    def is_applicable(self, customer_data: Dict[str, Any]) -> bool:
        current_month = datetime.now().month
        return current_month in self.months
    
    def calculate_discount(self, order_amount: float, customer_data: Dict[str, Any]) -> float:
        if self.is_applicable(customer_data):
            return order_amount * self.percentage
        return 0

class MinimumOrderDiscount(Discount):
    def __init__(self, name: str, minimum_amount: float, discount_amount: float):
        super().__init__(name, f"${discount_amount} off orders over ${minimum_amount}")
        self.minimum_amount = minimum_amount
        self.discount_amount = discount_amount
    
    @property
    def discount_type(self) -> DiscountType:
        return DiscountType.FIXED_AMOUNT
    
    def is_applicable(self, customer_data: Dict[str, Any]) -> bool:
        return True  # Always check order amount
    
    def calculate_discount(self, order_amount: float, customer_data: Dict[str, Any]) -> float:
        if order_amount >= self.minimum_amount:
            return self.discount_amount
        return 0

# Easy to extend with new discount types
class LoyaltyPointsDiscount(Discount):
    def __init__(self, points_required: int, discount_percentage: float):
        super().__init__("Loyalty Points Discount", f"{discount_percentage*100}% off with {points_required} points")
        self.points_required = points_required
        self.discount_percentage = discount_percentage
    
    @property
    def discount_type(self) -> DiscountType:
        return DiscountType.PERCENTAGE
    
    def is_applicable(self, customer_data: Dict[str, Any]) -> bool:
        return customer_data.get("loyalty_points", 0) >= self.points_required
    
    def calculate_discount(self, order_amount: float, customer_data: Dict[str, Any]) -> float:
        if self.is_applicable(customer_data):
            return order_amount * self.discount_percentage
        return 0

class BirthdayDiscount(Discount):
    def __init__(self, percentage: float):
        super().__init__("Birthday Discount", f"{percentage*100}% birthday discount")
        self.percentage = percentage
    
    @property
    def discount_type(self) -> DiscountType:
        return DiscountType.PERCENTAGE
    
    def is_applicable(self, customer_data: Dict[str, Any]) -> bool:
        birthday = customer_data.get("birthday")
        if not birthday:
            return False
        
        today = datetime.now()
        return (birthday.month == today.month and birthday.day == today.day)
    
    def calculate_discount(self, order_amount: float, customer_data: Dict[str, Any]) -> float:
        if self.is_applicable(customer_data):
            return order_amount * self.percentage
        return 0

class VolumeDiscount(Discount):
    def __init__(self, name: str, item_thresholds: Dict[int, float]):
        super().__init__(name, "Volume-based discount")
        self.item_thresholds = item_thresholds
    
    @property
    def discount_type(self) -> DiscountType:
        return DiscountType.PERCENTAGE
    
    def is_applicable(self, customer_data: Dict[str, Any]) -> bool:
        return "item_count" in customer_data
    
    def calculate_discount(self, order_amount: float, customer_data: Dict[str, Any]) -> float:
        item_count = customer_data.get("item_count", 0)
        
        applicable_percentage = 0
        for threshold, percentage in sorted(self.item_thresholds.items()):
            if item_count >= threshold:
                applicable_percentage = percentage
        
        return order_amount * applicable_percentage

class DiscountEngine:
    """Engine that applies multiple discounts without modification"""
    def __init__(self, max_discount_percentage: float = 0.5):
        self.discounts: List[Discount] = []
        self.max_discount_percentage = max_discount_percentage
    
    def register_discount(self, discount: Discount):
        """Register a new discount rule"""
        self.discounts.append(discount)
    
    def calculate_total_discount(self, order_amount: float, customer_data: Dict[str, Any]) -> Dict[str, Any]:
        applicable_discounts = []
        total_discount = 0
        
        for discount in self.discounts:
            if discount.is_applicable(customer_data):
                discount_amount = discount.calculate_discount(order_amount, customer_data)
                if discount_amount > 0:
                    applicable_discounts.append({
                        "name": discount.name,
                        "description": discount.description,
                        "amount": discount_amount,
                        "type": discount.discount_type.value
                    })
                    total_discount += discount_amount
        
        # Apply maximum discount limit
        max_discount = order_amount * self.max_discount_percentage
        final_discount = min(total_discount, max_discount)
        
        return {
            "original_amount": order_amount,
            "total_discount": final_discount,
            "final_amount": order_amount - final_discount,
            "applied_discounts": applicable_discounts,
            "discount_capped": total_discount > max_discount
        }
    
    def get_available_discounts(self, customer_data: Dict[str, Any]) -> List[Dict[str, str]]:
        """Get all applicable discounts for a customer"""
        available = []
        for discount in self.discounts:
            if discount.is_applicable(customer_data):
                available.append({
                    "name": discount.name,
                    "description": discount.description
                })
        return available

# Usage
engine = DiscountEngine(max_discount_percentage=0.6)  # 60% max discount

# Register various discount types
engine.register_discount(CustomerTierDiscount("regular", 0.0))
engine.register_discount(CustomerTierDiscount("premium", 0.10))
engine.register_discount(CustomerTierDiscount("vip", 0.20))
engine.register_discount(CustomerTierDiscount("employee", 0.30))

engine.register_discount(PromoCodeDiscount("SAVE10", 0.10))
engine.register_discount(PromoCodeDiscount("SAVE20", 0.20))
engine.register_discount(PromoCodeDiscount("FIRSTTIME", 0.15, "First-time customer discount"))
engine.register_discount(PromoCodeDiscount("LOYALTY", 0.25, "Loyalty customer special"))

engine.register_discount(SeasonalDiscount("Holiday Discount", [12], 0.05))
engine.register_discount(SeasonalDiscount("Summer Sale", [6, 7, 8], 0.03))

engine.register_discount(MinimumOrderDiscount("Free Shipping", 50, 10))
engine.register_discount(MinimumOrderDiscount("Big Order Bonus", 200, 25))

engine.register_discount(LoyaltyPointsDiscount(1000, 0.15))
engine.register_discount(BirthdayDiscount(0.20))

volume_thresholds = {5: 0.05, 10: 0.10, 20: 0.15}
engine.register_discount(VolumeDiscount("Volume Discount", volume_thresholds))

# Test discount calculation
customer_data = {
    "tier": "premium",
    "promo_code": "SAVE10",
    "loyalty_points": 1200,
    "item_count": 8,
    "birthday": datetime(1990, 12, 25)  # Assuming today is their birthday in December
}

result = engine.calculate_total_discount(150.0, customer_data)

print("Discount Calculation Result:")
print(f"Original Amount: ${result['original_amount']}")
print(f"Total Discount: ${result['total_discount']:.2f}")
print(f"Final Amount: ${result['final_amount']:.2f}")
print(f"Discount Capped: {result['discount_capped']}")
print("\nApplied Discounts:")
for discount in result['applied_discounts']:
    print(f"- {discount['name']}: ${discount['amount']:.2f} ({discount['description']})")
```

## Design Patterns Supporting OCP

### 1. Strategy Pattern
Encapsulates algorithms and makes them interchangeable without modifying client code.

### 2. Template Method Pattern
Defines the skeleton of an algorithm, allowing subclasses to override specific steps.

### 3. Observer Pattern
Allows objects to be notified of changes without tight coupling.

### 4. Factory Pattern
Creates objects without specifying their exact classes.

### 5. Decorator Pattern
Adds new functionality to objects without altering their structure.

### 6. Command Pattern
Encapsulates requests as objects, allowing for parameterization and queuing.

## Benefits of Following OCP

### 1. **Reduced Risk**
New features don't risk breaking existing functionality.

### 2. **Easier Testing**
New functionality can be tested independently.

### 3. **Better Team Collaboration**
Multiple developers can work on extensions simultaneously.

### 4. **Simplified Deployment**
New features can be deployed as separate modules.

### 5. **Improved Maintainability**
Code becomes more modular and easier to understand.

### 6. **Enhanced Flexibility**
System becomes more adaptable to changing requirements.

## Common Pitfalls

### 1. **Over-Abstraction**
Creating abstractions for every possible extension point, leading to unnecessary complexity.

### 2. **Premature Abstraction**
Abstracting too early before understanding the actual extension patterns.

### 3. **Wrong Abstractions**
Creating abstractions that don't align with actual business needs.

### 4. **Performance Overhead**
Excessive use of polymorphism and indirection can impact performance.

### 5. **Complexity Creep**
Too many abstraction layers can make code difficult to follow.

### 6. **Analysis Paralysis**
Spending too much time designing for hypothetical future extensions.

## Best Practices

### 1. **Identify Variation Points**
Look for areas where requirements are likely to change or extend.

### 2. **Use Interfaces and Abstract Classes**
Define clear contracts that concrete implementations must follow.

### 3. **Favor Composition Over Inheritance**
Use composition to build complex behavior from simple components.

### 4. **Apply YAGNI (You Ain't Gonna Need It)**
Don't over-engineer for extensions that may never be needed.

### 5. **Start Simple, Refactor When Needed**
Begin with simple implementations and abstract when patterns emerge.

### 6. **Use Dependency Injection**
Inject dependencies to reduce coupling and improve testability.

### 7. **Document Extension Points**
Clearly document how the system can be extended.

### 8. **Regular Refactoring**
Continuously refactor to maintain clean abstractions.

## Measuring OCP Compliance

### Code Metrics to Watch:
- **Cyclomatic Complexity**: High complexity in conditional statements often indicates OCP violations
- **Number of Modifications**: Frequent changes to existing classes suggest OCP issues
- **Code Coverage**: New features should be testable without modifying existing tests
- **Coupling Metrics**: High coupling between modules indicates potential OCP violations

### Questions to Ask:
1. Can I add new functionality without modifying existing code?
2. Are my abstractions stable and unlikely to change?
3. Do I have clear extension points in my design?
4. Can new features be added through configuration or plugin mechanisms?
5. Are my conditional statements based on types or behaviors?

## Conclusion

The Open/Closed Principle is essential for building maintainable and extensible software systems. By designing code that is open for extension but closed for modification, you create systems that can evolve and grow without the risk of breaking existing functionality.

The key to successfully applying OCP is to identify the right abstractions and create stable interfaces that can accommodate future extensions. This requires a good understanding of the domain and careful consideration of likely variation points.

Remember that OCP is not about creating abstractions for everything - it's about creating the right abstractions for the areas where extension is likely or valuable. Start with simple implementations and refactor towards better abstractions as patterns emerge and requirements become clearer.

When applied correctly, OCP leads to code that is more flexible, maintainable, and robust, allowing your software to adapt to changing requirements with minimal risk and effort.