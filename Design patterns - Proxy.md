#sd

The Proxy design pattern is a structural pattern that provides a substitute or placeholder for another object, controlling access to it. Think of it as a gatekeeper or an intermediary for the real object.


**Principle:** The client interacts with the Proxy just as it would with the RealSubject (the object being protected or controlled). The Proxy handles the request first and decides whether to forward it to the RealSubject.
**Goal:** To control access, reduce complexity, or add a layer of functionality (like security, logging, or caching) before or after the request hits the core object.

```python
from abc import ABC, abstractmethod

# 1. Subject Interface
class BankAccount(ABC):
    @abstractmethod
    def deposit(self, amount):
        pass
    
    @abstractmethod
    def withdraw(self, amount):
        pass

# 2. RealSubject
class RealBankAccount(BankAccount):
    def __init__(self, balance=0):
        self._balance = balance
        
    def deposit(self, amount):
        self._balance += amount
        print(f"Deposited ${amount}. New balance: ${self._balance}")
        
    def withdraw(self, amount):
        if self._balance >= amount:
            self._balance -= amount
            print(f"Withdrew ${amount}. New balance: ${self._balance}")
        else:
            print("Withdrawal denied: Insufficient funds.")

# 3. Proxy
class AccountProtectionProxy(BankAccount):
    def __init__(self, account: RealBankAccount, user_role: str):
        self._real_account = account
        self._user_role = user_role

    def deposit(self, amount):
        # The Proxy adds a security check (Protection)
        if self._user_role == "Manager":
            self._real_account.deposit(amount)
        else:
            print(f"ACCESS DENIED: {self._user_role} cannot deposit funds.")

    def withdraw(self, amount):
        # The Proxy adds a security check (Protection)
        if self._user_role == "Manager":
            self._real_account.withdraw(amount)
        else:
            print(f"ACCESS DENIED: {self._user_role} cannot withdraw funds.")

# --- Usage ---
real_account = RealBankAccount(500)

# Client 1: A regular employee (user)
user_proxy = AccountProtectionProxy(real_account, "User")
print("User attempts deposit:")
user_proxy.deposit(100) # Denied

# Client 2: A manager
manager_proxy = AccountProtectionProxy(real_account, "Manager")
print("\nManager attempts withdrawal:")
manager_proxy.withdraw(100) # Allowed
```

## Example on network

### Forward proxy

A **Forward Proxy** sits **in front of the client (user)** and handles requests going out to the internet. It acts on behalf of the client.

**Analogy to the Pattern:** The Proxy object represents the Client on the network.

**Access Control/Firewall:** Restricts which websites a Client (employee) can access. (This is a **Protection Proxy** use case.)

**Anonymity:** Hides the Client's (user's) true IP address from the RealSubject (the web server).

**Caching:** Speeds up requests by caching external website data. (This is a **Caching Proxy** use case.)
### Reverse proxy

A **Reverse Proxy** sits **in front of the server** (the RealSubject) and handles requests coming in from the internet. It acts on behalf of the server.

**Analogy to the Pattern:** The Proxy object is the _only thing_ the Client can see; it protects the RealSubject (the origin server).


**Security (Protection):** Hides the identity and structure of the internal web servers.

**Load Balancing:** Distributes incoming traffic across multiple internal RealSubject servers to prevent overload.

**SSL Termination:** Decrypts incoming encrypted traffic before passing it to the internal servers.