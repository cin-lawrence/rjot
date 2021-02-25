# Clean code: Object-oriented approach

## 5. Conditionals & Error handling
- Avoid deep nesting.
- Using factory functions & polymorphism.
- Prefer positive checks (`isEmpty(...)` instead of `isNotEmpty(...)`).
- Utilize errors.

### 5.1. Guards
Use Guards and Fail fast.
```js
if (email.includes('@')) {
  // do stuff
}
```
Using a guard unwraps a level of bracket, also help to **fail fast.**
```js
// Guard
if (!email.includes('@')) {
  return;
}

// do stuff
```

### 5.2. Deep nesting
#### 5.2.1. Extracting Control Structures into Functions
#### 5.2.2. Inverting conditional logic

### 5.3. Error handling
> Throwing + handling errors can replace if statements and lead to more focused functions.

> Simple rule: If something is an error, make it an error.

```js
if (!isEmail(email)) {
  throw new ValidationError("Invalid input");
}
```

so that we can do
```js
try {
  processTransactions(transactions);
} catch (error) {
  if (error instanceof ValidationError) {
    showErrorMessage("Invalid transactions");
  }
}
```

#### Error handling is "One Thing"
**Not doing anything in addition to a try-catch block.**

**Examples** Consider this function:

```js
function processTransactions(transactions) {
  validateTransactions(transactions);

  for (const transaction of transactions) {
    try {
      processTransaction(transaction);
    } catch (error) {
      showErrorMessage(error.message, error.item);
    }
}
```

This function does more than just error handling. One way to work around is to move the try-catch block into the `processTransaction(...)` function.

## 5.4. Factory functions
A function that returns another set of functions when there are many blocks of code that share the same logic and just differ by property types or depends on some split logic.

```js
// Factory function
function getTransactionProcessors(transaction) {
  let processors = {
    processPayment: null,
    processRefund: null,
  };

  if (usesTransactionMethod(transaction, 'CREDIT_CARD')) {
    processors.processPayment = processCreditCardPayment;
    processors.processRefund = processCreditCardRefund;
  } else if (usesTransactionMethod(transaction, 'PAYPAL')) {
    processors.processPayment = processPayPalPayment;
    processors.processRefund = processPayPalRefund;
  } else if (usesTransactionMethod(transaction, 'PLAN')) {
    processors.processPayment = processPlanPayment;
    processors.processRefund = processPlanRefund;
  }

  return processors;
}
```

## 6. Classes & Data Structures

### 6.1. Object vs. Data Container (Data Structure/Dataclass)

|Object|Data Container/Data Structure|
|:-|:-|
|Private internals/properties, public API (methods)|Public internals/properties, (almost) not API (methods)|
|Contain your business logic (in OOP)|Store and transport data|
|Abstractions over concretions|Concretions only|

**Examples**
```ts
class Database {
  private uri: string;
  private provider: any;
  private connection: any;

  constructor(uri: string, provider: any) {
    this.uri = uri;
    this.provider = provider;
  }

  connect() {
    ...
  }

  disconnect() {
    ...
  }
}
```
The `Database` class is hiding its internal, whilst expose only `connect(...)` and `disconnect(...)` methods which are high-level methods, which then internally do everything to set up a connection or to close a connection. We don't know what exactly the class will to internally. That why it's got abstractions over concretions.

```ts
class UserCredentials {
  public email: string;
  public password: string;
}
```
On the other hand, `UserCredentials` just stores `email` and `password` and act as a data container, like `Struct` in C++.

Mixing these concepts might leads to writing bad code.

### 6.2. Polymorphism

**Before**
```ts
class Delivery {
  private purchase: Purchase;

  constructor(purchase: Purchase) {
    this.purchase = purchase;
  }

  deliverProduct() {
    if (this.purchase.deliveryType === 'express') {
      Logistics.issueExpressDelivery(this.purchase.product);
    } else if (this.purchase.deliveryType === 'insured') {
      Logistics.issueInsuredDelivery(this.purchase.product);
    } else {
      Logistics.issueStandardDelivery(this.purchase.product);
    }
  }

  trackProduct() {
    if (this.purchase.deliveryType === 'express') {
      Logistics.trackExpressDelivery(this.purchase.product);
    } else if (this.purchase.deliveryType === 'insured') {
      Logistics.trackInsuredDelivery(this.purchase.product);
    } else {
      Logistics.trackStandardDelivery(this.purchase.product);
    }
  }
}
```

