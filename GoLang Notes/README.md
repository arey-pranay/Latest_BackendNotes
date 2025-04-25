![image](https://github.com/user-attachments/assets/805e8628-21aa-4e20-a076-4d75888cc088)

# Functions AND Error(s)
![image](https://github.com/user-attachments/assets/e5df74eb-b244-4d37-af7a-0a0faebbbb01)
![image](https://github.com/user-attachments/assets/bcf24d13-022b-4c3d-b0a8-271ce60f1e54)

# Non-ASCII characters in string
![image](https://github.com/user-attachments/assets/307d68ab-71f5-4246-9450-7c5a00aedb95)


---

# 🧠 Ultimate Golang Backend Development Guide  
*For real-world backend development, database integration, and frontend API connections (e.g., with Next.js).*

---

## ✅ 1. Go Language Core: Syntax, Types, & Best Practices

---

### 🔸 Variable Declaration

```go
x := 42                 // inferred type (recommended inside functions)
var y int = 99          // explicit
var z float64           // default 0.0
```

Use `:=` inside functions only. For global or package level, use `var`.

---

### 🔸 Constants

```go
const Pi = 3.14
const (
  A = 1
  B = "hello"
)
```

Constants must be assigned at compile time. No computed expressions allowed like in JavaScript.

---

### 🔸 Primitive Types & Limits

| Type         | Description                   | Size (bits) | Range / Notes                            |
|--------------|-------------------------------|-------------|------------------------------------------|
| `int`        | Platform-dependent integer     | 32 or 64    | `-2^31 to 2^31-1` or `-2^63 to 2^63-1`    |
| `int8`       | Smallest signed int            | 8           | `-128 to 127`                            |
| `int16`      | Short signed int               | 16          | `-32,768 to 32,767`                      |
| `int32`      | Normal signed int              | 32          | `-2,147,483,648 to 2,147,483,647`        |
| `int64`      | Long signed int                | 64          | `-9.2e18 to 9.2e18`                      |
| `uint`       | Unsigned int                   | 32/64       | `0 to 4.29e9` or `0 to 1.84e19`          |
| `uint8`      | Alias for byte                 | 8           | `0 to 255`                               |
| `float32`    | 32-bit float                   | 32          | ~`±1.5e−45 to ±3.4e+38` (7 digits)       |
| `float64`    | 64-bit float                   | 64          | ~`±5.0e−324 to ±1.7e+308` (15 digits)    |
| `complex64`  | Real + Imaginary float32       | 64          | Rarely used                              |
| `complex128` | Real + Imaginary float64       | 128         | Rarely used                              |
| `bool`       | true or false                  | 1           | Only `true` or `false`                  |
| `string`     | UTF-8 sequence of bytes        | dynamic     | Immutable, internally `[]byte`          |
| `byte`       | Alias for `uint8`              | 8           | Use for raw data                        |
| `rune`       | Alias for `int32` (Unicode)    | 32          | Can store all Unicode, inc. emoji       |

> 🧠 **Runes vs bytes**: `byte` is one ASCII character (1 byte), `rune` is a Unicode character (up to 4 bytes in UTF-8, but typically 2+ bytes for non-ASCII like emoji or accented characters).

---

### 🔸 Structs

```go
type User struct {
  Name string
  Age  int
}
```

Use structs to define models like `User`, `Post`, etc.

- Can include `json` tags:
  ```go
  type User struct {
    Name string `json:"name"`
    Age  int    `json:"age"`
  }
  ```

---

### 🔸 Pointers

```go
x := 5
p := &x     // pointer to x
*p = 10     // modifies x
```

Used to avoid copying large structs, modify data in-place.

---

### 🔸 Functions

```go
func add(a int, b int) int {
  return a + b
}
```

- Return multiple values:
  ```go
  func divide(a, b int) (int, error) {
    if b == 0 {
      return 0, fmt.Errorf("division by zero")
    }
    return a / b, nil
  }
  ```

---

### 🔸 Defer

```go
func example() {
  defer fmt.Println("Done") // runs *after* function returns
  fmt.Println("Running")
}
```

Use for closing DBs, files, etc.

---

### 🔸 Maps

![image](https://github.com/user-attachments/assets/3a64b56f-9297-475f-a6e7-ab6817f55478)


```go
m := make(map[string]int)
m["apples"] = 5
delete(m, "apples")
val, ok := m["apples"]
```

---

### 🔸 Slices (Dynamic Arrays)

```go
nums := []int{1, 2, 3}
nums = append(nums, 4)
```

---

### 🔸 Interfaces

```go
type Animal interface {
  Speak() string
}
```

Implemented implicitly. No need for `implements`.

---

## 🏗️ 2. Backend-Ready Golang Features

---

### 🧵 Goroutines & Channels

```go
go someFunc() // run in background

c := make(chan int)
go func() {
  c <- 5
}()
val := <-c
```

Channels are used to safely share data between goroutines (like threads).

---

### 🗂 Packages

Organize code like:

```
/main.go
/controllers/
  user.go
/models/
  user.go
```

Use:
```go
import "myapp/controllers"
```

---

## 🗄️ 3. PostgreSQL Integration

---

### 🔧 Setup (Go + pgx or GORM)

Use GORM for easy ORM mapping:

```bash
go get gorm.io/gorm
go get gorm.io/driver/postgres
```

```go
dsn := "host=localhost user=postgres password=pass dbname=test port=5432 sslmode=disable"
db, err := gorm.Open(postgres.Open(dsn), &gorm.Config{})
```

---

### 🧱 Model + Migration

```go
type User struct {
  gorm.Model
  Name string
  Age  int
}
db.AutoMigrate(&User{})
```

---

### 📦 CRUD

```go
// CREATE
db.Create(&User{Name: "Alice", Age: 20})

// READ
var user User
db.First(&user, 1)

// UPDATE
db.Model(&user).Update("Age", 21)

// DELETE
db.Delete(&user)
```

---

## 🍃 4. MongoDB Integration

---

### 🔧 Setup

```bash
go get go.mongodb.org/mongo-driver/mongo
```

```go
client, _ := mongo.Connect(ctx, options.Client().ApplyURI("mongodb://localhost:27017"))
collection := client.Database("test").Collection("users")
```

---

### 📦 CRUD Example

```go
type User struct {
  Name string `bson:"name"`
  Age  int    `bson:"age"`
}
collection.InsertOne(ctx, user)
```

---

## 🔗 5. Connect Go Backend to Next.js Frontend

---

### 🌐 Go API Server (Basic)

```go
func handler(w http.ResponseWriter, r *http.Request) {
  json.NewEncoder(w).Encode(map[string]string{"message": "hello from go"})
}
```

```go
http.HandleFunc("/api", handler)
http.ListenAndServe(":8080", nil)
```

---

### ⚛️ Next.js API Fetch

```js
const res = await fetch("http://localhost:8080/api");
const data = await res.json();
```

✅ Use this to connect frontend and backend easily.

---

## 📊 6. Go vs JavaScript vs Java (Single Comparison Table)

| Feature              | Go                            | JavaScript                    | Java                             |
|----------------------|-------------------------------|--------------------------------|----------------------------------|
| Typing               | Static (strong)               | Dynamic                        | Static (strict)                 |
| Compilation          | Native binary                 | Interpreted                    | JVM Bytecode                    |
| OOP                  | Structs + interfaces           | Prototype + classes            | Classical OOP                   |
| Concurrency          | Goroutines + channels          | Async/Await                    | Threads, Executors              |
| Error Handling       | Error returns                  | try-catch                      | try-catch, exceptions           |
| Null Handling        | `nil`, Zero values             | `undefined`, `null`            | `null`                          |
| Memory Management    | GC built-in                    | GC via V8                      | GC via JVM                      |
| Web Usage            | Backend APIs, CLI, microservices | Fullstack, frontend, backend | Full backend, Android, etc.     |

---

## ✅ 7. Pro Backend Advice

- Always validate request bodies with struct + custom validation
- Use context (`context.Context`) for timeouts and cancellation
- Use GORM for SQL, but raw SQL (`pgx`) for performance-heavy queries
- In MongoDB, model indexes properly — Go driver supports them
- For large responses, stream using `http.ResponseWriter`
- Always close resources: DB, file, etc. (`defer ...Close()`)
