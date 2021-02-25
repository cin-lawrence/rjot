# Clean Code

## What is clean code
- It should be **readable and meaningful.**
- It should **reduce cognitive load.**
- It should be **concise** and **"to the point".**
- It should **avoid unintuitive names, complex nesting, and big code blocks.**
- It should follow **common best practices and patterns.**
- It should be **fun to write and to maintain.**

## Compare to clean architecture
- Clean code helps to write code that is readable & easy to understand.
- Patterns & Principles help to write code that is maintainable and extensible.
- Clean architecture helps to find where to write which code.

This document lists out all pain points and how to deal with these.

## 1. Names
### 1.1. Assigning good names
> **Always argue for more details until you think it's enough.**

#### 1.1.1. Variables, Constants & Properties
- Use **nouns** or **short phrases with adjectives.**

|Value Type|How to describe|Sample|
|:-|:-|:-|
|Object|Describe the value, provide more details without introducing redundancy|`user`, `database`, `authenticatedUser`, `sqlDatabase`|
|Number or String|Describe the value, provide more details without introducing redundancy|`name`, `firstName`, `age`|
|Boolean|Answer a true/false question|`isActive`, `loggedIn`|

#### 1.1.2. Functions, methods
- Use **verbs** or **short phrases with adjectives.**
- `validate(...)` can be used if the function tends to throw `Exception`.

|Function Type|How to describe|Sample|
|:-|:-|:-|
|Performs an operation|Describe the operation, provide more details without introducing redundancy|`getUser(...)`, `response.send()`, `getUserByEmail(...)`|
|Computes a Boolean|Answer a true/false question, prodive more details without introducing redendancy|`isValid(...)`, `purchase.isPaid()`, `emailIsValid(...)`|

#### 1.1.3. Classes
- Use **nouns** or **short phrases with nouns.**
- Describe the Object.
- Provide more details without introducing redundancy.
- Avoid redundant suffixes.

**Examples:**
+ `User`, `Product`
+ `Customer`, `Course`
+ `Database`
- `DatabaseManager` → there should be a better name which is more clear

#### 1.1.4. Exceptions
- Having a good reason to not following naming convention.
- Naming a Utility class, with many static/class methods.

**Examples:** `class DateUtil {}`

- Naming getter, setter, @property, the method has its name as a property.

**Examples:** `Database.connectedClient(...)`

### 1.2. Common mistakes & pitfalls

#### 1.2.1. Include redundant information in names
**Examples:** `userWithNameAndAge = User('Max', 31)`
- Even without knowing the class definition, it's easy to guess that this user has a name and age.
- In general, it's expected that a `User` will contain some user data.
- We should look into the class definition if we want to learn more about the "User" object.

#### 1.2.2. Avoid Slang, Unclear abbreviations & Disinformation

**Slang:**

`product.diePlease()`, `user.facePalm()`

→ `product.remove()`, `user.sendErrorMessage()`

**Unclear Abbreviations:**

`message(n)`, `ymdt = '20210121CET`

→ `message(newUser)`, `dateWithTimezone = '20210121CET'`

**Disinformation:**

`userList = { u1: ..., u2: ... }`, `allAccounts = accounts.filter()`

→ `userMap = { u1: ..., u2: ... }`, `filteredAccounts = accounts.filter()`

#### 1.2.3. Indistinctive names

|Names to avoid|Correction|
|:-|:-|
|`analytics.getDailyData(day)`|`analytics.getDailyReport(day)`|
|`analytics.getDayData(day)`|`analytics.getDailyReport(day)`|
|`analytics.getDailyData(day)`|`analytics.getDailyReport(day)`|
|`analytics.getDailyData(day)`|`analytics.getDailyReport(day)`|

#### 1.2.4. Inconsistent
Stick with the same pattern that you used.

**Examples:** `getUsers()`, `fetchUsers()`, `retrieveUsers()`

Use only one of these in the whole solution.