**After**
```ts
type Purchase = any;

let Logistics: any;

interface Delivery {
  deliverProduct();
  trackProduct();
}


class DeliveryImplementation {
  protected purchase: Purchase;

  constructor(purchase: Purchase) {
    this.purchase = purchase;
  }
}


class ExpressDelivery extends DeliveryImplementation implements Delivery{
  deliverProduct() {
    Logistics.issueExpressDelivery(this.purchase.product);
  }

  trackProduct() {
    Logistics.trackExpressDelivery(this.purchase.product);
  }
}


class InsuredDelivery extends DeliveryImplementation implements Delivery {
    Logistics.issueInsuredDelivery(this.purchase.product);
  }

  trackProduct() {
    Logistics.trackInsuredDelivery(this.purchase.product);
  }
}


class StandardDelivery extends DeliveryImplementation implements Delivery {
    Logistics.issueStandardDelivery(this.purchase.product);
  }

  trackProduct() {
    Logistics.trackStandardDelivery(this.purchase.product);
  }
}


function createDelivery(purchase) {
  if (purchase.deliveryType === 'exporess') {
    delivery = new ExpressDelivery(purchase);
  } else if (purchase.deliveryType === 'insured') {
    delivery = new InsuredDelivery(purchase);
  } else {
    delivery = new StandardDelivery(purchase);
  }
  return delivery;
}


let delivery: Delivery = createDelivery({});

delivery.deliveryProduct();
```

### 6.3. Missing Distinction & Bloated Classes
> **Classes should be small.**
- You typically should prefer many small classes over a few large classes.
- Classes should have a single responsibility (SRP of SOLID).

**Examples** A Product class is responsible for product "issues" (*e.g.* change the product name).

### 6.4. Cohesion
Similar to levels of abstraction in functions, classes have their counterpart of cohesion.

> How much are your class methods using the class properties.
> We want to write highly cohesive classes.

|Maximum cohesion|No cohesion|
|:-|:-|
|All methods each use all properties|All methods don't use any class properties|
|A highly cohesive object|Data structure/container with utility methods|

In the `OnlineShop` example of `Single-Responsibility Principle`, most of the class methods are using just 1 of 3 given properties (`orders`, `offeredProducts`, and `customers`). Hence, it doesn't have great cohesion.
If more than 70% of the methods are using 2 over 3 of the given properties, that's okay. Otherwise, consider splitting the class.

### 6.5. The Law of Demeter
```js
this.customer.lastPurchase.date
```
> Drilling deeply into other objects is discouraged by the Law of Demeter.

> **Principle of Least Knowledge:** Don't depend on the internals of "strangers" (other objects which you don't directly know).
> It encourages to not rely too much on the internals of strangers of other objects.

A `customer` would be a class property, so interacting with it is fine, and accessing `lastPurchase` is fine, but `lastPurchase` is a stranger.

> Code in a method may only access direct internals (properties and methods) of:
> - The object it **belongs to.**
> - Objects that are **stored in properties of that object.**
> - Objects which are **received as method parameters.**
> - Objects which are **created in the method.**

**Why?**
- When something changes, all place where this line of code presents needs to change as well.
- The code will then be harder to maintain or extend.
- Statements like these are harder to read, analyzing such lines of code takes longer.

**Examples**
```ts
class DeliveryJob {
  ...
  deliverLastPurchase() {
    const date = this.customer.lastPurchase.date;
    this.warehouse.deliverPurchasesByDate(this.customer, date);
  }
}
```

To make this code clean, in the first approach, `lastPurchase.date` is outsourced into a new method of the `Customer`.
`getLastPurchaseDate(...)` internally access the `date` of `lastPurchase`, the problem seems solved.

```ts
class DeliveryJob {
  ...

  deliverLastPurchase() {
    const date = this.customer.getLastPurchaseDate();
    this.warehouse.deliverPurchasesByDate(this.customer, date);
  }
}

```

But actually, the problem is outsourced into `Customer`, we might then end up with plenty of helper methods.
The better solution to really make the code clean follows the principle: **Tell, Don't Ask.**

### 6.6. Tell, Don't Ask
> Don't ask for information then use it, but instead, simply tell other classes what to do if that is possible.
Instead of trying to get `date`, we can just forward the `lastPurchase` into `deliverPurchases(...)`.
```ts
class DeliveryJob {
  ...
  deliverLastPurchase() {
    this.warehouse.deliverPurchase(this.customer.lastPurchase);
  }
}
```

## 7. SOLID
See `clean-code-solid.md`

## 8. The complete example

