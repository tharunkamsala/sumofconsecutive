# Delphi/Object Pascal Interpreter

A Delphi/Object Pascal interpreter built with **ANTLR4** and **Java**, extending a Pascal grammar with object-oriented features.

## What Works

### Core Requirements

| Feature | Status | Description |
|---------|--------|-------------|
| **Classes and Objects** | ✅ Working | Class definitions with fields and methods, object instantiation via `ClassName.Create(...)`, field access via `obj.field` and `SELF.field` |
| **Constructors and Destructors** | ✅ Working | `CONSTRUCTOR Create(...)` for initialization, `DESTRUCTOR Destroy` called via `obj.Free` |
| **Encapsulation** | ✅ Working | Visibility modifiers: `PRIVATE`, `PROTECTED`, `PUBLIC`, `PUBLISHED` for fields and methods |
| **Terminal I/O** | ✅ Working | `WriteInt(n)` prints integers, `WriteLn` prints newline, `ReadInt(var)` reads integers from terminal |

### Bonus Features (20%)

| Feature | Status | Description |
|---------|--------|-------------|
| **Inheritance** | ✅ Working | Subclasses via `ChildClass = CLASS(ParentClass)`, `INHERITED` keyword for calling parent methods, inherited field and method access |
| **Interfaces** | ✅ Working | Interface definitions via `INTERFACE...END`, classes can implement interface methods |

### Additional Features

- Variable declarations and assignments
- Arithmetic expressions (`+`, `-`, `*`, `/`, `DIV`, `MOD`)
- Relational operators (`=`, `<>`, `<`, `<=`, `>`, `>=`)
- Control flow: `IF/THEN/ELSE`, `WHILE/DO`, `REPEAT/UNTIL`, `FOR/TO/DOWNTO`
- Boolean logic (`AND`, `OR`, `NOT`, `TRUE`, `FALSE`)
- String literals
- Constants
- Function return values via Delphi convention (`FunctionName := value`)

## Prerequisites

- **Java JDK 11** or higher
- **Apache Maven 3.6+**

## How to Run

### Step 1: Build the project

```bash

mvn clean package
```

This will:
1. Generate the lexer and parser from `delphi.g4` using ANTLR4
2. Compile all Java source files
3. Package everything and copy dependencies

### Step 2: Run a test file

```bash
java -cp "target/classes:target/dependency/*" delphi.DelphiInterpreter tests/test1.pas
```

Replace `test1.pas` with any test file (`test1.pas` through `test7.pas`).

**Alternative classpath** (if `target/dependency` is not available): use the ANTLR runtime from your local Maven repo:

```bash
export CP="target/classes:$HOME/.m2/repository/org/antlr/antlr4-runtime/4.13.1/antlr4-runtime-4.13.1.jar"
java -cp "$CP" delphi.DelphiInterpreter tests/test1.pas
```

**Note:** Tests 4 and 7 (bonus tests for inheritance and interfaces) require keyboard input. You can provide input via pipe:

```bash
echo "42" | java -cp "target/classes:target/dependency/*" delphi.DelphiInterpreter tests/test4.pas
echo "5" | java -cp "target/classes:target/dependency/*" delphi.DelphiInterpreter tests/test7.pas
```

### Run all tests at once

Tests 4 and 7 need stdin: use input `42` for test4 and `5` for test7 so output matches the expected values below.

```bash
for i in 1 2 3 4 5 6 7; do
  echo "=== Test $i ==="
  if [ $i -eq 4 ]; then
    echo "42" | java -cp "target/classes:target/dependency/*" delphi.DelphiInterpreter tests/test$i.pas
  elif [ $i -eq 7 ]; then
    echo "5" | java -cp "target/classes:target/dependency/*" delphi.DelphiInterpreter tests/test$i.pas
  else
    java -cp "target/classes:target/dependency/*" delphi.DelphiInterpreter tests/test$i.pas
  fi
  echo "---"
done
```

## Test Cases

### test1.pas — Classes and Objects
Tests basic class definition, constructor with parameters, field assignment via `SELF`, and method calls.
- Creates a `Point` class with `x, y` fields
- Constructor sets fields: `SELF.x := xVal`
- `Display` method prints field values using `WriteInt`
- **Expected output:** `1020` (10 and 20 printed together, followed by newline)

### test2.pas — Constructors and Destructors
Tests constructor initialization, method calls (increment), function return values, and destructor invocation via `Free`.
- Creates a `Counter` class with private `count` field
- Constructor initializes count, `Increment` adds 1, `GetValue` returns count
- `Destroy` destructor prints `999` when `c.Free` is called
- **Expected output:** `7` (newline) `999` (newline)

