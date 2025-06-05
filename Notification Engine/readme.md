# 📓 System Design Notes – Day 14

## 🌟 Topic: Notification System using OOP + Design Patterns

---

## 🔧 Patterns Used

| Pattern   | Where/Why                                                                    |
| --------- | ---------------------------------------------------------------------------- |
| Singleton | `NotificationService`: Global single source to manage sending notifications  |
| Observer  | `NotificationObservable` ➝ notifies `Logger`, `NotificationEngine` observers |
| Decorator | Decorate `INotification` with timestamp, signature, etc.                     |
| Strategy  | Different ways to deliver notifications: Email, SMS, Popup                   |

---

## 📊 UML Breakdown

### 🔷 Core Notification Components

```plaintext
+--------------------+         <<abstract>>
|   INotification    |------------------------------+
| + getContent()     |                              |
+--------------------+                              |
          ^                                        has-a
          |                                         |
+--------------------------+        +-----------------------------+
| SimpleNotification       |        | INotificationDecorator      |
| - text: string           |        | - INotification* notifi     |
| + getContent()           |        | + getContent()              |
+--------------------------+        +-----------------------------+
                                          ^           ^
                                +----------------+  +------------------+
                                | TimestampDecorator | SignatureDecorator|
                                +----------------+  +------------------+
```

---

### 🟡 Observable - Observer Structure

```plaintext
+-------------------+        1     *        +-----------------+
| IObservable       |---------------------->| IObserver       |
| + addObserver()   |                      | + update()      |
| + removeObserver()|                      +-----------------+
| + notify()        |
+-------------------+
        ^                                    ^
        |                                    |
+------------------------+        +------------------------+
| NotificationObservable |        | Logger                |
| - vector<IObserver*>   |        | NotificationEngine    |
| - INotification* notifi|        | - vector<Strategy*>   |
| + notifyObservers()    |        | + update()            |
+------------------------+        +------------------------+
```

---

### 🧩 Strategy Interfaces (Delivery Mechanism)

```plaintext
          <<abstract>>
+----------------------------+
|  INotificationStrategy     |
| + sendNotification(string) |
+----------------------------+
       ^        ^        ^
       |        |        |
+-----------+ +---------+ +-------------+
| Email     | |  SMS    | |  Popup      |
+-----------+ +---------+ +-------------+
```

---

## 🧱 Code Explanation

### 🔶 1. Notification Interface & Decorators

```cpp
class INotification { virtual string getContent() = 0; };
class SimpleNotification : public INotification { ... };
class INotificationDecorator : public INotification {
  INotification* notification;
};
class TimestampDecorator : public INotificationDecorator { ... };
class SignatureDecorator : public INotificationDecorator { ... };
```

➡️ Decorators add timestamp and signature dynamically without changing the base `SimpleNotification`.

---

### 🔶 2. Observable & Observer (Observer Pattern)

```cpp
class IObserver { virtual void update() = 0; };
class IObservable {
  void addObserver();
  void removeObserver();
  void notifyObservers();
};
```

#### 🔁 `NotificationObservable`

* Holds list of `IObserver`.
* When `setNotification()` is called → triggers `notifyObservers()`.

#### 📝 `Logger` & `NotificationEngine`

* Observers who register for updates
* `Logger` just prints
* `NotificationEngine` delivers via strategies

---

### 🔶 3. Strategy Pattern for Delivery

```cpp
class INotificationStrategy {
  virtual void sendNotification(string content) = 0;
};
```

#### Implementations:

* `EmailStrategy`
* `SMSStrategy`
* `PopupStrategy`

Used inside `NotificationEngine` to loop through all and deliver!

---

### 🧠 Flow Summary (main)

```cpp
1. Create NotificationService (Singleton)
2. Register Observers → Logger + NotificationEngine
3. Add Strategies to Engine: Email, SMS, Popup
4. Create a Notification → SimpleNotification
   → Wrap with TimestampDecorator
   → Wrap with SignatureDecorator
5. Call NotificationService.sendNotification(notification)
6. NotificationObservable calls notifyObservers()
7. Each observer calls `update()`
   Logger → prints
   Engine → sends via all strategies
```

---

## 🧾 Sample Output

```bash
Logging New Notification:
[2025-04-13 14:22:00] Your order has been shipped!
-- Customer Care

Sending email Notification to: random.person@gmail.com
[2025-04-13 14:22:00] Your order has been shipped!
-- Customer Care

Sending SMS Notification to: +91 9876543210
...
```

---

## ✅ Summary Table

| Concept           | Role                                        |
| ----------------- | ------------------------------------------- |
| Decorator Pattern | Wraps notification with timestamp/signature |
| Observer Pattern  | Updates Logger and Engine on notification   |
| Strategy Pattern  | Deliver via different modes                 |
| Singleton         | Manages system via `NotificationService`    |

---
