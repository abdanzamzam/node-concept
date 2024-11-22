
# Dependency Injection (DI)

## **Pengertian Dependency Injection (DI)**

**Dependency Injection (DI)** adalah sebuah teknik desain dalam pemrograman di mana sebuah objek menerima dependensinya dari sumber eksternal daripada membuat atau menginisialisasi dependensi tersebut sendiri. DI bertujuan untuk meningkatkan modularitas, meningkatkan kemampuan pengujian (testability), dan memisahkan tanggung jawab (separation of concerns).

### **Manfaat DI**
- Mempermudah pengujian unit (unit testing).
- Memungkinkan penggantian dependensi dengan mudah (mock atau implementasi lain).
- Mengurangi keterikatan (tight coupling) antara komponen.

---

## **Contoh dan Penjelasan**

### **1. Tanpa Dependency Injection**
Pendekatan tanpa DI langsung membuat dependensi di dalam kelas itu sendiri, sehingga tight coupling terjadi.

```javascript
class Database {
    findUser(userId) {
        return `User with ID ${userId}`;
    }
}

class UserService {
    constructor() {
        this.database = new Database(); // Hardcoded dependency
    }

    getUser(userId) {
        return this.database.findUser(userId);
    }
}

// Penggunaan
const userService = new UserService();
console.log(userService.getUser(1));
```

- **Kelebihan**: Mudah dipahami dan diimplementasikan untuk kasus sederhana.
- **Kekurangan**: Sulit diuji, sulit mengganti dependensi, menyebabkan tight coupling.

---

### **2. Dengan Dependency Injection**

#### **a. Constructor Injection**
Dependensi disuntikkan melalui constructor.

```javascript
class UserService {
    constructor(database) {
        this.database = database;
    }

    getUser(userId) {
        return this.database.findUser(userId);
    }
}

class Database {
    findUser(userId) {
        return `User with ID ${userId}`;
    }
}

// Penggunaan
const database = new Database();
const userService = new UserService(database);
console.log(userService.getUser(1));
```

- **Kelebihan**:
  - Dependensi eksplisit.
  - Mudah diuji dan mengganti dependensi.
- **Kekurangan**:
  - Jika ada banyak dependensi, constructor bisa menjadi terlalu panjang (constructor bloat).

---

#### **b. Method Injection**
Dependensi disuntikkan melalui parameter metode.

```javascript
class UserService {
    getUser(userId, database) {
        return database.findUser(userId);
    }
}

class Database {
    findUser(userId) {
        return `User with ID ${userId}`;
    }
}

// Penggunaan
const database = new Database();
const userService = new UserService();
console.log(userService.getUser(1, database));
```

- **Kelebihan**:
  - Cocok untuk dependensi sementara yang mungkin berubah.
- **Kekurangan**:
  - Dependensi kurang terlihat pada saat instansiasi.
  - Bisa menyebabkan masalah jika developer lupa menyuntikkan dependensi.

---

#### **c. Property Injection**
Dependensi disuntikkan melalui properti, biasanya dengan setter.

```javascript
class UserService {
    setDatabase(database) {
        this.database = database;
    }

    getUser(userId) {
        return this.database.findUser(userId);
    }
}

class Database {
    findUser(userId) {
        return `User with ID ${userId}`;
    }
}

// Penggunaan
const database = new Database();
const userService = new UserService();
userService.setDatabase(database);
console.log(userService.getUser(1));
```

- **Kelebihan**:
  - Fleksibel, memungkinkan dependensi diubah saat runtime.
- **Kekurangan**:
  - Dependensi tidak eksplisit, sulit mendeteksi jika belum disuntikkan.

---

#### **d. Interface Injection**
Dependensi disuntikkan melalui antarmuka (interface) yang diimplementasikan oleh penerima.

```javascript
class DatabaseInterface {
    injectDependency(database) {
        throw new Error("This method should be implemented");
    }
}

class UserService extends DatabaseInterface {
    injectDependency(database) {
        this.database = database;
    }

    getUser(userId) {
        return this.database.findUser(userId);
    }
}

class Database {
    findUser(userId) {
        return `User with ID ${userId}`;
    }
}

// Penggunaan
const database = new Database();
const userService = new UserService();
userService.injectDependency(database);
console.log(userService.getUser(1));
```

