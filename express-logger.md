
# Membuat Logger di Express.js

Logger di aplikasi **Express.js** digunakan untuk mencatat informasi seperti permintaan HTTP, error, atau aktivitas lainnya untuk kebutuhan debugging, monitoring, dan analisis.

---

## **1. Menggunakan Middleware Logger Sederhana**

Express mendukung middleware, sehingga Anda bisa membuat logger sederhana untuk mencatat setiap permintaan yang masuk.

### **Contoh: Logger Middleware**
```javascript
const express = require('express');
const app = express();

// Middleware untuk mencatat permintaan
app.use((req, res, next) => {
  const now = new Date().toISOString();
  console.log(`[${now}] ${req.method} ${req.url}`);
  next(); // Lanjutkan ke middleware berikutnya
});

app.get('/', (req, res) => {
  res.send('Hello, world!');
});

const PORT = 3000;
app.listen(PORT, () => {
  console.log(`Server berjalan di port ${PORT}`);
});
```

---

## **2. Menggunakan Library `morgan`**

[`morgan`](https://www.npmjs.com/package/morgan) adalah library populer untuk logging HTTP di aplikasi Express.js.

### **Langkah-Langkah:**
1. Instal `morgan`:
   ```bash
   npm install morgan
   ```

2. Tambahkan `morgan` sebagai middleware:
   ```javascript
   const express = require('express');
   const morgan = require('morgan');
   const app = express();

   // Gunakan morgan untuk mencatat log
   app.use(morgan('combined')); // Format log: 'combined'

   app.get('/', (req, res) => {
     res.send('Hello, world!');
   });

   const PORT = 3000;
   app.listen(PORT, () => {
     console.log(`Server berjalan di port ${PORT}`);
   });
   ```

### **Format `morgan`**
- **`combined`**: Format standar dengan detail lengkap (IP, metode, status, dll.).
- **`dev`**: Format ringkas untuk pengembangan.
- **`common`**: Format sederhana untuk log umum.

---

## **3. Membuat Logger Kustom dengan `winston`**

[`winston`](https://www.npmjs.com/package/winston) adalah library logging yang fleksibel, mendukung penyimpanan log ke file, konsol, atau layanan eksternal.

### **Langkah-Langkah:**
1. Instal `winston`:
   ```bash
   npm install winston
   ```

2. Buat logger dengan `winston`:
   ```javascript
   const express = require('express');
   const winston = require('winston');

   const app = express();

   // Konfigurasi winston
   const logger = winston.createLogger({
     level: 'info', // Level log: 'info', 'error', 'warn', dll.
     format: winston.format.combine(
       winston.format.timestamp(),
       winston.format.printf(({ timestamp, level, message }) => {
         return `[${timestamp}] ${level.toUpperCase()}: ${message}`;
       })
     ),
     transports: [
       new winston.transports.Console(), // Log ke konsol
       new winston.transports.File({ filename: 'logs/app.log' }), // Log ke file
     ],
   });

   // Middleware untuk mencatat permintaan
   app.use((req, res, next) => {
     logger.info(`${req.method} ${req.url}`);
     next();
   });

   app.get('/', (req, res) => {
     res.send('Hello, world!');
   });

   const PORT = 3000;
   app.listen(PORT, () => {
     logger.info(`Server berjalan di port ${PORT}`);
   });
   ```

### **Hasil:**
- **Log di Konsol**:
  ```
  [2024-11-22T12:34:56.789Z] INFO: GET /
  [2024-11-22T12:34:56.789Z] INFO: Server berjalan di port 3000
  ```

- **Log di File (`logs/app.log`)**:
  ```
  [2024-11-22T12:34:56.789Z] INFO: GET /
  [2024-11-22T12:34:56.789Z] INFO: Server berjalan di port 3000
  ```

---

## **4. Logger untuk Error**

Tambahkan logger khusus untuk mencatat error di aplikasi.

### **Contoh: Logging Error**
```javascript
// Middleware untuk menangani error
app.use((err, req, res, next) => {
  logger.error(`${err.message} - ${req.method} ${req.url}`);
  res.status(500).send('Internal Server Error');
});
```

### **Penggunaan:**
Jika ada error, log akan mencatat detailnya:
```
[2024-11-22T12:34:56.789Z] ERROR: Something went wrong - GET /example
```

---

## **5. Menggabungkan Morgan dan Winston**

Anda dapat menggabungkan `morgan` untuk mencatat HTTP request dan `winston` untuk mencatat error dan aktivitas lainnya.

### **Contoh Implementasi Gabungan**
```javascript
const express = require('express');
const morgan = require('morgan');
const winston = require('winston');

const app = express();

// Konfigurasi winston
const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.printf(({ timestamp, level, message }) => {
      return `[${timestamp}] ${level.toUpperCase()}: ${message}`;
    })
  ),
  transports: [
    new winston.transports.Console(),
    new winston.transports.File({ filename: 'logs/app.log' }),
  ],
});

// Gunakan morgan dengan winston
app.use(
  morgan('combined', {
    stream: {
      write: (message) => logger.info(message.trim()), // Kirim log morgan ke winston
    },
  })
);

app.get('/', (req, res) => {
  res.send('Hello, world!');
});

// Middleware untuk menangani error
app.use((err, req, res, next) => {
  logger.error(`${err.message} - ${req.method} ${req.url}`);
  res.status(500).send('Internal Server Error');
});

const PORT = 3000;
app.listen(PORT, () => {
  logger.info(`Server berjalan di port ${PORT}`);
});
```

---

## **Kesimpulan**

- **Logger Middleware Sederhana**: Cocok untuk aplikasi kecil tanpa banyak dependensi.
- **Morgan**: Pilihan ringan untuk mencatat permintaan HTTP.
- **Winston**: Solusi fleksibel untuk logging ke berbagai tujuan (file, konsol, layanan eksternal).
- **Gabungan Morgan dan Winston**: Kombinasi terbaik untuk mencatat permintaan HTTP dan log lainnya dengan fleksibilitas tinggi.

Pilih metode logging yang sesuai dengan kebutuhan proyek Anda!