## 2. Structure & Comments

### 2.1. Code formatting

#### 2.1.1. Vertical formatting
> Code should be readable like an essay — top to bottom without too many "jumps".

- Consider splitting files with multiple concepts (*e.g.* classes) into multiple files
- Different concepts ("areas") should be separated by spacing.
- Similar concepts ("areas") should not be separated by spacing.
- Related concepts should be closed to each other.

#### 2.1.2. Horizontal formatting

> Lines of code should be readable without scrolling — avoid very long "sentences".

- Use indentation — even if not required technically.
- Break long statements into multiple shorter ones.
- Use clear but not unreadably long names.

### 2.2. Comments
#### 2.2.1. Good comments
**Legal information**
```js
// (c) Surge
// Created in 2021
```

**Explanations which can't be replaced by good naming**
```js
// Accepts [text]@[text].[text]
const emailRegex = /\S+@\S+\.\S+/;
```

**Warnings**
```js
// Only works in browser environment
localStorage.setItem('user', 'test@test.com');
```

**To-do Notes**
```
// TODO Implement a better logic
...
```

**Documentation**
```py
"""Returns a foobang

Optional plotz says to frobnicate the bizbaz first.
"""
```

#### 2.2.2. Bad Comments
**Redundant information**

`private dbDriver: any;  // the database engine to which we connect`

→ `private dbEngine: any;`

If possible, the name should be self-explanatory itself.

**Divider/Block markers**
```js
// ***************
// GLOBALS
// ***************
let sqlDriver: any;
let mongoDbDriver: any;
```
→ Just remove these.

**Misleading comments**
```js
insertData(data: any) {
  this.dbEngine.insert(data);  // update a user
}
```

**Commented-Out Code**

Use wisely and rely on version control system instead.

## 3. Functions
- Calling the function should be readable → The number and order of arguments matter.
- Working with the function should be easy/readable → The length of the function body matters.

### 3.1. Length

#### 3.1.1. Functions should be small
Easier to read and comprehend

#### 3.1.2. Functions should do exactly One Thing
**The principle of "One Thing"**
```js
function renderContent(renderInformation) {
  const element = renderInformation.element;
  const rootElement = renderInformation.root;

  validateElementType(element);

  const content = createRenderableContent(renderInformation);

  renderOnRoot(rootElement, content);
}
```
It is common to have functions that kick off multiple operations. Writing functions where every function has just one line of code is unrealistic and unhelpful.

**One thing** is not one operation, but one level of abstraction.
- **Different Operations** examples: Validate + Save User Input.
- **Different Levels of Abstraction** examples: `email.includes('@')` and `saveUser(email, password)` are different in abstraction level, in which the former is low level and the latter is high level.

> **Functions should do work that's one level of abstraction below their name.**

**DO** This function should return true/false based on the email validity.
```js
function isEmailValid(email) {
  return email.includes('@');
}
```

**DON'T** This function should orchestrate all the steps that are required to save a user.
```js
function saveUser(email) {
  if (email.includes('@')) { ... }
}
```

instead, outsource the `isEmailValid(..)`
```js
function saveUser(email) {
  if (isEmailValid(email)) { ... }
}
```

Another example, the first one requires reading, understanding and interpreting different steps.
```js
if (!email.includes('@')) {
  console.log('Invalid email!');
} else {
  const user = new User(email);
  user.save();
}
```

And here we just need to read different steps, which is much better when abstract functions out.
```js
if (!isEmail(email)) {
  showError('Invalid email!');
} else {
  saveNewUser(email);
}
```
#### 3.1.3. Keep functions short
**Rules of Thumb:**
- Extract code that works on the same functionality.
```js
// Before
user.setAge(31);
user.setName('Max');

// After
user.update({ age: 31, name: 'Max' });
```
- Extract code that requires more interpretation than the surrounding code.
```js
// Before
if (!email.includes('@')) { ... }
saveNewUser(email);

// After
if (!isValid(email)) { ... }
saveNewUser(email);
```