### test3.pas — Encapsulation
Tests visibility modifiers (`PRIVATE`, `PROTECTED`, `PUBLIC`) on fields.
- Creates a `BankAccount` class with private `balance` and protected `accountNumber`
- Public methods `Deposit` and `GetBalance` access private fields
- **Expected output:** `1500` (1000 initial + 500 deposit)

### test4.pas — Inheritance with ReadInt (Bonus)
Tests class inheritance, `INHERITED` keyword, and terminal input (`ReadInt`).
- `Animal` base class with `sound` field, `Create` constructor, and `GetSound` function
- `Dog` subclass calls `INHERITED Create(s)` to set parent fields
- Overrides `Speak` procedure to print the sound value
- Reads an integer from terminal input using `ReadInt`
- Tests that inherited methods (`GetSound`) work correctly on subclass instances
- **Sample run with input `42`:** Output is `0` (prompt), `42` (Dog.Speak), `42` (Dog.GetSound via inherited method)

### test5.pas — Basic Integer I/O
Tests basic variable declaration, arithmetic, and terminal output.
- Declares integer variables, performs addition
- Uses `WriteInt` and `WriteLn` for output
- **Expected output:** `10` (newline) `20` (newline) `40` (newline)

### test6.pas — Methods and Field Mutation
Tests getter/setter methods and object field mutation.
- Creates a `Rectangle` class with private `width` and `height`
- `GetArea` returns `width * height`
- `SetWidth` and `SetHeight` modify fields
- **Expected output:** `50` (5×10) (newline) `96` (8×12) (newline)

### test7.pas — Interfaces with ReadInt (Bonus)
Tests interface definition, multiple classes implementing the same interface, and terminal input (`ReadInt`).
- Defines a `Drawable` interface with a `Draw` procedure
- `Circle` class implements `Draw` to print its radius
- `Square` class implements `Draw` to print side × side (area)
- Reads an integer from terminal input using `ReadInt`
- **Sample run with input `5`:** Output is `0` (prompt), `5` (Circle.Draw), `25` (Square.Draw = 5×5)

## What to Turn In

Bundle these in a **.zip** archive for submission:

- `delphi.g4` — grammar
- All Java source files under `src/main/java/delphi/`
- `pom.xml` — Maven build
- `tests/test1.pas` … `tests/test7.pas` — test cases
- `README.md` — this file (what works and how to run)

Exclude the `target/` directory when creating the zip (it is regenerated by `mvn clean package`).

## Project Structure

```
project 1/
├── delphi.g4                          # ANTLR4 grammar (Delphi/Object Pascal)
├── pom.xml                            # Maven build configuration
├── README.md                          # This file
├── src/main/java/delphi/
│   ├── DelphiInterpreter.java         # Main entry point
│   ├── DelphiASTVisitor.java          # AST visitor (interpreter logic)
│   ├── Environment.java               # Variable/scope management
│   ├── DelphiClass.java               # Class definition representation
│   ├── DelphiInstance.java            # Runtime object instance
│   └── DelphiInterface.java           # Interface definition representation
├── tests/
│   ├── test1.pas                      # Classes and Objects
│   ├── test2.pas                      # Constructors and Destructors
│   ├── test3.pas                      # Encapsulation
│   ├── test4.pas                      # Inheritance + ReadInt (Bonus)
│   ├── test5.pas                      # Basic Integer I/O
│   ├── test6.pas                      # Methods and Field Mutation
│   └── test7.pas                      # Interfaces + ReadInt (Bonus)
└── target/                            # Build output (auto-generated)
    └── generated-sources/antlr4/      # ANTLR-generated parser/lexer
```

## How It Works

1. **Grammar (`delphi.g4`)** — Defines the syntax rules for the Delphi language, extending a standard Pascal grammar with OOP features (classes, constructors, destructors, visibility modifiers, inheritance, interfaces).

2. **ANTLR4** — Reads the grammar and auto-generates a lexer (tokenizer) and parser. The parser produces a parse tree (AST) from source `.pas` files.

3. **Interpreter (`DelphiASTVisitor.java`)** — Walks the AST using the Visitor pattern. Each `visit` method handles a grammar rule: evaluating expressions, executing statements, managing objects, calling methods, etc.

4. **Runtime (`Environment.java`, `DelphiClass.java`, `DelphiInstance.java`)** — Manages the runtime state: variable scopes, class definitions, and live object instances with their field values.
