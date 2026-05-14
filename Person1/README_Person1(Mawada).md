# Person 1 - Factory Patterns & Chain of Responsibility

## 📌 Overview

This module provides:
1. **Factory Method & Abstract Factory** for drink creation
2. **Chain of Responsibility** for order processing and role-based authorization

All code is in package `creational` and `chain`.

---

## 🧠 Concept 1: Factory Method Pattern

### What is it?
A design pattern that defines an interface for creating an object, but lets subclasses decide which class to instantiate.

### In Simple Words
You have one **abstract creator** (`DrinkFactory`) and multiple **concrete creators** (`CoffeeFactory`, `TeaFactory`, `JuiceFactory`). Each concrete creator creates one specific product.

### In This Project

| Component | Name |
|-----------|------|
| Product Interface | `Drink` |
| Concrete Products | `Coffee`, `Tea`, `Juice` |
| Creator | `DrinkFactory` (abstract) |
| Concrete Creators | `CoffeeFactory`, `TeaFactory`, `JuiceFactory` |

### Visual
```
        DrinkFactory (abstract)
             ↑
    ┌────────┼────────┐
    │        │        │
CoffeeFactory TeaFactory JuiceFactory
    │           │           │
    ↓           ↓           ↓
  Coffee       Tea         Juice
```

### Code Example
```java
DrinkFactory factory = new CoffeeFactory();
Drink coffee = factory.createDrink();  // Always creates Coffee
coffee.prepare();
```

### When to Use
- When a class doesn't know what subclasses will be needed
- When subclasses specify which objects to create
- You have one product type but multiple ways to create it

---

## 🧠 Concept 2: Abstract Factory Pattern

### What is it?
A design pattern that provides an interface for creating **families of related products** without specifying their concrete classes.

### In Simple Words
Instead of one factory for one product, you have one factory for **multiple related products**. Here, `HotDrinkFactory` creates both `Coffee` and `Tea` (hot drinks family), while `ColdDrinkFactory` creates `Juice` (cold drinks family).

### In This Project

| Component | Name |
|-----------|------|
| Abstract Factory Interface | `DrinkFamilyFactory` |
| Concrete Factories | `HotDrinkFactory`, `ColdDrinkFactory` |
| Products | `Coffee`, `Tea`, `Juice` |
| Factory Resolver | `DrinkFactoryResolver` |

### Visual
```
        DrinkFamilyFactory (interface)
                  ↑
        ┌─────────┴─────────┐
        │                   │
  HotDrinkFactory     ColdDrinkFactory
        │                   │
    creates:             creates:
    - Coffee             - Juice
    - Tea
```

### Code Example
```java
DrinkFamilyFactory factory = DrinkFactoryResolver.getFactory("HOT");
Drink coffee = factory.createCoffee();  // Works
Drink tea = factory.createTea();        // Works
Drink juice = factory.createJuice();    // Returns null (not in hot family)
```

### When to Use
- When you have families of related products
- When you want to enforce that products from the same family are used together
- When system should be independent of how products are created

---

## 🧠 Concept 3: Chain of Responsibility - Order Processing (Optional)

### What is it?
A behavioral pattern that passes a request along a chain of handlers. Each handler decides either to process the request or pass it to the next handler.

### In Simple Words
Your order goes through multiple **checkpoints** (handlers). Each checkpoint checks one thing:
1. Is the drink on the menu?
2. Is enough stock available?
3. Is payment sufficient?
4. Prepare the drink
5. Send notification

If any checkpoint fails, the order stops there with an error message.

### In This Project

| Handler | Responsibility |
|---------|---------------|
| `MenuValidatorHandler` | Checks if drink exists on menu |
| `StockValidatorHandler` | Checks if enough stock available |
| `PaymentValidatorHandler` | Checks if payment is sufficient |
| `OrderProcessorHandler` | Prepares the drink |
| `NotificationHandler` | Sends confirmation |

### Visual
```
Order Request
     ↓
MenuValidator → ❌ "Drink not found"
     ↓ ✅
StockValidator → ❌ "Not enough stock"
     ↓ ✅
PaymentValidator → ❌ "Insufficient payment"
     ↓ ✅
OrderProcessor → 🛠️ Preparing drink
     ↓
NotificationHandler → 📧 Sending confirmation
     ↓
✅ Order Complete
```

### Code Example
```java
OrderHandler chain = OrderProcessingChain.buildChain();

OrderRequest request = new OrderRequest("Coffee", 2, 50.0);
chain.handle(request);

if (request.isProcessed()) {
    System.out.println("Order ready!");
} else {
    System.out.println(request.getErrorMessage());
}
```

### Menu & Prices
| Drink | Price (EGP) | Stock |
|-------|-------------|-------|
| Coffee | 15 | 10 |
| Tea | 10 | 5 |
| Juice | 20 | 3 |

---

## 🧠 Concept 4: Chain of Responsibility - Roles (Authorization)

### What is it?
A behavioral pattern that passes an authorization request along a chain of role handlers. Each role checks if the user has permission to perform the action.