### 3.2. Parameters

#### 3.2.1. Minimize the number of parameters
- The more parameters a function uses, the harder it gets to call them.
- Hard to memorize which values should be passed, a lot more code to write, read, and keep in mind.
- Try to pass less than 3 parameters.

**Examples:**

The first approach with 2 parameters.
```js
function saveUser(email, password) {
  const user = {
    id: Math.random().toString(),
    email,
    password,
  };

  db.insert('users', user);
}
```

Refactoring to use only one parameter.
```js
# Assume a user object is created somewhere outside
function saveUser(user) {
  db.insert('users', user);
}
```

Adding a User class instead, so that `insert` takes no argument.
```js
class User {
  constructor(email, password) {
    this.email = email;
    this.password = password;
    this.id = Math.random().toString();
  }

  save() {
    db.insert('users', this);
  }
}
```

#### 3.2.2. One-parameter function
```js
function square(number) {
  return number * number;
}

function isEmailValid(email) {
  return email.includes('@');
}
```

#### 3.2.3. Output parameters
Try to avoid output arguments, especially if they are unexpected.

**Examples:**

The first approach — not great since the user gets modified unexpectedly.
```js
function createId(user) {
  user.id = 'u1';
}
const user = { name: 'Max' };
createId(user);
```

The second approach — implicitly implies that user object is going to be manipulated.
```js
function addId(user) {
  user.id = 'u1';
}
const user = { name: 'Max' };
addId(user);
```

The object-oriented approach — explicitly implies changes for the `user` object.
```js
class User {
  constructor(name) {
    this.name = name;
  }

  addId() {
    this.id = 'u1';
  }
}
const customer = new User('Max');
customer.addId();
```

### 3.3. DRY (Don't repeat yourself)
**Reusability matters:** Don't write the same code more than once

Two common signs of writing non-DRY code:
- Copying & pasting code.
- Applying the same change to multiple places in the codebase.

**Examples:**
```js
function createSupportChannel(email) {
  if (!email || !email.includes('@')) {
    console.log('Invalid email - could not create channel');
  }
}

function isInputValid(email, password) {
  return email && email.includes('@') && password && password.trim() !== '';
}
```

To stay DRY, the logic for `email` should be separated.
```js
function isEmailValid(email) {
  return email && email.includes('@');
}
```

Now we have mixed levels of abstraction in `isInputValid(...)`, hence `isPasswordValid(...)` has to be separated.
```js
function isPasswordValid(password) {
  return password && password.trim() !== '';
}

function isInputValid(email, password) {
  return isEmailValid(email) && isPassowrdValid(password);
}
```

#### 3.3.1. Split functions reasonably
> Being as **granular** as possible **won't automatically improve readability**.

Make reasonable decisions and don't split if:
- You are just renaming the operation.
- Finding the new function will take longer than reading the extracted code.
- You can't produce a reasonable name for the extracted function.

#### 3.3.2. Try keeping functions pure
**Pure Function:** The same input always yields the same output, and has no side effect. Impure functions are less predictable than pure functions.

**Side Effect:** An operation that does not just act on function inputs and change the function output but which instead changes the overall system/program state.

→ Side effects are not automatically bad, but unexpected side effects should be avoided since it's problematic.

**When you need a side effect:**
- Choose a function name that implies it.
- Move the side effect into another function/place.

**Examples:** An impure function
```js
function generateId(userName) {
  const id = userName + Math.random().toString();
  return id;
}
```

#### Bonus: Unit testing
> Can you easily test a function?
> - **Yes:** Great!
> - **No:** Consider splitting the function.
The more a function is split, the more unit test available to write.

**For more details on how should we use the Object-oriented approach, navigate to `clean-code-oop.md`.**
### 4. Object-oriented approach
### 5. Conditionals & Error Handling
### 6. Classes & Data Structures
