# Panduan Lengkap Node.js + EJS + MySQL

## Daftar Isi
1. [Persiapan Project](#persiapan-project)
2. [Struktur Folder](#struktur-folder)
3. [Konfigurasi Database](#konfigurasi-database)
4. [Setup Express dan Dependencies](#setup-express-dan-dependencies)
5. [Model Database](#model-database)
6. [Controller](#controller)
7. [Routes](#routes)
8. [Views (EJS Templates)](#views-ejs-templates)
9. [App.js (Main File)](#appjs-main-file)
10. [Menjalankan Aplikasi](#menjalankan-aplikasi)

## Persiapan Project

### 1. Buat Folder Project
```bash
mkdir nodejs-ejs-crud
cd nodejs-ejs-crud
```

### 2. Inisialisasi NPM
```bash
npm init -y
```

### 3. Install Dependencies
```bash
npm install express ejs mysql2 bcryptjs express-session connect-mysql body-parser dotenv
npm install --save-dev nodemon
```

## Struktur Folder

Buat struktur folder seperti berikut:
```
nodejs-ejs-crud/
├── config/
│   └── database.js
├── controllers/
│   ├── authController.js
│   └── bookController.js
├── models/
│   ├── User.js
│   └── Book.js
├── routes/
│   ├── auth.js
│   └── books.js
├── views/
│   ├── partials/
│   │   ├── header.ejs
│   │   └── footer.ejs
│   ├── auth/
│   │   ├── login.ejs
│   │   └── register.ejs
│   ├── books/
│   │   ├── index.ejs
│   │   ├── create.ejs
│   │   ├── edit.ejs
│   │   └── show.ejs
│   └── index.ejs
├── public/
│   ├── css/
│   │   └── style.css
│   └── js/
├── .env
├── app.js
└── package.json
```

## Konfigurasi Database

### 1. File .env
```env
DB_HOST=localhost
DB_USER=root
DB_PASSWORD=your_password
DB_NAME=nodejs_crud_app
PORT=3000
SESSION_SECRET=your_secret_key_here
```

### 2. config/database.js
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
        console.error('Error connecting to database: ' + err.stack);
        return;
    }
    console.log('Connected to database as ID ' + connection.threadId);
});

module.exports = connection;
```

### 3. Buat Database dan Tabel

Jalankan query SQL berikut di MySQL:

```sql
CREATE DATABASE nodejs_crud_app;
USE nodejs_crud_app;

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE books (
    id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    author VARCHAR(100) NOT NULL,
    publisher VARCHAR(100),
    year_published INT,
    isbn VARCHAR(20),
    description TEXT,
    user_id INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

## Setup Express dan Dependencies

### 1. package.json (update scripts)
```json
{
  "name": "nodejs-ejs-crud",
  "version": "1.0.0",
  "description": "",
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
    "connect-mysql": "^3.0.0",
    "body-parser": "^1.20.2",
    "dotenv": "^16.3.1"
  },
  "devDependencies": {
    "nodemon": "^3.0.1"
  }
}
```

## Model Database

### 1. models/User.js
```javascript
const db = require('../config/database');
const bcrypt = require('bcryptjs');

class User {
    constructor(username, email, password) {
        this.username = username;
        this.email = email;
        this.password = password;
    }

    async save() {
        try {
            const hashedPassword = await bcrypt.hash(this.password, 10);
            const query = 'INSERT INTO users (username, email, password) VALUES (?, ?, ?)';
            return new Promise((resolve, reject) => {
                db.query(query, [this.username, this.email, hashedPassword], (err, result) => {
                    if (err) reject(err);
                    else resolve(result);
                });
            });
        } catch (error) {
            throw error;
        }
    }

    static findByEmail(email) {
        const query = 'SELECT * FROM users WHERE email = ?';
        return new Promise((resolve, reject) => {
            db.query(query, [email], (err, results) => {
                if (err) reject(err);
                else resolve(results[0]);
            });
        });
    }

    static findById(id) {
        const query = 'SELECT * FROM users WHERE id = ?';
        return new Promise((resolve, reject) => {
            db.query(query, [id], (err, results) => {
                if (err) reject(err);
                else resolve(results[0]);
            });
        });
    }

    static async validatePassword(inputPassword, hashedPassword) {
        return await bcrypt.compare(inputPassword, hashedPassword);
    }
}

module.exports = User;
```

### 2. models/Book.js
```javascript
const db = require('../config/database');

class Book {
    constructor(title, author, publisher, year_published, isbn, description, user_id) {
        this.title = title;
        this.author = author;
        this.publisher = publisher;
        this.year_published = year_published;
        this.isbn = isbn;
        this.description = description;
        this.user_id = user_id;
    }

    save() {
        const query = `INSERT INTO books (title, author, publisher, year_published, isbn, description, user_id) 
                       VALUES (?, ?, ?, ?, ?, ?, ?)`;
        return new Promise((resolve, reject) => {
            db.query(query, [
                this.title, 
                this.author, 
                this.publisher, 
                this.year_published, 
                this.isbn, 
                this.description, 
                this.user_id
            ], (err, result) => {
                if (err) reject(err);
                else resolve(result);
            });
        });
    }

    static findAll() {
        const query = `SELECT b.*, u.username 
                       FROM books b 
                       LEFT JOIN users u ON b.user_id = u.id 
                       ORDER BY b.created_at DESC`;
        return new Promise((resolve, reject) => {
            db.query(query, (err, results) => {
                if (err) reject(err);
                else resolve(results);
            });
        });
    }

    static findById(id) {
        const query = `SELECT b.*, u.username 
                       FROM books b 
                       LEFT JOIN users u ON b.user_id = u.id 
                       WHERE b.id = ?`;
        return new Promise((resolve, reject) => {
            db.query(query, [id], (err, results) => {
                if (err) reject(err);
                else resolve(results[0]);
            });
        });
    }

    static update(id, bookData) {
        const query = `UPDATE books 
                       SET title = ?, author = ?, publisher = ?, year_published = ?, 
                           isbn = ?, description = ? 
                       WHERE id = ?`;
        return new Promise((resolve, reject) => {
            db.query(query, [
                bookData.title,
                bookData.author,
                bookData.publisher,
                bookData.year_published,
                bookData.isbn,
                bookData.description,
                id
            ], (err, result) => {
                if (err) reject(err);
                else resolve(result);
            });
        });
    }

    static delete(id) {
        const query = 'DELETE FROM books WHERE id = ?';
        return new Promise((resolve, reject) => {
            db.query(query, [id], (err, result) => {
                if (err) reject(err);
                else resolve(result);
            });
        });
    }

    static findByUserId(userId) {
        const query = `SELECT * FROM books WHERE user_id = ? ORDER BY created_at DESC`;
        return new Promise((resolve, reject) => {
            db.query(query, [userId], (err, results) => {
                if (err) reject(err);
                else resolve(results);
            });
        });
    }
}

module.exports = Book;
```

## Controller

### 1. controllers/authController.js
```javascript
const User = require('../models/User');

const authController = {
    // Render halaman register
    showRegister: (req, res) => {
        res.render('auth/register', { error: null, success: null });
    },

    // Proses register
    register: async (req, res) => {
        try {
            const { username, email, password, confirmPassword } = req.body;

            // Validasi
            if (!username || !email || !password || !confirmPassword) {
                return res.render('auth/register', { 
                    error: 'Semua field harus diisi', 
                    success: null 
                });
            }

            if (password !== confirmPassword) {
                return res.render('auth/register', { 
                    error: 'Password tidak cocok', 
                    success: null 
                });
            }

            // Cek apakah email sudah ada
            const existingUser = await User.findByEmail(email);
            if (existingUser) {
                return res.render('auth/register', { 
                    error: 'Email sudah terdaftar', 
                    success: null 
                });
            }

            // Buat user baru
            const newUser = new User(username, email, password);
            await newUser.save();

            res.render('auth/register', { 
                error: null, 
                success: 'Registrasi berhasil! Silakan login.' 
            });

        } catch (error) {
            console.error(error);
            res.render('auth/register', { 
                error: 'Terjadi kesalahan server', 
                success: null 
            });
        }
    },

    // Render halaman login
    showLogin: (req, res) => {
        res.render('auth/login', { error: null });
    },

    // Proses login
    login: async (req, res) => {
        try {
            const { email, password } = req.body;

            if (!email || !password) {
                return res.render('auth/login', { 
                    error: 'Email dan password harus diisi' 
                });
            }

            // Cari user berdasarkan email
            const user = await User.findByEmail(email);
            if (!user) {
                return res.render('auth/login', { 
                    error: 'Email atau password salah' 
                });
            }

            // Validasi password
            const isValidPassword = await User.validatePassword(password, user.password);
            if (!isValidPassword) {
                return res.render('auth/login', { 
                    error: 'Email atau password salah' 
                });
            }

            // Set session
            req.session.userId = user.id;
            req.session.username = user.username;

            res.redirect('/books');

        } catch (error) {
            console.error(error);
            res.render('auth/login', { 
                error: 'Terjadi kesalahan server' 
            });
        }
    },

    // Logout
    logout: (req, res) => {
        req.session.destroy((err) => {
            if (err) {
                console.error(err);
            }
            res.redirect('/');
        });
    }
};

module.exports = authController;
```

### 2. controllers/bookController.js
```javascript
const Book = require('../models/Book');

const bookController = {
    // Tampilkan semua buku
    index: async (req, res) => {
        try {
            const books = await Book.findAll();
            res.render('books/index', { 
                books, 
                user: req.session 
            });
        } catch (error) {
            console.error(error);
            res.status(500).send('Server Error');
        }
    },

    // Tampilkan form create
    showCreate: (req, res) => {
        res.render('books/create', { 
            error: null, 
            user: req.session 
        });
    },

    // Proses create
    create: async (req, res) => {
        try {
            const { title, author, publisher, year_published, isbn, description } = req.body;

            if (!title || !author) {
                return res.render('books/create', { 
                    error: 'Judul dan penulis harus diisi',
                    user: req.session 
                });
            }

            const newBook = new Book(
                title, 
                author, 
                publisher, 
                year_published, 
                isbn, 
                description, 
                req.session.userId
            );

            await newBook.save();
            res.redirect('/books');

        } catch (error) {
            console.error(error);
            res.render('books/create', { 
                error: 'Terjadi kesalahan server',
                user: req.session 
            });
        }
    },

    // Tampilkan detail buku
    show: async (req, res) => {
        try {
            const book = await Book.findById(req.params.id);
            if (!book) {
                return res.status(404).send('Buku tidak ditemukan');
            }
            res.render('books/show', { 
                book, 
                user: req.session 
            });
        } catch (error) {
            console.error(error);
            res.status(500).send('Server Error');
        }
    },

    // Tampilkan form edit
    showEdit: async (req, res) => {
        try {
            const book = await Book.findById(req.params.id);
            if (!book) {
                return res.status(404).send('Buku tidak ditemukan');
            }
            res.render('books/edit', { 
                book, 
                error: null, 
                user: req.session 
            });
        } catch (error) {
            console.error(error);
            res.status(500).send('Server Error');
        }
    },

    // Proses update
    update: async (req, res) => {
        try {
            const { title, author, publisher, year_published, isbn, description } = req.body;

            if (!title || !author) {
                const book = await Book.findById(req.params.id);
                return res.render('books/edit', { 
                    book,
                    error: 'Judul dan penulis harus diisi',
                    user: req.session 
                });
            }

            await Book.update(req.params.id, {
                title, 
                author, 
                publisher, 
                year_published, 
                isbn, 
                description
            });

            res.redirect('/books');

        } catch (error) {
            console.error(error);
            const book = await Book.findById(req.params.id);
            res.render('books/edit', { 
                book,
                error: 'Terjadi kesalahan server',
                user: req.session 
            });
        }
    },

    // Proses delete
    delete: async (req, res) => {
        try {
            await Book.delete(req.params.id);
            res.redirect('/books');
        } catch (error) {
            console.error(error);
            res.status(500).send('Server Error');
        }
    }
};

module.exports = bookController;
```

## Routes

### 1. routes/auth.js
```javascript
const express = require('express');
const router = express.Router();
const authController = require('../controllers/authController');

// Route register
router.get('/register', authController.showRegister);
router.post('/register', authController.register);

// Route login
router.get('/login', authController.showLogin);
router.post('/login', authController.login);

// Route logout
router.get('/logout', authController.logout);

module.exports = router;
```

### 2. routes/books.js
```javascript
const express = require('express');
const router = express.Router();
const bookController = require('../controllers/bookController');

// Middleware untuk cek login
const requireAuth = (req, res, next) => {
    if (!req.session.userId) {
        return res.redirect('/auth/login');
    }
    next();
};

// Routes CRUD Books
router.get('/', requireAuth, bookController.index);
router.get('/create', requireAuth, bookController.showCreate);
router.post('/create', requireAuth, bookController.create);
router.get('/:id', requireAuth, bookController.show);
router.get('/:id/edit', requireAuth, bookController.showEdit);
router.post('/:id/edit', requireAuth, bookController.update);
router.post('/:id/delete', requireAuth, bookController.delete);

module.exports = router;
```

## Views (EJS Templates)

### 1. views/partials/header.ejs
```html
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title><%= typeof title !== 'undefined' ? title : 'Aplikasi CRUD Buku' %></title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="/css/style.css">
</head>
<body>
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
        <div class="container">
            <a class="navbar-brand" href="/">Aplikasi CRUD Buku</a>
            <div class="navbar-nav ms-auto">
                <% if (typeof user !== 'undefined' && user.userId) { %>
                    <span class="navbar-text me-3">Halo, <%= user.username %>!</span>
                    <a class="nav-link" href="/books">Daftar Buku</a>
                    <a class="nav-link" href="/books/create">Tambah Buku</a>
                    <a class="nav-link" href="/auth/logout">Logout</a>
                <% } else { %>
                    <a class="nav-link" href="/auth/login">Login</a>
                    <a class="nav-link" href="/auth/register">Register</a>
                <% } %>
            </div>
        </div>
    </nav>
    <div class="container mt-4">
```

### 2. views/partials/footer.ejs
```html
    </div>
    <footer class="bg-dark text-white text-center py-3 mt-5">
        <p>&copy; 2024 Aplikasi CRUD Buku. All rights reserved.</p>
    </footer>
    
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

### 3. views/index.ejs
```html
<%- include('partials/header') %>

<div class="jumbotron bg-primary text-white p-5 rounded">
    <h1 class="display-4">Selamat Datang di Aplikasi CRUD Buku</h1>
    <p class="lead">Kelola koleksi buku Anda dengan mudah dan efisien.</p>
    <% if (typeof user === 'undefined' || !user.userId) { %>
        <a class="btn btn-light btn-lg" href="/auth/register">Mulai Sekarang</a>
    <% } else { %>
        <a class="btn btn-light btn-lg" href="/books">Lihat Daftar Buku</a>
    <% } %>
</div>

<div class="row mt-5">
    <div class="col-md-4">
        <div class="card">
            <div class="card-body">
                <h5 class="card-title">Tambah Buku</h5>
                <p class="card-text">Tambahkan buku baru ke dalam koleksi Anda.</p>
            </div>
        </div>
    </div>
    <div class="col-md-4">
        <div class="card">
            <div class="card-body">
                <h5 class="card-title">Kelola Buku</h5>
                <p class="card-text">Edit, hapus, dan lihat detail buku yang ada.</p>
            </div>
        </div>
    </div>
    <div class="col-md-4">
        <div class="card">
            <div class="card-body">
                <h5 class="card-title">Cari Buku</h5>
                <p class="card-text">Temukan buku yang Anda cari dengan mudah.</p>
            </div>
        </div>
    </div>
</div>

<%- include('partials/footer') %>
```

### 4. views/auth/register.ejs
```html
<%- include('../partials/header') %>

<div class="row justify-content-center">
    <div class="col-md-6">
        <div class="card">
            <div class="card-header">
                <h4>Registrasi</h4>
            </div>
            <div class="card-body">
                <% if (error) { %>
                    <div class="alert alert-danger"><%= error %></div>
                <% } %>
                <% if (success) { %>
                    <div class="alert alert-success"><%= success %></div>
                <% } %>
                
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
                    </div>
                    <div class="mb-3">
                        <label for="confirmPassword" class="form-label">Konfirmasi Password</label>
                        <input type="password" class="form-control" id="confirmPassword" name="confirmPassword" required>
                    </div>
                    <button type="submit" class="btn btn-primary w-100">Daftar</button>
                </form>
                
                <div class="text-center mt-3">
                    <p>Sudah punya akun? <a href="/auth/login">Login di sini</a></p>
                </div>
            </div>
        </div>
    </div>
</div>

<%- include('../partials/footer') %>
```

### 5. views/auth/login.ejs
```html
<%- include('../partials/header') %>

<div class="row justify-content-center">
    <div class="col-md-6">
        <div class="card">
            <div class="card-header">
                <h4>Login</h4>
            </div>
            <div class="card-body">
                <% if (error) { %>
                    <div class="alert alert-danger"><%= error %></div>
                <% } %>
                
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
                    <p>Belum punya akun? <a href="/auth/register">Daftar di sini</a></p>
                </div>
            </div>
        </div>
    </div>
</div>

<%- include('../partials/footer') %>
```

### 6. views/books/index.ejs
```html
<%- include('../partials/header') %>

<div class="d-flex justify-content-between align-items-center mb-4">
    <h2>Daftar Buku</h2>
    <a href="/books/create" class="btn btn-primary">Tambah Buku Baru</a>
</div>

<% if (books.length === 0) { %>
    <div class="alert alert-info">
        Belum ada buku yang ditambahkan. <a href="/books/create">Tambah buku pertama</a>
    </div>
<% } else { %>
    <div class="row">
        <% books.forEach(book => { %>
            <div class="col-md-4 mb-4">
                <div class="card">
                    <div class="card-body">
                        <h5 class="card-title"><%= book.title %></h5>
                        <h6 class="card-subtitle mb-2 text-muted">oleh <%= book.author %></h6>
                        <p class="card-text">
                            <small class="text-muted">
                                Penerbit: <%= book.publisher || 'Tidak diketahui' %><br>
                                Tahun: <%= book.year_published || 'Tidak diketahui' %><br>
                                Ditambahkan oleh: <%= book.username %>
                            </small>
                        </p>
                        <div class="d-flex gap-2">
                            <a href="/books/<%= book.id %>" class="btn btn-info btn-sm">Detail</a>
                            <a href="/books/<%= book.id %>/edit" class="btn btn-warning btn-sm">Edit</a>
                            <form action="/books/<%= book.id %>/delete" method="POST" style="display: inline;">
                                <button type="submit" class="btn btn-danger btn-sm" 
                                        onclick="return confirm('Yakin ingin menghapus buku ini?')">
                                    Hapus
                                </button>
                            </form>
                        </div>
                    </div>
                </div>
            </div>
        <% }) %>
    </div>
<% } %>

<%- include('../partials/footer') %>
```

### 7. views/books/create.ejs
```html
<%- include('../partials/header') %>

<div class="row justify-content-center">
    <div class="col-md-8">
        <div class="card">
            <div class="card-header">
                <h4>Tambah Buku Baru</h4>
            </div>
            <div class="card-body">
                <% if (error) { %>
                    <div class="alert alert-danger"><%= error %></div>
                <% } %>
                
                <form action="/books/create" method="POST">
                    <div class="row">
                        <div class="col-md-6">
                            <div class="mb-3">
                                <label for="title" class="form-label">Judul Buku *</label>
                                <input type="text" class="form-control" id="title" name="title" required>
                            </div>
                        </div>
                        <div class="col-md-6">
                            <div class="mb-3">
                                <label for="author" class="form-label">Penulis *</label>
                                <input type="text" class="form-control" id="author" name="author" required>
                            </div>
                        </div>
                    </div>
                    
                    <div class="row">
                        <div class="col-md-6">
                            <div class="mb-3">
                                <label for="publisher" class="form-label">Penerbit</label>
                                <input type="text" class="form-control" id="publisher" name="publisher">
                            </div>
                        </div>
                        <div class="col-md-6">
                            <div class="mb-3">
                                <label for="year_published" class="form-label">Tahun Terbit</label>
                                <input type="number" class="form-control" id="year_published" name="year_published">
                            </div>
                        </div>
                    </div>
                    
                    <div class="mb-3">
                        <label for="isbn" class="form-label">ISBN</label>
                        <input type="text" class="form-control" id="isbn" name="isbn">
                    </div>
                    
                    <div class="mb-3">
                        <label for="description" class="form-label">Deskripsi</label>
                        <textarea class="form-control" id="description" name="description" rows="4"></textarea>
                    </div>
                    
                    <div class="d-flex gap-2">
                        <button type="submit" class="btn btn-primary">Simpan</button>
                        <a href="/books" class="btn btn-secondary">Batal</a>
                    </div>
                </form>
            </div>
        </div>
    </div>
</div>

<%- include('../partials/footer') %>
```

### 8. views/books/edit.ejs
```html
<%- include('../partials/header') %>

<div class="row justify-content-center">
    <div class="col-md-8">
        <div class="card">
            <div class="card-header">
                <h4>Edit Buku</h4>
            </div>
            <div class="card-body">
                <% if (error) { %>
                    <div class="alert alert-danger"><%= error %></div>
                <% } %>
                
                <form action="/books/<%= book.id %>/edit" method="POST">
                    <div class="row">
                        <div class="col-md-6">
                            <div class="mb-3">
                                <label for="title" class="form-label">Judul Buku *</label>
                                <input type="text" class="form-control" id="title" name="title" 
                                       value="<%= book.title %>" required>
                            </div>
                        </div>
                        <div class="col-md-6">
                            <div class="mb-3">
                                <label for="author" class="form-label">Penulis *</label>
                                <input type="text" class="form-control" id="author" name="author" 
                                       value="<%= book.author %>" required>
                            </div>
                        </div>
                    </div>
                    
                    <div class="row">
                        <div class="col-md-6">
                            <div class="mb-3">
                                <label for="publisher" class="form-label">Penerbit</label>
                                <input type="text" class="form-control" id="publisher" name="publisher" 
                                       value="<%= book.publisher || '' %>">
                            </div>
                        </div>
                        <div class="col-md-6">
                            <div class="mb-3">
                                <label for="year_published" class="form-label">Tahun Terbit</label>
                                <input type="number" class="form-control" id="year_published" name="year_published" 
                                       value="<%= book.year_published || '' %>">
                            </div>
                        </div>
                    </div>
                    
                    <div class="mb-3">
                        <label for="isbn" class="form-label">ISBN</label>
                        <input type="text" class="form-control" id="isbn" name="isbn" 
                               value="<%= book.isbn || '' %>">
                    </div>
                    
                    <div class="mb-3">
                        <label for="description" class="form-label">Deskripsi</label>
                        <textarea class="form-control" id="description" name="description" rows="4"><%= book.description || '' %></textarea>
                    </div>
                    
                    <div class="d-flex gap-2">
                        <button type="submit" class="btn btn-primary">Update</button>
                        <a href="/books" class="btn btn-secondary">Batal</a>
                    </div>
                </form>
            </div>
        </div>
    </div>
</div>

<%- include('../partials/footer') %>
```

### 9. views/books/show.ejs
```html
<%- include('../partials/header') %>

<div class="row justify-content-center">
    <div class="col-md-8">
        <div class="card">
            <div class="card-header d-flex justify-content-between align-items-center">
                <h4>Detail Buku</h4>
                <div>
                    <a href="/books/<%= book.id %>/edit" class="btn btn-warning btn-sm">Edit</a>
                    <a href="/books" class="btn btn-secondary btn-sm">Kembali</a>
                </div>
            </div>
            <div class="card-body">
                <div class="row">
                    <div class="col-md-6">
                        <h5 class="card-title"><%= book.title %></h5>
                        <h6 class="card-subtitle mb-3 text-muted">oleh <%= book.author %></h6>
                    </div>
                    <div class="col-md-6 text-end">
                        <small class="text-muted">
                            Ditambahkan oleh: <%= book.username %><br>
                            Tanggal: <%= new Date(book.created_at).toLocaleDateString('id-ID') %>
                        </small>
                    </div>
                </div>
                
                <hr>
                
                <div class="row">
                    <div class="col-md-6">
                        <p><strong>Penerbit:</strong> <%= book.publisher || 'Tidak diketahui' %></p>
                        <p><strong>Tahun Terbit:</strong> <%= book.year_published || 'Tidak diketahui' %></p>
                    </div>
                    <div class="col-md-6">
                        <p><strong>ISBN:</strong> <%= book.isbn || 'Tidak diketahui' %></p>
                    </div>
                </div>
                
                <% if (book.description) { %>
                    <div class="mt-3">
                        <strong>Deskripsi:</strong>
                        <p class="mt-2"><%= book.description %></p>
                    </div>
                <% } %>
                
                <hr>
                
                <div class="d-flex gap-2">
                    <a href="/books/<%= book.id %>/edit" class="btn btn-warning">Edit Buku</a>
                    <form action="/books/<%= book.id %>/delete" method="POST" style="display: inline;">
                        <button type="submit" class="btn btn-danger" 
                                onclick="return confirm('Yakin ingin menghapus buku ini?')">
                            Hapus Buku
                        </button>
                    </form>
                    <a href="/books" class="btn btn-secondary">Kembali ke Daftar</a>
                </div>
            </div>
        </div>
    </div>
</div>

<%- include('../partials/footer') %>
```

### 10. public/css/style.css
```css
body {
    background-color: #f8f9fa;
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

.navbar-brand {
    font-weight: bold;
}

.card {
    border: none;
    box-shadow: 0 0.125rem 0.25rem rgba(0, 0, 0, 0.075);
    transition: box-shadow 0.15s ease-in-out;
}

.card:hover {
    box-shadow: 0 0.5rem 1rem rgba(0, 0, 0, 0.15);
}

.jumbotron {
    background: linear-gradient(135deg, #007bff, #0056b3);
}

.btn {
    border-radius: 0.375rem;
}

.form-control:focus {
    border-color: #007bff;
    box-shadow: 0 0 0 0.2rem rgba(0, 123, 255, 0.25);
}

footer {
    margin-top: auto;
}

.alert {
    border-radius: 0.375rem;
}

.card-title {
    color: #333;
    font-weight: 600;
}

.card-subtitle {
    font-style: italic;
}

.navbar-text {
    color: rgba(255, 255, 255, 0.75) !important;
}
```

## App.js (Main File)

```javascript
const express = require('express');
const session = require('express-session');
const bodyParser = require('body-parser');
const path = require('path');
require('dotenv').config();

const app = express();

// Import routes
const authRoutes = require('./routes/auth');
const bookRoutes = require('./routes/books');

// Middleware
app.use(bodyParser.urlencoded({ extended: true }));
app.use(bodyParser.json());

// Static files
app.use(express.static(path.join(__dirname, 'public')));

// Session configuration
app.use(session({
    secret: process.env.SESSION_SECRET,
    resave: false,
    saveUninitialized: false,
    cookie: {
        secure: false, // Set true jika menggunakan HTTPS
        maxAge: 24 * 60 * 60 * 1000 // 24 jam
    }
}));

// View engine setup
app.set('view engine', 'ejs');
app.set('views', path.join(__dirname, 'views'));

// Routes
app.get('/', (req, res) => {
    res.render('index', { user: req.session });
});

app.use('/auth', authRoutes);
app.use('/books', bookRoutes);

// Error handling middleware
app.use((req, res, next) => {
    res.status(404).send('Halaman tidak ditemukan');
});

app.use((err, req, res, next) => {
    console.error(err.stack);
    res.status(500).send('Terjadi kesalahan server');
});

const PORT = process.env.PORT || 3000;

app.listen(PORT, () => {
    console.log(`Server berjalan pada port ${PORT}`);
    console.log(`Buka browser dan akses: http://localhost:${PORT}`);
});
```

## Menjalankan Aplikasi

### 1. Persiapan Database
- Pastikan MySQL sudah terinstall dan berjalan
- Buat database dan tabel sesuai dengan SQL yang sudah disediakan
- Update file `.env` dengan konfigurasi database yang sesuai

### 2. Install Dependencies
```bash
npm install
```

### 3. Jalankan Aplikasi
Untuk development:
```bash
npm run dev
```

Untuk production:
```bash
npm start
```

### 4. Akses Aplikasi
Buka browser dan akses: `http://localhost:3000`

## Fitur yang Tersedia

1. **Registrasi User**: Pendaftaran akun baru dengan validasi
2. **Login/Logout**: Sistem autentikasi dengan session
3. **CRUD Buku**:
   - **Create**: Tambah buku baru
   - **Read**: Lihat daftar dan detail buku
   - **Update**: Edit informasi buku
   - **Delete**: Hapus buku
4. **Relasi Database**: Buku terhubung dengan user yang menambahkannya
5. **Responsive Design**: Menggunakan Bootstrap untuk tampilan yang responsif

## Tips Pengembangan Lebih Lanjut

1. **Validasi**: Tambahkan validasi yang lebih ketat di sisi server
2. **Pagination**: Implementasikan pagination untuk daftar buku
3. **Search**: Tambahkan fitur pencarian buku
4. **Upload Image**: Tambahkan fitur upload cover buku
5. **Authorization**: Implementasikan sistem role (admin, user)
6. **API**: Buat REST API untuk mobile app
7. **Security**: Implementasikan CSRF protection dan rate limiting

Dengan mengikuti panduan ini, Anda akan memiliki aplikasi CRUD lengkap dengan Node.js, EJS, dan MySQL yang mencakup sistem autentikasi dan manajemen data buku.