### In Simple Words
Different people have different permissions:
- **Customer**: Can only order basic drinks
- **Staff**: Can order more drinks + give small discounts (up to 10%)
- **Manager**: Can give bigger discounts (up to 30%) + add temporary drinks
- **Admin**: Can do anything

If a user doesn't have permission, the request passes to the next higher role.

### In This Project

| Role | Allowed Actions |
|------|----------------|
| `Customer` | Order basic drinks (`Coffee`, `Tea`, `Juice`) only |
| `Staff` | Order more drinks + discount up to 10% |
| `Manager` | Any drink + discount up to 30% + add temporary drinks |
| `Admin` | Full access (any discount, add permanent drinks, edit system) |

### Available Actions
- `ORDER_DRINK` - Order a drink
- `APPLY_DISCOUNT` - Apply a discount percentage
- `ADD_TEMP_DRINK` - Add a temporary drink to menu (Manager+)
- `ADD_PERMANENT_DRINK` - Add a permanent drink to menu (Admin only)
- `EDIT_SYSTEM` - Edit system settings (Admin only)

### Visual
```
User Request (Role: Staff, Action: 20% discount)
     ↓
CustomerHandler → "Not my job, passing..."
     ↓
StaffHandler → "20% exceeds my limit (max 10%), passing..."
     ↓
ManagerHandler → "20% is within my limit (max 30%), approved ✅"
```

### Code Example
```java
RoleHandler chain = RoleChainBuilder.buildChain();

RoleRequest request = new RoleRequest("ORDER_DRINK", "Customer");
request.setDrinkName("Coffee");
chain.handle(request);

if (request.isApproved()) {
    // Proceed with order
} else {
    System.out.println(request.getMessage());
}
```

---

## 🔌 How to Integrate

### For Person 5 (Facade) - Full Integration Example
```java
public class CafeFacade {
    private RoleHandler roleChain = RoleChainBuilder.buildChain();
    private OrderHandler orderChain = OrderProcessingChain.buildChain();
    
    public String placeOrder(String role, String drinkType, int quantity, double payment) {
        // Step 1: Check role permission
        RoleRequest roleRequest = new RoleRequest("ORDER_DRINK", role);
        roleRequest.setDrinkName(drinkType);
        roleChain.handle(roleRequest);
        
        if (!roleRequest.isApproved()) {
            return roleRequest.getMessage();
        }
        
        // Step 2: Create drink using factory
        DrinkFamilyFactory factory = DrinkFactoryResolver.getFactory("HOT");
        Drink drink = null;
        if (drinkType.equals("Coffee")) drink = factory.createCoffee();
        else if (drinkType.equals("Tea")) drink = factory.createTea();
        else if (drinkType.equals("Juice")) drink = factory.createJuice();
        
        if (drink == null) {
            return "Drink not available";
        }
        
        // Step 3: Validate order
        OrderRequest orderRequest = new OrderRequest(drinkType, quantity, payment);
        orderChain.handle(orderRequest);
        
        if (orderRequest.isProcessed()) {
            drink.prepare();
            return "Order ready! Total: " + drink.getCost();
        } else {
            return orderRequest.getErrorMessage();
        }
    }
}
```

### For Person 2 (Builder)
```java
Drink baseDrink = new CoffeeFactory().createDrink();
// Then customize using your Builder pattern
```

### For Person 3 (OrderManager)
```java
OrderHandler chain = OrderProcessingChain.buildChain();
OrderRequest request = new OrderRequest("Tea", 1, 10.0);
chain.handle(request);
if (request.isProcessed()) {
    // Add to order
}
```

### For Person 4 (Decorator)
```java
Drink coffee = new CoffeeFactory().createDrink();
// Then wrap with decorators
```

---

## 📦 Package Structure

```
src/
├── creational/
│   ├── Drink.java
│   ├── Coffee.java
│   ├── Tea.java
│   ├── Juice.java
│   ├── DrinkFactory.java
│   ├── CoffeeFactory.java
│   ├── TeaFactory.java
│   ├── JuiceFactory.java
│   ├── DrinkFamilyFactory.java
│   ├── HotDrinkFactory.java
│   ├── ColdDrinkFactory.java
│   └── DrinkFactoryResolver.java
│
└── chain/
    ├── order/
    │   └── OrderProcessingChain.java
    └── roles/
        └── RoleChain.java
```

---

## ✅ Summary Table

| Pattern | Purpose | Key Classes |
|---------|---------|-------------|
| **Factory Method** | Create one specific product | `DrinkFactory`, `CoffeeFactory`, `TeaFactory`, `JuiceFactory` |
| **Abstract Factory** | Create families of products | `DrinkFamilyFactory`, `HotDrinkFactory`, `ColdDrinkFactory`, `DrinkFactoryResolver` |
| **Order Chain** | Validate order step by step | `MenuValidatorHandler`, `StockValidatorHandler`, `PaymentValidatorHandler`, `OrderProcessorHandler`, `NotificationHandler` |
| **Roles Chain** | Authorize user actions | `CustomerHandler`, `StaffHandler`, `ManagerHandler`, `AdminHandler` |

---

## 📞 Questions

Contact me if anything is unclear. Good luck team!
```