- **Kelebihan**:
  - Memastikan dependensi mengikuti kontrak tertentu.
- **Kekurangan**:
  - Membutuhkan boilerplate tambahan.

---

#### **e. Context Injection**
Dependensi dikelola oleh konteks atau IoC container (biasanya digunakan dalam framework).

**Contoh dengan NestJS:**
```javascript
import { Injectable } from '@nestjs/common';

@Injectable()
class Database {
    findUser(userId) {
        return `User with ID ${userId}`;
    }
}

@Injectable()
class UserService {
    constructor(database) {
        this.database = database;
    }

    getUser(userId) {
        return this.database.findUser(userId);
    }
}
```

- **Kelebihan**:
  - Automasi pengelolaan dependensi oleh framework.
- **Kekurangan**:
  - Membutuhkan framework dan konfigurasi tambahan.

---

#### **f. Service Locator**
Dependensi dikelola oleh service locator atau IoC container.

```javascript
class ServiceLocator {
    constructor() {
        this.services = {};
    }

    register(name, instance) {
        this.services[name] = instance;
    }

    get(name) {
        return this.services[name];
    }
}

class Database {
    findUser(userId) {
        return `User with ID ${userId}`;
    }
}

class UserService {
    constructor(locator) {
        this.database = locator.get('Database');
    }

    getUser(userId) {
        return this.database.findUser(userId);
    }
}

// Penggunaan
const locator = new ServiceLocator();
locator.register('Database', new Database());

const userService = new UserService(locator);
console.log(userService.getUser(1));
```

- **Kelebihan**:
  - Centralized dependency management.
- **Kekurangan**:
  - Dependency injection tidak eksplisit.

---

### **3. Dependency Injection dengan Library**
Library seperti **InversifyJS** atau **NestJS** mempermudah pengelolaan DI.

**Contoh dengan InversifyJS:**
```javascript
const { Container, injectable, inject } = require("inversify");

@injectable()
class Database {
    findUser(userId) {
        return `User with ID ${userId}`;
    }
}

@injectable()
class UserService {
    constructor(@inject("Database") database) {
        this.database = database;
    }

    getUser(userId) {
        return this.database.findUser(userId);
    }
}

// Konfigurasi DI container
const container = new Container();
container.bind("Database").to(Database);
container.bind(UserService).toSelf();

const userService = container.get(UserService);
console.log(userService.getUser(1));
```

- **Kelebihan**:
  - Automasi pengelolaan dependensi untuk aplikasi skala besar.
- **Kekurangan**:
  - Membutuhkan pembelajaran framework/library.

---

### **Perbandingan**

| **Metode**             | **Kelebihan**                                           | **Kekurangan**                                     |
|-------------------------|-------------------------------------------------------|---------------------------------------------------|
| **Tanpa DI**            | Simpel dan mudah dipahami                              | Tight coupling, sulit diuji.                     |
| **Constructor Injection** | Dependensi eksplisit, mudah diuji                    | Constructor bisa menjadi panjang.                |
| **Method Injection**    | Fleksibel untuk dependensi sementara                   | Kurang eksplisit, dependensi tidak persist.       |
| **Property Injection**  | Dependensi mudah diubah                                | Bisa menyebabkan runtime error jika lupa inject. |
| **Interface Injection** | Dependensi mengikuti kontrak tertentu                 | Membutuhkan boilerplate, kurang lazim di JS.     |
| **Context Injection**   | Automasi pengelolaan oleh framework                    | Bergantung pada framework.                       |
| **Service Locator**     | Centralized dependency management                      | Dependensi tidak eksplisit, bisa jadi anti-pattern. |
| **Library**             | Cocok untuk aplikasi skala besar                      | Membutuhkan pembelajaran framework/library.      |

Pilih metode DI berdasarkan kebutuhan proyek dan skala aplikasi Anda.