### Before
```js
function processTransactions(transactions) {
  if (transactions && transactions.length > 0) {
    for (const transaction of transactions) {
      if (transaction.type === 'PAYMENT') {
        if (transaction.status === 'OPEN') {
          if (transaction.method === 'CREDIT_CARD') {
            processCreditCardPayment(transaction);
          } else if (transaction.method === 'PAYPAL') {
            processPayPalPayment(transaction);
          } else if (transaction.method === 'PLAN') {
            processPlanPayment(transaction);
          }
        } else {
          console.log('Invalid transaction type!', transaction);
        }
      } else if (transaction.type === 'REFUND') {
        if (transaction.status === 'OPEN') {
          if (transaction.method === 'CREDIT_CARD') {
            processCreditCardRefund(transaction);
          } else if (transaction.method === 'PAYPAL') {
            processPayPalRefund(transaction);
          } else if (transaction.method === 'PLAN') {
            processPlanRefund(transaction);
          }
        } else {
          console.log('Invalid transaction type!', transaction);
        }
      } else {
        console.log('Invalid transaction type!', transaction);
      }
    }  // for
  } else {
    console.log('No transactions provided!);
  }
}
```

### After
```js
class ValidationError extends Error {
  constructor(code, message) {
    this.code = code;
    this.message = message;
  }
}


// For equal levels of abstraction && apply positive checks
function isEmpty(transaction) {
  return !transaaction || transactions.length === 0;
}


function isOpen(transaction) {
  return transaction.status === 'OPEN';
}


function isPayment(transaction) {
  return transaction.type === 'PAYMENT';
}


function isRefund(transaction) {
  return transaction.type === 'REFUND';
}


function usesCreditCard(transaction) {
  return transaction.method === 'CREDIT_CARD';
}


function usesPayPal(transaction) {
  return transaction.method === 'PAYPAL';
}


function usesPlan(transaction) {
  return transaction.method === 'PLAN';
}


function usesTransactionMethod(transaction, method) {
  return transaction.method === method;
}


function showErrorMessage(message, item = {}) {
  console.error(message);
  console.error(item);
}


function processCreditCardTransaction(transaction) {
  if (isPayment(transaction)) {
    processCreditCardPayment(transaction);
  } else if (isRefund(transaction)) {
    processCreditCardRefund(transaction);
  }
}


function processPayPalTransaction(transaction) {
  if (isPayment(transaction)) {
    processPayPalPayment(transaction);
  } else if (isRefund(transaction)) {
    processPayPalRefund(transaction);
  }
}


function processPlanTransaction(transaction) {
  if (isPayment(transaction)) {
    processPlanPayment(transaction);
  } else if (isRefund(transaction)) {
    processPlanRefund(transaction);
  }
}


function validateTransaction(transaction) {
  if (!isOpen(transaction)) {
    throw new ValidationError(2, "Invalid transaction type!");
  }

  if (!isPayment(transaction) && !isRefund(transaction)) {
    throw new ValidationError(2, "Invalid transaction type!");
  }
}

function validateTransactions(transactions) {
  // Using guard
  if (isEmpty(transactions)) {
    throw new ValidationError(1, "No transactions provided!");
  }
}


class Transaction {
  processPayment() { ... }
  processRefund() { ... }
}


class CreditCardTransaction extends Transaction {
  processPayment() { ... }
  processRefund() { ... }
}

class PayPalTransaction extends Transaction {
  processPayment() { ... }
  processRefund() { ... }
}

class PlanTransaction extends Transaction {
  processPayment() { ... }
  processRefund() { ... }
}


function getTransacction(transaction) {
  if (usesTransactionMethod(transaction, 'CREDIT_CARD')) {
    return CreditCardTransaction(transaction);
  } else if (usesTransactionMethod(transaction, 'PAYPAL')) {
    return PayPalTransaction(transaction);
  } else if (usesTransactionMethod(transaction, 'PLAN')) {
    return PlanTransaction(transaction);
  }
}


function processWithInstance(transaction) {
  const transactionObject = getTransaction(transaction);
  if (isPayment(transaction)) {
    transactionObject.processPayment(transaction);
  } else {
    transactionObject.processRefund(transaction);
  }
}


function processTransaction(transaction) {
  try {
    validateTransaction(transaction);
    processWithInstance(transaction);
  } catch (error) {
    showErrorMessage(error.message, error.item);
  }
}


function processTransactions(transactions) {
  // Use validator instead
  validateTransactions(transactions);

  for (const transaction of transactions) {
    processTransaction(transaction);
  }
}

```
