# Panduan Lengkap Node.js dengan EJS dan MySQL

## Daftar Isi
1. [Setup Awal](#setup-awal)
2. [Konfigurasi Database](#konfigurasi-database)
3. [Struktur Folder](#struktur-folder)
4. [Model Database](#model-database)
5. [Middleware dan Utilitas](#middleware-dan-utilitas)
6. [Routes](#routes)
7. [Controllers](#controllers)
8. [Views (EJS Templates)](#views-ejs-templates)
9. [File Utama (app.js)](#file-utama-appjs)
10. [Menjalankan Aplikasi](#menjalankan-aplikasi)

## Setup Awal

### 1. Inisialisasi Project
```bash
mkdir belajar-nodejs-ejs
cd belajar-nodejs-ejs
npm init -y
```

### 2. Install Dependencies
```bash
npm install express ejs mysql2 bcryptjs express-session connect-flash dotenv body-parser method-override
npm install --save-dev nodemon
```

### 3. Update package.json
```json
{
  "name": "belajar-nodejs-ejs",
  "version": "1.0.0",
  "description": "Belajar Node.js dengan EJS dan MySQL",
  "main": "app.js",
  "scripts": {
    "start": "node app.js",
    "dev": "nodemon app.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "ejs": "^3.1.9",
    "mysql2": "^3.6.0",
    "bcryptjs": "^2.4.3",
    "express-session": "^1.17.3",
    "connect-flash": "^0.1.1",
    "dotenv": "^16.3.1",
    "body-parser": "^1.20.2",
    "method-override": "^3.0.0"
  },
  "devDependencies": {
    "nodemon": "^3.0.1"
  }
}
```

## Konfigurasi Database

### 1. File .env
```env
# Database Configuration
DB_HOST=localhost
DB_USER=root
DB_PASSWORD=
DB_NAME=belajar_nodejs

# Session Secret
SESSION_SECRET=rahasia_session_kamu

# Server Port
PORT=3000
```

### 2. SQL Database Setup
```sql
-- Buat database
CREATE DATABASE belajar_nodejs;
USE belajar_nodejs;

-- Tabel users untuk autentikasi
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- Tabel products untuk contoh CRUD
CREATE TABLE products (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    price DECIMAL(10,2) NOT NULL,
    stock INT DEFAULT 0,
    user_id INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE SET NULL
);
```

## Struktur Folder

```
belajar-nodejs-ejs/
├── app.js
├── .env
├── package.json
├── config/
│   └── database.js
├── models/
│   ├── index.js
│   ├── User.js
│   └── Product.js
├── controllers/
│   ├── AuthController.js
│   └── ProductController.js
├── routes/
│   ├── auth.js
│   └── products.js
├── middleware/
│   └── auth.js
├── views/
│   ├── layouts/
│   │   └── main.ejs
│   ├── auth/
│   │   ├── login.ejs
│   │   └── register.ejs
│   ├── products/
│   │   ├── index.ejs
│   │   ├── create.ejs
│   │   ├── edit.ejs
│   │   └── show.ejs
│   └── dashboard.ejs
└── public/
    ├── css/
    │   └── style.css
    └── js/
        └── script.js
```

## Model Database

### 1. config/database.js
```javascript
const mysql = require('mysql2');
require('dotenv').config();

const connection = mysql.createConnection({
    host: process.env.DB_HOST,
    user: process.env.DB_USER,
    password: process.env.DB_PASSWORD,
    database: process.env.DB_NAME
});

connection.connect((err) => {
    if (err) {
        console.error('Error connecting to database:', err);
        return;
    }
    console.log('Connected to MySQL database');
});

module.exports = connection;
```

### 2. models/User.js
```javascript
const db = require('../config/database');
const bcrypt = require('bcryptjs');

const User = {
    // Membuat user baru
    create: (userData, callback) => {
        const { username, email, password } = userData;
        
        // Hash password
        bcrypt.hash(password, 10, (err, hashedPassword) => {
            if (err) return callback(err);
            
            const query = 'INSERT INTO users (username, email, password) VALUES (?, ?, ?)';
            db.query(query, [username, email, hashedPassword], callback);
        });
    },

    // Mencari user berdasarkan email
    findByEmail: (email, callback) => {
        const query = 'SELECT * FROM users WHERE email = ?';
        db.query(query, [email], (err, results) => {
            if (err) return callback(err);
            callback(null, results[0]);
        });
    },

    // Mencari user berdasarkan username
    findByUsername: (username, callback) => {
        const query = 'SELECT * FROM users WHERE username = ?';
        db.query(query, [username], (err, results) => {
            if (err) return callback(err);
            callback(null, results[0]);
        });
    },

    // Mencari user berdasarkan ID
    findById: (id, callback) => {
        const query = 'SELECT * FROM users WHERE id = ?';
        db.query(query, [id], (err, results) => {
            if (err) return callback(err);
            callback(null, results[0]);
        });
    },

    // Memverifikasi password
    verifyPassword: (password, hashedPassword, callback) => {
        bcrypt.compare(password, hashedPassword, callback);
    }
};

module.exports = User;
```

### 3. models/Product.js
```javascript
const db = require('../config/database');

const Product = {
    // Mengambil semua produk
    getAll: (callback) => {
        const query = `
            SELECT p.*, u.username 
            FROM products p 
            LEFT JOIN users u ON p.user_id = u.id 
            ORDER BY p.created_at DESC
        `;
        db.query(query, callback);
    },

    // Mengambil produk berdasarkan user_id
    getByUserId: (userId, callback) => {
        const query = `
            SELECT p.*, u.username 
            FROM products p 
            LEFT JOIN users u ON p.user_id = u.id 
            WHERE p.user_id = ?
            ORDER BY p.created_at DESC
        `;
        db.query(query, [userId], callback);
    },

    // Mengambil produk berdasarkan ID
    getById: (id, callback) => {
        const query = `
            SELECT p.*, u.username 
            FROM products p 
            LEFT JOIN users u ON p.user_id = u.id 
            WHERE p.id = ?
        `;
        db.query(query, [id], (err, results) => {
            if (err) return callback(err);
            callback(null, results[0]);
        });
    },

    // Membuat produk baru
    create: (productData, callback) => {
        const { name, description, price, stock, user_id } = productData;
        const query = 'INSERT INTO products (name, description, price, stock, user_id) VALUES (?, ?, ?, ?, ?)';
        db.query(query, [name, description, price, stock, user_id], callback);
    },

    // Update produk
    update: (id, productData, callback) => {
        const { name, description, price, stock } = productData;
        const query = 'UPDATE products SET name = ?, description = ?, price = ?, stock = ? WHERE id = ?';
        db.query(query, [name, description, price, stock, id], callback);
    },

    // Hapus produk
    delete: (id, callback) => {
        const query = 'DELETE FROM products WHERE id = ?';
        db.query(query, [id], callback);
    }
};

module.exports = Product;
```

### 4. models/index.js
```javascript
const User = require('./User');
const Product = require('./Product');

// Setup relasi dan fungsi utilitas
const Models = {
    User,
    Product,

    // Fungsi untuk inisialisasi database
    initializeDatabase: () => {
        console.log('Database models initialized');
        console.log('Available models: User, Product');
    },

    // Fungsi untuk mengecek koneksi database
    checkConnection: (callback) => {
        const db = require('../config/database');
        db.query('SELECT 1', (err, results) => {
            if (err) {
                console.error('Database connection failed:', err);
                return callback(false);
            }
            console.log('Database connection successful');
            callback(true);
        });
    }
};

module.exports = Models;
```

## Middleware dan Utilitas

### 1. middleware/auth.js
```javascript
// Middleware untuk memastikan user sudah login
const requireAuth = (req, res, next) => {
    if (req.session && req.session.userId) {
        return next();
    } else {
        req.flash('error', 'Silakan login terlebih dahulu');
        return res.redirect('/auth/login');
    }
};

// Middleware untuk memastikan user belum login
const requireGuest = (req, res, next) => {
    if (req.session && req.session.userId) {
        return res.redirect('/dashboard');
    } else {
        return next();
    }
};

module.exports = { requireAuth, requireGuest };
```

## Routes

### 1. routes/auth.js
```javascript
const express = require('express');
const router = express.Router();
const AuthController = require('../controllers/AuthController');
const { requireGuest } = require('../middleware/auth');

// Halaman register
router.get('/register', requireGuest, AuthController.showRegister);
router.post('/register', requireGuest, AuthController.register);

// Halaman login
router.get('/login', requireGuest, AuthController.showLogin);
router.post('/login', requireGuest, AuthController.login);

// Logout
router.post('/logout', AuthController.logout);

module.exports = router;
```

### 2. routes/products.js
```javascript
const express = require('express');
const router = express.Router();
const ProductController = require('../controllers/ProductController');
const { requireAuth } = require('../middleware/auth');

// Semua route produk membutuhkan autentikasi
router.use(requireAuth);

// Routes untuk produk
router.get('/', ProductController.index);
router.get('/create', ProductController.showCreate);
router.post('/create', ProductController.create);
router.get('/:id', ProductController.show);
router.get('/:id/edit', ProductController.showEdit);
router.put('/:id', ProductController.update);
router.delete('/:id', ProductController.delete);

module.exports = router;
```

## Controllers

### 1. controllers/AuthController.js
```javascript
const User = require('../models/User');

const AuthController = {
    // Menampilkan halaman register
    showRegister: (req, res) => {
        res.render('auth/register', {
            title: 'Register',
            messages: req.flash()
        });
    },

    // Proses register
    register: (req, res) => {
        const { username, email, password, confirmPassword } = req.body;

        // Validasi input
        if (!username || !email || !password || !confirmPassword) {
            req.flash('error', 'Semua field harus diisi');
            return res.redirect('/auth/register');
        }

        if (password !== confirmPassword) {
            req.flash('error', 'Password dan konfirmasi password tidak sama');
            return res.redirect('/auth/register');
        }

        if (password.length < 6) {
            req.flash('error', 'Password minimal 6 karakter');
            return res.redirect('/auth/register');
        }

        // Cek apakah email sudah terdaftar
        User.findByEmail(email, (err, existingUser) => {
            if (err) {
                req.flash('error', 'Terjadi kesalahan pada server');
                return res.redirect('/auth/register');
            }

            if (existingUser) {
                req.flash('error', 'Email sudah terdaftar');
                return res.redirect('/auth/register');
            }

            // Cek apakah username sudah terdaftar
            User.findByUsername(username, (err, existingUser) => {
                if (err) {
                    req.flash('error', 'Terjadi kesalahan pada server');
                    return res.redirect('/auth/register');
                }

                if (existingUser) {
                    req.flash('error', 'Username sudah terdaftar');
                    return res.redirect('/auth/register');
                }

                // Buat user baru
                User.create({ username, email, password }, (err, result) => {
                    if (err) {
                        req.flash('error', 'Gagal membuat akun');
                        return res.redirect('/auth/register');
                    }

                    req.flash('success', 'Akun berhasil dibuat. Silakan login.');
                    res.redirect('/auth/login');
                });
            });
        });
    },

    // Menampilkan halaman login
    showLogin: (req, res) => {
        res.render('auth/login', {
            title: 'Login',
            messages: req.flash()
        });
    },

    // Proses login
    login: (req, res) => {
        const { email, password } = req.body;

        if (!email || !password) {
            req.flash('error', 'Email dan password harus diisi');
            return res.redirect('/auth/login');
        }

        // Cari user berdasarkan email
        User.findByEmail(email, (err, user) => {
            if (err) {
                req.flash('error', 'Terjadi kesalahan pada server');
                return res.redirect('/auth/login');
            }

            if (!user) {
                req.flash('error', 'Email atau password salah');
                return res.redirect('/auth/login');
            }

            // Verifikasi password
            User.verifyPassword(password, user.password, (err, isMatch) => {
                if (err) {
                    req.flash('error', 'Terjadi kesalahan pada server');
                    return res.redirect('/auth/login');
                }

                if (!isMatch) {
                    req.flash('error', 'Email atau password salah');
                    return res.redirect('/auth/login');
                }

                // Set session
                req.session.userId = user.id;
                req.session.username = user.username;
                req.session.email = user.email;

                req.flash('success', `Selamat datang, ${user.username}!`);
                res.redirect('/dashboard');
            });
        });
    },

    // Logout
    logout: (req, res) => {
        req.session.destroy((err) => {
            if (err) {
                req.flash('error', 'Gagal logout');
                return res.redirect('/dashboard');
            }
            res.redirect('/auth/login');
        });
    }
};

module.exports = AuthController;
```

### 2. controllers/ProductController.js
```javascript
const Product = require('../models/Product');

const ProductController = {
    // Menampilkan daftar produk
    index: (req, res) => {
        Product.getByUserId(req.session.userId, (err, products) => {
            if (err) {
                req.flash('error', 'Gagal mengambil data produk');
                return res.redirect('/dashboard');
            }

            res.render('products/index', {
                title: 'Daftar Produk',
                products: products,
                messages: req.flash(),
                user: req.session
            });
        });
    },

    // Menampilkan form create produk
    showCreate: (req, res) => {
        res.render('products/create', {
            title: 'Tambah Produk',
            messages: req.flash(),
            user: req.session
        });
    },

    // Proses create produk
    create: (req, res) => {
        const { name, description, price, stock } = req.body;

        // Validasi input
        if (!name || !price) {
            req.flash('error', 'Nama produk dan harga harus diisi');
            return res.redirect('/products/create');
        }

        if (isNaN(price) || price <= 0) {
            req.flash('error', 'Harga harus berupa angka positif');
            return res.redirect('/products/create');
        }

        if (stock && (isNaN(stock) || stock < 0)) {
            req.flash('error', 'Stok harus berupa angka non-negatif');
            return res.redirect('/products/create');
        }

        const productData = {
            name,
            description: description || '',
            price: parseFloat(price),
            stock: parseInt(stock) || 0,
            user_id: req.session.userId
        };

        Product.create(productData, (err, result) => {
            if (err) {
                req.flash('error', 'Gagal menambahkan produk');
                return res.redirect('/products/create');
            }

            req.flash('success', 'Produk berhasil ditambahkan');
            res.redirect('/products');
        });
    },

    // Menampilkan detail produk
    show: (req, res) => {
        const productId = req.params.id;

        Product.getById(productId, (err, product) => {
            if (err) {
                req.flash('error', 'Gagal mengambil data produk');
                return res.redirect('/products');
            }

            if (!product) {
                req.flash('error', 'Produk tidak ditemukan');
                return res.redirect('/products');
            }

            // Cek apakah produk milik user yang login
            if (product.user_id !== req.session.userId) {
                req.flash('error', 'Anda tidak memiliki akses ke produk ini');
                return res.redirect('/products');
            }

            res.render('products/show', {
                title: 'Detail Produk',
                product: product,
                messages: req.flash(),
                user: req.session
            });
        });
    },

    // Menampilkan form edit produk
    showEdit: (req, res) => {
        const productId = req.params.id;

        Product.getById(productId, (err, product) => {
            if (err) {
                req.flash('error', 'Gagal mengambil data produk');
                return res.redirect('/products');
            }

            if (!product) {
                req.flash('error', 'Produk tidak ditemukan');
                return res.redirect('/products');
            }

            // Cek apakah produk milik user yang login
            if (product.user_id !== req.session.userId) {
                req.flash('error', 'Anda tidak memiliki akses ke produk ini');
                return res.redirect('/products');
            }

            res.render('products/edit', {
                title: 'Edit Produk',
                product: product,
                messages: req.flash(),
                user: req.session
            });
        });
    },

    // Proses update produk
    update: (req, res) => {
        const productId = req.params.id;
        const { name, description, price, stock } = req.body;

        // Validasi input
        if (!name || !price) {
            req.flash('error', 'Nama produk dan harga harus diisi');
            return res.redirect(`/products/${productId}/edit`);
        }

        if (isNaN(price) || price <= 0) {
            req.flash('error', 'Harga harus berupa angka positif');
            return res.redirect(`/products/${productId}/edit`);
        }

        if (stock && (isNaN(stock) || stock < 0)) {
            req.flash('error', 'Stok harus berupa angka non-negatif');
            return res.redirect(`/products/${productId}/edit`);
        }

        // Cek apakah produk ada dan milik user yang login
        Product.getById(productId, (err, product) => {
            if (err) {
                req.flash('error', 'Gagal mengambil data produk');
                return res.redirect('/products');
            }

            if (!product) {
                req.flash('error', 'Produk tidak ditemukan');
                return res.redirect('/products');
            }

            if (product.user_id !== req.session.userId) {
                req.flash('error', 'Anda tidak memiliki akses ke produk ini');
                return res.redirect('/products');
            }

            const productData = {
                name,
                description: description || '',
                price: parseFloat(price),
                stock: parseInt(stock) || 0
            };

            Product.update(productId, productData, (err, result) => {
                if (err) {
                    req.flash('error', 'Gagal mengupdate produk');
                    return res.redirect(`/products/${productId}/edit`);
                }

                req.flash('success', 'Produk berhasil diupdate');
                res.redirect(`/products/${productId}`);
            });
        });
    },

    // Hapus produk
    delete: (req, res) => {
        const productId = req.params.id;

        // Cek apakah produk ada dan milik user yang login
        Product.getById(productId, (err, product) => {
            if (err) {
                req.flash('error', 'Gagal mengambil data produk');
                return res.redirect('/products');
            }

            if (!product) {
                req.flash('error', 'Produk tidak ditemukan');
                return res.redirect('/products');
            }

            if (product.user_id !== req.session.userId) {
                req.flash('error', 'Anda tidak memiliki akses ke produk ini');
                return res.redirect('/products');
            }

            Product.delete(productId, (err, result) => {
                if (err) {
                    req.flash('error', 'Gagal menghapus produk');
                    return res.redirect('/products');
                }

                req.flash('success', 'Produk berhasil dihapus');
                res.redirect('/products');
            });
        });
    }
};

module.exports = ProductController;
```

## Views (EJS Templates)

### 1. views/layouts/main.ejs
```html
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title><%= title %> - Belajar Node.js</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="/css/style.css">
</head>
<body>
    <!-- Navigation -->
    <nav class="navbar navbar-expand-lg navbar-dark bg-primary">
        <div class="container">
            <a class="navbar-brand" href="/">Belajar Node.js</a>
            
            <% if (typeof user !== 'undefined' && user.userId) { %>
                <div class="navbar-nav ms-auto">
                    <a class="nav-link" href="/dashboard">Dashboard</a>
                    <a class="nav-link" href="/products">Produk</a>
                    <div class="nav-item dropdown">
                        <a class="nav-link dropdown-toggle" href="#" id="navbarDropdown" role="button" data-bs-toggle="dropdown">
                            <%= user.username %>
                        </a>
                        <ul class="dropdown-menu">
                            <li>
                                <form action="/auth/logout" method="POST" class="d-inline">
                                    <button type="submit" class="dropdown-item">Logout</button>
                                </form>
                            </li>
                        </ul>
                    </div>
                </div>
            <% } else { %>
                <div class="navbar-nav ms-auto">
                    <a class="nav-link" href="/auth/login">Login</a>
                    <a class="nav-link" href="/auth/register">Register</a>
                </div>
            <% } %>
        </div>
    </nav>

    <!-- Flash Messages -->
    <% if (typeof messages !== 'undefined') { %>
        <% if (messages.success && messages.success.length > 0) { %>
            <div class="container mt-3">
                <div class="alert alert-success alert-dismissible fade show" role="alert">
                    <%= messages.success[0] %>
                    <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
                </div>
            </div>
        <% } %>
        
        <% if (messages.error && messages.error.length > 0) { %>
            <div class="container mt-3">
                <div class="alert alert-danger alert-dismissible fade show" role="alert">
                    <%= messages.error[0] %>
                    <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
                </div>
            </div>
        <% } %>
    <% } %>

    <!-- Main Content -->
    <main class="container mt-4">
        <%- body %>
    </main>

    <!-- Footer -->
    <footer class="bg-light mt-5 py-4">
        <div class="container text-center">
            <p>&copy; 2024 Belajar Node.js dengan EJS dan MySQL</p>
        </div>
    </footer>

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/js/bootstrap.bundle.min.js"></script>
    <script src="/js/script.js"></script>
</body>
</html>
```

### 2. views/auth/register.ejs
```html
<% layout('layouts/main') -%>

<div class="row justify-content-center">
    <div class="col-md-6">
        <div class="card">
            <div class="card-header">
                <h4 class="mb-0">Register</h4>
            </div>
            <div class="card-body">
                <form action="/auth/register" method="POST">
                    <div class="mb-3">
                        <label for="username" class="form-label">Username</label>
                        <input type="text" class="form-control" id="username" name="username" required>
                    </div>
                    
                    <div class="mb-3">
                        <label for="email" class="form-label">Email</label>
                        <input type="email" class="form-control" id="email" name="email" required>
                    </div>
                    
                    <div class="mb-3">
                        <label for="password" class="form-label">Password</label>
                        <input type="password" class="form-control" id="password" name="password" required>
                        <div class="form-text">Password minimal 6 karakter</div>
                    </div>
                    
                    <div class="mb-3">
                        <label for="confirmPassword" class="form-label">Konfirmasi Password</label>
                        <input type="password" class="form-control" id="confirmPassword" name="confirmPassword" required>
                    </div>
                    
                    <button type="submit" class="btn btn-primary w-100">Register</button>
                </form>
                
                <div class="text-center mt-3">
                    <p>Sudah punya akun? <a href="/auth/login">Login di sini</a></p>
                </div>
            </div>
        </div>
    </div>
</div>
```

### 3. views/auth/login.ejs
```html
<% layout('layouts/main') -%>

<div class="row justify-content-center">
    <div class="col-md-6">
        <div class="card">
            <div class="card-header">
                <h4 class="mb-0">Login</h4>
            </div>
            <div class="card-body">
                <form action="/auth/login" method="POST">
                    <div class="mb-3">
                        <label for="email" class="form-label">Email</label>
                        <input type="email" class="form-control" id="email" name="email" required>
                    </div>
                    
                    <div class="mb-3">
                        <label for="password" class="form-label">Password</label>
                        <input type="password" class="form-control" id="password" name="password" required>
                    </div>
                    
                    <button type="submit" class="btn btn-primary w-100">Login</button>
                </form>
                
                <div class="text-center mt-3">
                    <p>Belum punya akun? <a href="/auth/register">Register di sini</a></p>
                </div>
            </div>
        </div>
    </div>
</div>
```

### 4. views/dashboard.ejs
```html
<% layout('layouts/main') -%>

<div class="row">
    <div class="col-12">
        <h1>Dashboard</h1>
        <p class="lead">Selamat datang, <%= user.username %>!</p>
    </div>
</div>

<div class="row mt-4">
    <div class="col-md-4">
        <div class="card bg-primary text-white">
            <div class="card-body">
                <h5 class="card-title">Produk</h5>
                <p class="card-text">Kelola produk Anda</p>
                <a href="/products" class="btn btn-light">Lihat Produk</a>
            </div>
        </div>
    </div>
    
    <div class="col-md-4">
        <div class="card bg-success text-white">
            <div class="card-body">
                <h5 class="card-title">Tambah Produk</h5>
                <p class="card-text">Tambahkan produk baru</p>
                <a href="/products/create" class="btn btn-light">Tambah Produk</a>
            </div>
        </div>
    </div>
    
    <div class="col-md-4">
        <div class="card bg-info text-white">
            <div class="card-body">
                <h5 class="card-title">Profile</h5>
                <p class="card-text">Email: <%= user.email %></p>
                <p class="card-text">Username: <%= user.username %></p>
            </div>
        </div>
    </div>
</div>
```

### 5. views/products/index.ejs
```html
<% layout('layouts/main') -%>

<div class="d-flex justify-content-between align-items-center mb-4">
    <h1>Daftar Produk</h1>
    <a href="/products/create" class="btn btn-primary">Tambah Produk</a>
</div>

<% if (products && products.length > 0) { %>
    <div class="row">
        <% products.forEach(product => { %>
            <div class="col-md-4 mb-4">
                <div class="card">
                    <div class="card-body">
                        <h5 class="card-title"><%= product.name %></h5>
                        <p class="card-text"><%= product.description || 'Tidak ada deskripsi' %></p>
                        <p class="card-text">
                            <strong>Harga:</strong> Rp <%= product.price.toLocaleString('id-ID') %><br>
                            <strong>Stok:</strong> <%= product.stock %>
                        </p>
                        <div class="btn-group" role="group">
                            <a href="/products/<%= product.id %>" class="btn btn-outline-primary btn-sm">Detail</a>
                            <a href="/products/<%= product.id %>/edit" class="btn btn-outline-warning btn-sm">Edit</a>
                            <form action="/products/<%= product.id %>?_method=DELETE" method="POST" class="d-inline">
                                <button type="submit" class="btn btn-outline-danger btn-sm" 
                                        onclick="return confirm('Yakin ingin menghapus produk ini?')">Hapus</button>
                            </form>
                        </div>
                    </div>
                </div>
            </div>
        <% }) %>
    </div>
<% } else { %>
    <div class="text-center py-5">
        <h3>Belum ada produk</h3>
        <p>Mulai dengan menambahkan produk pertama Anda</p>
        <a href="/products/create" class="btn btn-primary">Tambah Produk</a>
    </div>
<% } %>
```

### 6. views/products/create.ejs
```html
<% layout('layouts/main') -%>

<div class="row">
    <div class="col-md-8">
        <h1>Tambah Produk</h1>
        
        <form action="/products/create" method="POST">
            <div class="mb-3">
                <label for="name" class="form-label">Nama Produk *</label>
                <input type="text" class="form-control" id="name" name="name" required>
            </div>
            
            <div class="mb-3">
                <label for="description" class="form-label">Deskripsi</label>
                <textarea class="form-control" id="description" name="description" rows="3"></textarea>
            </div>
            
            <div class="row">
                <div class="col-md-6">
                    <div class="mb-3">
                        <label for="price" class="form-label">Harga *</label>
                        <input type="number" class="form-control" id="price" name="price" step="0.01" min="0" required>
                    </div>
                </div>
                
                <div class="col-md-6">
                    <div class="mb-3">
                        <label for="stock" class="form-label">Stok</label>
                        <input type="number" class="form-control" id="stock" name="stock" min="0" value="0">
                    </div>
                </div>
            </div>
            
            <div class="mb-3">
                <button type="submit" class="btn btn-primary">Simpan</button>
                <a href="/products" class="btn btn-secondary">Batal</a>
            </div>
        </form>
    </div>
</div>
```

### 7. views/products/edit.ejs
```html
<% layout('layouts/main') -%>

<div class="row">
    <div class="col-md-8">
        <h1>Edit Produk</h1>
        
        <form action="/products/<%= product.id %>?_method=PUT" method="POST">
            <div class="mb-3">
                <label for="name" class="form-label">Nama Produk *</label>
                <input type="text" class="form-control" id="name" name="name" value="<%= product.name %>" required>
            </div>
            
            <div class="mb-3">
                <label for="description" class="form-label">Deskripsi</label>
                <textarea class="form-control" id="description" name="description" rows="3"><%= product.description || '' %></textarea>
            </div>
            
            <div class="row">
                <div class="col-md-6">
                    <div class="mb-3">
                        <label for="price" class="form-label">Harga *</label>
                        <input type="number" class="form-control" id="price" name="price" step="0.01" min="0" value="<%= product.price %>" required>
                    </div>
                </div>
                
                <div class="col-md-6">
                    <div class="mb-3">
                        <label for="stock" class="form-label">Stok</label>
                        <input type="number" class="form-control" id="stock" name="stock" min="0" value="<%= product.stock %>">
                    </div>
                </div>
            </div>
            
            <div class="mb-3">
                <button type="submit" class="btn btn-primary">Update</button>
                <a href="/products/<%= product.id %>" class="btn btn-secondary">Batal</a>
            </div>
        </form>
    </div>
</div>
```

### 8. views/products/show.ejs
```html
<% layout('layouts/main') -%>

<div class="row">
    <div class="col-md-8">
        <div class="d-flex justify-content-between align-items-start mb-4">
            <h1><%= product.name %></h1>
            <div class="btn-group">
                <a href="/products/<%= product.id %>/edit" class="btn btn-warning">Edit</a>
                <form action="/products/<%= product.id %>?_method=DELETE" method="POST" class="d-inline">
                    <button type="submit" class="btn btn-danger" 
                            onclick="return confirm('Yakin ingin menghapus produk ini?')">Hapus</button>
                </form>
            </div>
        </div>
        
        <div class="card">
            <div class="card-body">
                <h5 class="card-title">Detail Produk</h5>
                
                <div class="row">
                    <div class="col-md-6">
                        <p><strong>Nama:</strong> <%= product.name %></p>
                        <p><strong>Harga:</strong> Rp <%= product.price.toLocaleString('id-ID') %></p>
                        <p><strong>Stok:</strong> <%= product.stock %></p>
                    </div>
                    
                    <div class="col-md-6">
                        <p><strong>Dibuat:</strong> <%= new Date(product.created_at).toLocaleDateString('id-ID') %></p>
                        <p><strong>Diupdate:</strong> <%= new Date(product.updated_at).toLocaleDateString('id-ID') %></p>
                    </div>
                </div>
                
                <% if (product.description) { %>
                    <div class="mt-3">
                        <strong>Deskripsi:</strong>
                        <p><%= product.description %></p>
                    </div>
                <% } %>
            </div>
        </div>
        
        <div class="mt-3">
            <a href="/products" class="btn btn-secondary">Kembali ke Daftar Produk</a>
        </div>
    </div>
</div>
```

## File Utama (app.js)

```javascript
require('dotenv').config();
const express = require('express');
const session = require('express-session');
const flash = require('connect-flash');
const path = require('path');
const methodOverride = require('method-override');
const expressLayouts = require('express-ejs-layouts');

// Import routes
const authRoutes = require('./routes/auth');
const productRoutes = require('./routes/products');

// Import middleware
const { requireAuth } = require('./middleware/auth');

// Import models untuk inisialisasi
const Models = require('./models');

const app = express();
const PORT = process.env.PORT || 3000;

// Inisialisasi database models
Models.initializeDatabase();

// View engine setup
app.set('view engine', 'ejs');
app.set('views', path.join(__dirname, 'views'));
app.use(expressLayouts);
app.set('layout', 'layouts/main');

// Middleware
app.use(express.static(path.join(__dirname, 'public')));
app.use(express.urlencoded({ extended: true }));
app.use(express.json());
app.use(methodOverride('_method'));

// Session configuration
app.use(session({
    secret: process.env.SESSION_SECRET,
    resave: false,
    saveUninitialized: true,
    cookie: { 
        secure: false, // Set true jika menggunakan HTTPS
        maxAge: 24 * 60 * 60 * 1000 // 24 jam
    }
}));

app.use(flash());

// Global variables untuk views
app.use((req, res, next) => {
    res.locals.messages = req.flash();
    res.locals.user = req.session;
    next();
});

// Routes
app.get('/', (req, res) => {
    if (req.session.userId) {
        res.redirect('/dashboard');
    } else {
        res.redirect('/auth/login');
    }
});

app.get('/dashboard', requireAuth, (req, res) => {
    res.render('dashboard', {
        title: 'Dashboard',
        user: req.session
    });
});

// Auth routes
app.use('/auth', authRoutes);

// Product routes
app.use('/products', productRoutes);

// 404 handler
app.use((req, res) => {
    res.status(404).render('404', {
        title: 'Halaman Tidak Ditemukan',
        user: req.session
    });
});

// Error handler
app.use((err, req, res, next) => {
    console.error(err.stack);
    res.status(500).render('error', {
        title: 'Terjadi Kesalahan',
        error: err,
        user: req.session
    });
});

// Start server
app.listen(PORT, () => {
    console.log(`Server berjalan di http://localhost:${PORT}`);
    
    // Test database connection
    Models.checkConnection((isConnected) => {
        if (isConnected) {
            console.log('✅ Database connection successful');
        } else {
            console.log('❌ Database connection failed');
        }
    });
});
```

## File CSS dan JS

### public/css/style.css
```css
body {
    background-color: #f8f9fa;
}

.card {
    box-shadow: 0 0.125rem 0.25rem rgba(0, 0, 0, 0.075);
    border: 1px solid rgba(0, 0, 0, 0.125);
}

.navbar-brand {
    font-weight: bold;
}

.alert {
    border-radius: 0.375rem;
}

.btn-group .btn {
    margin-right: 0;
}

.card-title {
    color: #495057;
    font-weight: 600;
}

footer {
    margin-top: auto;
}

/* Form styling */
.form-label {
    font-weight: 600;
    color: #495057;
}

.form-control:focus {
    border-color: #80bdff;
    box-shadow: 0 0 0 0.2rem rgba(0, 123, 255, 0.25);
}

/* Custom button styles */
.btn-outline-primary:hover,
.btn-outline-warning:hover,
.btn-outline-danger:hover {
    transform: translateY(-1px);
    transition: transform 0.2s;
}

/* Card hover effect */
.card:hover {
    transform: translateY(-2px);
    transition: transform 0.3s;
    box-shadow: 0 0.5rem 1rem rgba(0, 0, 0, 0.15);
}
```

### public/js/script.js
```javascript
// Auto hide alerts after 5 seconds
document.addEventListener('DOMContentLoaded', function() {
    const alerts = document.querySelectorAll('.alert');
    alerts.forEach(alert => {
        setTimeout(() => {
            const bsAlert = new bootstrap.Alert(alert);
            bsAlert.close();
        }, 5000);
    });
});

// Confirm delete
function confirmDelete(message = 'Yakin ingin menghapus item ini?') {
    return confirm(message);
}

// Form validation
document.addEventListener('DOMContentLoaded', function() {
    const forms = document.querySelectorAll('form');
    
    forms.forEach(form => {
        form.addEventListener('submit', function(event) {
            if (!form.checkValidity()) {
                event.preventDefault();
                event.stopPropagation();
            }
            form.classList.add('was-validated');
        });
    });
});

// Price formatter
function formatPrice(input) {
    let value = input.value.replace(/[^\d.]/g, '');
    input.value = value;
}

// Auto-focus on first input
document.addEventListener('DOMContentLoaded', function() {
    const firstInput = document.querySelector('input:not([type="hidden"]):not([readonly])');
    if (firstInput) {
        firstInput.focus();
    }
});
```

## Menjalankan Aplikasi

### 1. Setup Database
```bash
# Masuk ke MySQL
mysql -u root -p

# Jalankan script SQL yang sudah dibuat di atas
source path/to/your/database.sql
```

### 2. Konfigurasi Environment
```bash
# Copy file .env dan sesuaikan dengan konfigurasi database Anda
cp .env.example .env
```

### 3. Install Dependencies
```bash
npm install
```

### 4. Jalankan Aplikasi
```bash
# Development mode
npm run dev

# Production mode
npm start
```

### 5. Akses Aplikasi
Buka browser dan akses: `http://localhost:3000`

## Fitur yang Tersedia

1. **Autentikasi**
   - Register pengguna baru
   - Login dengan email dan password
   - Logout
   - Session management

2. **CRUD Produk**
   - Tambah produk baru
   - Lihat daftar produk
   - Edit produk
   - Hapus produk
   - Detail produk

3. **Keamanan**
   - Password hashing dengan bcrypt
   - Session-based authentication
   - Input validation
   - SQL injection protection

4. **UI/UX**
   - Responsive design dengan Bootstrap
   - Flash messages untuk feedback
   - Form validation
   - Konfirmasi penghapusan

## Tips Pengembangan

1. **Debugging**: Gunakan `console.log()` untuk debugging
2. **Error Handling**: Selalu sertakan error handling di setiap fungsi callback
3. **Validation**: Validasi input di sisi server dan client
4. **Security**: Jangan pernah menyimpan password dalam bentuk plain text
5. **Performance**: Gunakan connection pooling untuk database di production

Selamat belajar! 🚀