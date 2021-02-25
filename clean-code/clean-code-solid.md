# Clean Code: SOLID

## SOLID
- Single Responsibility Principle
- Open-Closed Principle
- Liskov Substitution Principle
- Interface Segregation Principle
- Dependency Inversion Principle

### S — Single-Responsibility Principle (SRP)
> Classes should have a single responsibility — a class shouldn't change for more than one reason.
```ts
class OnlineShop {
  private orders: any;
  private offeredProducts: any;
  private customers: any;

  public addProduct(title: string, price: number) {}  // offeredProducts

  public updateProduct(productId: string, title: string, price: number) {}  // offeredProducts

  public removeProduct(productId: string) {}  // offeredProducts

  public getAvailableItems(productId: string) {}  // offeredProducts

  public restockProduct(productId: string) {}  // offeredProducts

  public createCustomer(email: string, password: string) {}  // customers

  public loginCustomer(email: string, password: string) {}  // customers

  public makePurchase(customerId: string, productId: string) {}  // customers, orders, offeredProducts

  public addOrder(customerId: string, productId: string, quantity: number) {} // customers, orders, offeredProducts

  public refund(orderId: string) {}  // customers, orders

  public updateCustomerProfile(customerId: string, name: string) {}  // customers
}
```
This class's got three properties, a bunch of methods but it doesn't have just a single responsibility. It's dealing with orders, products, customers, and has multiple purposes.

Consider split the responsibilities, to have several smaller classes like below:
```ts
class Order {
  public refund() {}
}


class Customer {
  private orders: Order[];

  constructor(email: string, password: string) {}

  public login(email: string, password: string) {}

  public updateProfile(name: string) {}

  public makePurchase(productId: string) {}
}


class Product {
  constructor(title: string, price: number) {}

  public update(Id: string, title: string, price: number) {}

  public remove(Id: string) {}
}


class Inventory {
  private products: Product;

  public getAvailableItems(productId: string) {}

  public restockProduct(productId: string) {}
}
```

### O — The Open-Closed Principle (OCP)
> A class should be open for extension but closed for modification.

**Examples**
```ts
class Printer {
  printPDF(data: any) {
  }

  printWebDocument(data: any) {
  }

  printPage(data: any) {
  }

  verifyData(data: any) {
  }
}
```
If we want to, says adding a method for Microsft Word or Excel, we need to go back to this class every time to implement new methods.

Once the API is finished, we should close all the implemented `Printer` classes, and only open them for edit when doing bug fixing.

To add a new feature, we `extends` the existing parent class. Instead of having a single `Printer` class, specific subclasses are encouraged to implement.
```ts
interface Printer {
  print(data: any);
}


class PrinterImplementation {
  verifyData(data: any) {}
}

class WebPrinter extends PrinterImplementation implements Printer {
  print(data: any) {
  }
}


class PDFPrinter extends PrinterImplementation implements Printer {
  print(data: any) {
  }
}


class PagePrinter extends PrinterImplementation implements Printer {
  print(data: any) {
  }
}
```

**Why?**
- Extensibility ensures small classes (instead of growing classes) and helps prevent code duplication (DRY).
- Smaller clasess and DRY code increase readability and maintainability.

### L — Liskov Substitution Principle (LSP)
> Objects should be replaceable with instances of their subclasses without altering the behavior.
Model your data correctly, either `flyingBird.fly()`, `eagle.fly()`.
```js
class Bird {}

class FlyingBird {
  fly() { return true; }
}

class Eagle extends FlyingBird {
  dive() { ... }
}

function tryToFly(anyFlyingBird) {
  if (anyFlyingBird.fly()) {
    return "Flying";
  }
}

flyingBird = FlyingBird();
eagle = Eagle();
asssert tryToFly(flyingBird) == tryToFly(eagle);

class Penguin extends Bird {}
```

### I — Interface Segregation Principle
> Many client-specific interfaces are better than one general purpose interface.
```ts
interface Database {
  connect(uri: string);
  storeData(data: any);
}

class SQLDatabase implements Database {
  connect(uri: string) { ... }

  storeData(data: any) { ... }
}

class InMemoryDatabase implements Database {
  connect(uri: string) { ... }
}
```
Now the `InMemoryDatabase` should implements `storeData` since it `implements Database`, but the `storeData` doesn't make any sense since the database exists in memory only.
To make the code cleaner, consider separate interfaces.

```ts
interface Database {
  connect(uri: string);
}

interface RemoteDatabase {
  storeData(data: any);
}

class SQLDatabase implements Database, RemoteDatabase { ... }

class InMemoryDatabase implements Database { ... }
```

### D — Dependency Inversion Principle (DIP)
> You should depend upon abstractions, not concretions.
```ts
class App {
  private database: SQLDatabase | InMemoryDatabase;

  constructor(database: SQLDatabase | InMemoryDatabase) {
  }

  saveSettings() {
    this.database.storeData("Some data");
  }
}
```
This example depends on concretions, in which it has to behave base on the correct subclass instance given.

The code is hard to maintain because the more types and databases added in, the more if-checks also have to be added.

The fix below applies DIP.
```ts
class App {
  private database: Database;

  constructor(database: Database) {
    this.database = database;
  }

  saveSettings() {
    this.database.storeData("Some data");
  }
}


const sqlDatabase = new SQLDatabase();
sqlDatabase.connect("some-url");
const app = new App(sqlDatabase);
```
So that the App doesn't have to care which database to pass in.
