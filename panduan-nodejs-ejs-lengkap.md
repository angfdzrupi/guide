# Panduan Lengkap Node.js dengan EJS Templating

## Daftar Isi
1. [Persiapan Awal](#persiapan-awal)
2. [Setup Project](#setup-project)
3. [Struktur Folder](#struktur-folder)
4. [Konfigurasi Database](#konfigurasi-database)
5. [Model](#model)
6. [Controller](#controller)
7. [Routes](#routes)
8. [Views (EJS Templates)](#views-ejs-templates)
9. [Middleware](#middleware)
10. [Main App](#main-app)
11. [Menjalankan Aplikasi](#menjalankan-aplikasi)

## Persiapan Awal

### 1. Install Node.js
Pastikan Node.js sudah terinstall di komputer Anda. Download dari [nodejs.org](https://nodejs.org)

### 2. Install MySQL
Pastikan MySQL sudah terinstall dan berjalan di komputer Anda.

### 3. Buat Database
```sql
CREATE DATABASE nodejs_ejs_app;
USE nodejs_ejs_app;
```

## Setup Project

### 1. Inisialisasi Project
```bash
mkdir nodejs-ejs-app
cd nodejs-ejs-app
npm init -y
```

### 2. Install Dependencies
```bash
# Dependencies utama
npm install express ejs mysql2 bcrypt express-session dotenv

# Dependencies untuk development
npm install --save-dev nodemon
```

### 3. Update package.json
Tambahkan script untuk development:
```json
{
  "scripts": {
    "start": "node app.js",
    "dev": "nodemon app.js"
  }
}
```

## Struktur Folder

```
nodejs-ejs-app/
│
├── config/
│   └── database.js
├── controllers/
│   ├── authController.js
│   └── bookController.js
├── middleware/
│   └── auth.js
├── models/
│   ├── index.js
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
│   └── dashboard.ejs
├── public/
│   ├── css/
│   │   └── style.css
│   └── js/
│       └── main.js
├── .env
├── app.js
└── package.json
```

## Konfigurasi Database

### 1. File .env
```env
# Database Configuration
DB_HOST=localhost
DB_USER=root
DB_PASSWORD=your_password
DB_NAME=nodejs_ejs_app

# Session Secret
SESSION_SECRET=your_session_secret_key

# Port
PORT=3000
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
    console.error('Error connecting to database:', err);
    return;
  }
  console.log('Connected to MySQL database');
});

module.exports = connection;
```

## Model

### 1. models/index.js
```javascript
const db = require('../config/database');

// Membuat tabel users
const createUsersTable = () => {
  const sql = `
    CREATE TABLE IF NOT EXISTS users (
      id INT AUTO_INCREMENT PRIMARY KEY,
      username VARCHAR(50) UNIQUE NOT NULL,
      email VARCHAR(100) UNIQUE NOT NULL,
      password VARCHAR(255) NOT NULL,
      created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
      updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
    )
  `;
  
  db.query(sql, (err, result) => {
    if (err) {
      console.error('Error creating users table:', err);
    } else {
      console.log('Users table ready');
    }
  });
};

// Membuat tabel books
const createBooksTable = () => {
  const sql = `
    CREATE TABLE IF NOT EXISTS books (
      id INT AUTO_INCREMENT PRIMARY KEY,
      title VARCHAR(255) NOT NULL,
      author VARCHAR(255) NOT NULL,
      isbn VARCHAR(20) UNIQUE,
      publication_year INT,
      description TEXT,
      user_id INT,
      created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
      updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
      FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
    )
  `;
  
  db.query(sql, (err, result) => {
    if (err) {
      console.error('Error creating books table:', err);
    } else {
      console.log('Books table ready');
    }
  });
};

// Inisialisasi semua tabel
const initializeDatabase = () => {
  createUsersTable();
  createBooksTable();
};

module.exports = {
  initializeDatabase,
  createUsersTable,
  createBooksTable
};
```

### 2. models/User.js
```javascript
const db = require('../config/database');
const bcrypt = require('bcrypt');

class User {
  // Membuat user baru
  static async create(userData) {
    const { username, email, password } = userData;
    
    try {
      // Hash password
      const hashedPassword = await bcrypt.hash(password, 10);
      
      return new Promise((resolve, reject) => {
        const sql = 'INSERT INTO users (username, email, password) VALUES (?, ?, ?)';
        db.query(sql, [username, email, hashedPassword], (err, result) => {
          if (err) {
            reject(err);
          } else {
            resolve(result);
          }
        });
      });
    } catch (error) {
      throw error;
    }
  }

  // Mencari user berdasarkan email
  static findByEmail(email) {
    return new Promise((resolve, reject) => {
      const sql = 'SELECT * FROM users WHERE email = ?';
      db.query(sql, [email], (err, results) => {
        if (err) {
          reject(err);
        } else {
          resolve(results[0]);
        }
      });
    });
  }

  // Mencari user berdasarkan ID
  static findById(id) {
    return new Promise((resolve, reject) => {
      const sql = 'SELECT * FROM users WHERE id = ?';
      db.query(sql, [id], (err, results) => {
        if (err) {
          reject(err);
        } else {
          resolve(results[0]);
        }
      });
    });
  }

  // Validasi password
  static async validatePassword(inputPassword, hashedPassword) {
    return await bcrypt.compare(inputPassword, hashedPassword);
  }
}

module.exports = User;
```

### 3. models/Book.js
```javascript
const db = require('../config/database');

class Book {
  // Membuat buku baru
  static create(bookData) {
    const { title, author, isbn, publication_year, description, user_id } = bookData;
    
    return new Promise((resolve, reject) => {
      const sql = 'INSERT INTO books (title, author, isbn, publication_year, description, user_id) VALUES (?, ?, ?, ?, ?, ?)';
      db.query(sql, [title, author, isbn, publication_year, description, user_id], (err, result) => {
        if (err) {
          reject(err);
        } else {
          resolve(result);
        }
      });
    });
  }

  // Mengambil semua buku
  static findAll() {
    return new Promise((resolve, reject) => {
      const sql = `
        SELECT b.*, u.username 
        FROM books b 
        LEFT JOIN users u ON b.user_id = u.id 
        ORDER BY b.created_at DESC
      `;
      db.query(sql, (err, results) => {
        if (err) {
          reject(err);
        } else {
          resolve(results);
        }
      });
    });
  }

  // Mencari buku berdasarkan ID
  static findById(id) {
    return new Promise((resolve, reject) => {
      const sql = 'SELECT * FROM books WHERE id = ?';
      db.query(sql, [id], (err, results) => {
        if (err) {
          reject(err);
        } else {
          resolve(results[0]);
        }
      });
    });
  }

  // Update buku
  static update(id, bookData) {
    const { title, author, isbn, publication_year, description } = bookData;
    
    return new Promise((resolve, reject) => {
      const sql = 'UPDATE books SET title = ?, author = ?, isbn = ?, publication_year = ?, description = ? WHERE id = ?';
      db.query(sql, [title, author, isbn, publication_year, description, id], (err, result) => {
        if (err) {
          reject(err);
        } else {
          resolve(result);
        }
      });
    });
  }

  // Hapus buku
  static delete(id) {
    return new Promise((resolve, reject) => {
      const sql = 'DELETE FROM books WHERE id = ?';
      db.query(sql, [id], (err, result) => {
        if (err) {
          reject(err);
        } else {
          resolve(result);
        }
      });
    });
  }

  // Mencari buku berdasarkan user ID
  static findByUserId(userId) {
    return new Promise((resolve, reject) => {
      const sql = 'SELECT * FROM books WHERE user_id = ? ORDER BY created_at DESC';
      db.query(sql, [userId], (err, results) => {
        if (err) {
          reject(err);
        } else {
          resolve(results);
        }
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

class AuthController {
  // Menampilkan halaman register
  static showRegister(req, res) {
    res.render('auth/register', { 
      title: 'Register',
      error: null 
    });
  }

  // Proses registrasi
  static async register(req, res) {
    const { username, email, password, confirmPassword } = req.body;

    try {
      // Validasi input
      if (!username || !email || !password || !confirmPassword) {
        return res.render('auth/register', {
          title: 'Register',
          error: 'Semua field harus diisi'
        });
      }

      if (password !== confirmPassword) {
        return res.render('auth/register', {
          title: 'Register',
          error: 'Password tidak cocok'
        });
      }

      if (password.length < 6) {
        return res.render('auth/register', {
          title: 'Register',
          error: 'Password minimal 6 karakter'
        });
      }

      // Cek apakah user sudah ada
      const existingUser = await User.findByEmail(email);
      if (existingUser) {
        return res.render('auth/register', {
          title: 'Register',
          error: 'Email sudah terdaftar'
        });
      }

      // Buat user baru
      await User.create({ username, email, password });
      
      res.redirect('/auth/login?message=Registrasi berhasil! Silakan login.');
    } catch (error) {
      console.error('Registration error:', error);
      res.render('auth/register', {
        title: 'Register',
        error: 'Terjadi kesalahan saat registrasi'
      });
    }
  }

  // Menampilkan halaman login
  static showLogin(req, res) {
    const message = req.query.message || null;
    res.render('auth/login', { 
      title: 'Login',
      error: null,
      message 
    });
  }

  // Proses login
  static async login(req, res) {
    const { email, password } = req.body;

    try {
      // Validasi input
      if (!email || !password) {
        return res.render('auth/login', {
          title: 'Login',
          error: 'Email dan password harus diisi',
          message: null
        });
      }

      // Cari user berdasarkan email
      const user = await User.findByEmail(email);
      if (!user) {
        return res.render('auth/login', {
          title: 'Login',
          error: 'Email atau password salah',
          message: null
        });
      }

      // Validasi password
      const isValidPassword = await User.validatePassword(password, user.password);
      if (!isValidPassword) {
        return res.render('auth/login', {
          title: 'Login',
          error: 'Email atau password salah',
          message: null
        });
      }

      // Set session
      req.session.user = {
        id: user.id,
        username: user.username,
        email: user.email
      };

      res.redirect('/dashboard');
    } catch (error) {
      console.error('Login error:', error);
      res.render('auth/login', {
        title: 'Login',
        error: 'Terjadi kesalahan saat login',
        message: null
      });
    }
  }

  // Logout
  static logout(req, res) {
    req.session.destroy((err) => {
      if (err) {
        console.error('Logout error:', err);
      }
      res.redirect('/auth/login');
    });
  }

  // Dashboard
  static showDashboard(req, res) {
    res.render('dashboard', {
      title: 'Dashboard',
      user: req.session.user
    });
  }
}

module.exports = AuthController;
```

### 2. controllers/bookController.js
```javascript
const Book = require('../models/Book');

class BookController {
  // Menampilkan semua buku
  static async index(req, res) {
    try {
      const books = await Book.findAll();
      res.render('books/index', {
        title: 'Daftar Buku',
        books,
        user: req.session.user
      });
    } catch (error) {
      console.error('Error fetching books:', error);
      res.render('books/index', {
        title: 'Daftar Buku',
        books: [],
        user: req.session.user,
        error: 'Terjadi kesalahan saat mengambil data buku'
      });
    }
  }

  // Menampilkan form tambah buku
  static showCreate(req, res) {
    res.render('books/create', {
      title: 'Tambah Buku',
      user: req.session.user,
      error: null
    });
  }

  // Proses tambah buku
  static async create(req, res) {
    const { title, author, isbn, publication_year, description } = req.body;

    try {
      // Validasi input
      if (!title || !author) {
        return res.render('books/create', {
          title: 'Tambah Buku',
          user: req.session.user,
          error: 'Judul dan penulis harus diisi'
        });
      }

      // Buat buku baru
      await Book.create({
        title,
        author,
        isbn: isbn || null,
        publication_year: publication_year || null,
        description: description || null,
        user_id: req.session.user.id
      });

      res.redirect('/books?message=Buku berhasil ditambahkan');
    } catch (error) {
      console.error('Error creating book:', error);
      res.render('books/create', {
        title: 'Tambah Buku',
        user: req.session.user,
        error: 'Terjadi kesalahan saat menambah buku'
      });
    }
  }

  // Menampilkan detail buku
  static async show(req, res) {
    const { id } = req.params;

    try {
      const book = await Book.findById(id);
      if (!book) {
        return res.redirect('/books?error=Buku tidak ditemukan');
      }

      res.render('books/show', {
        title: `Detail Buku - ${book.title}`,
        book,
        user: req.session.user
      });
    } catch (error) {
      console.error('Error fetching book:', error);
      res.redirect('/books?error=Terjadi kesalahan saat mengambil data buku');
    }
  }

  // Menampilkan form edit buku
  static async showEdit(req, res) {
    const { id } = req.params;

    try {
      const book = await Book.findById(id);
      if (!book) {
        return res.redirect('/books?error=Buku tidak ditemukan');
      }

      // Cek apakah user berhak mengedit buku ini
      if (book.user_id !== req.session.user.id) {
        return res.redirect('/books?error=Anda tidak berhak mengedit buku ini');
      }

      res.render('books/edit', {
        title: `Edit Buku - ${book.title}`,
        book,
        user: req.session.user,
        error: null
      });
    } catch (error) {
      console.error('Error fetching book for edit:', error);
      res.redirect('/books?error=Terjadi kesalahan saat mengambil data buku');
    }
  }

  // Proses update buku
  static async update(req, res) {
    const { id } = req.params;
    const { title, author, isbn, publication_year, description } = req.body;

    try {
      // Cek apakah buku ada
      const book = await Book.findById(id);
      if (!book) {
        return res.redirect('/books?error=Buku tidak ditemukan');
      }

      // Cek apakah user berhak mengedit buku ini
      if (book.user_id !== req.session.user.id) {
        return res.redirect('/books?error=Anda tidak berhak mengedit buku ini');
      }

      // Validasi input
      if (!title || !author) {
        return res.render('books/edit', {
          title: `Edit Buku - ${book.title}`,
          book,
          user: req.session.user,
          error: 'Judul dan penulis harus diisi'
        });
      }

      // Update buku
      await Book.update(id, {
        title,
        author,
        isbn: isbn || null,
        publication_year: publication_year || null,
        description: description || null
      });

      res.redirect(`/books/${id}?message=Buku berhasil diperbarui`);
    } catch (error) {
      console.error('Error updating book:', error);
      const book = await Book.findById(id);
      res.render('books/edit', {
        title: `Edit Buku - ${book.title}`,
        book,
        user: req.session.user,
        error: 'Terjadi kesalahan saat memperbarui buku'
      });
    }
  }

  // Hapus buku
  static async delete(req, res) {
    const { id } = req.params;

    try {
      // Cek apakah buku ada
      const book = await Book.findById(id);
      if (!book) {
        return res.redirect('/books?error=Buku tidak ditemukan');
      }

      // Cek apakah user berhak menghapus buku ini
      if (book.user_id !== req.session.user.id) {
        return res.redirect('/books?error=Anda tidak berhak menghapus buku ini');
      }

      // Hapus buku
      await Book.delete(id);
      res.redirect('/books?message=Buku berhasil dihapus');
    } catch (error) {
      console.error('Error deleting book:', error);
      res.redirect('/books?error=Terjadi kesalahan saat menghapus buku');
    }
  }
}

module.exports = BookController;
```

## Routes

### 1. routes/auth.js
```javascript
const express = require('express');
const router = express.Router();
const AuthController = require('../controllers/authController');
const { requireAuth, redirectIfAuthenticated } = require('../middleware/auth');

// Routes untuk guest (belum login)
router.get('/register', redirectIfAuthenticated, AuthController.showRegister);
router.post('/register', redirectIfAuthenticated, AuthController.register);
router.get('/login', redirectIfAuthenticated, AuthController.showLogin);
router.post('/login', redirectIfAuthenticated, AuthController.login);

// Routes untuk user yang sudah login
router.post('/logout', requireAuth, AuthController.logout);

module.exports = router;
```

### 2. routes/books.js
```javascript
const express = require('express');
const router = express.Router();
const BookController = require('../controllers/bookController');
const { requireAuth } = require('../middleware/auth');

// Semua routes di sini memerlukan autentikasi
router.use(requireAuth);

// Routes untuk CRUD buku
router.get('/', BookController.index);
router.get('/create', BookController.showCreate);
router.post('/create', BookController.create);
router.get('/:id', BookController.show);
router.get('/:id/edit', BookController.showEdit);
router.post('/:id/edit', BookController.update);
router.post('/:id/delete', BookController.delete);

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
    <title><%= title %> - Node.js EJS App</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="/css/style.css">
</head>
<body>
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
        <div class="container">
            <a class="navbar-brand" href="/">Node.js EJS App</a>
            
            <% if (typeof user !== 'undefined' && user) { %>
                <div class="navbar-nav ms-auto">
                    <a class="nav-link" href="/dashboard">Dashboard</a>
                    <a class="nav-link" href="/books">Buku</a>
                    <span class="navbar-text me-3">Halo, <%= user.username %>!</span>
                    <form method="POST" action="/auth/logout" class="d-inline">
                        <button type="submit" class="btn btn-outline-light btn-sm">Logout</button>
                    </form>
                </div>
            <% } else { %>
                <div class="navbar-nav ms-auto">
                    <a class="nav-link" href="/auth/login">Login</a>
                    <a class="nav-link" href="/auth/register">Register</a>
                </div>
            <% } %>
        </div>
    </nav>

    <div class="container mt-4">
```

### 2. views/partials/footer.ejs
```html
    </div>

    <footer class="bg-dark text-light text-center py-3 mt-5">
        <div class="container">
            <p>&copy; 2023 Node.js EJS App. All rights reserved.</p>
        </div>
    </footer>

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/js/bootstrap.bundle.min.js"></script>
    <script src="/js/main.js"></script>
</body>
</html>
```

### 3. views/auth/register.ejs
```html
<%- include('../partials/header') %>

<div class="row justify-content-center">
    <div class="col-md-6">
        <div class="card">
            <div class="card-header">
                <h4 class="mb-0">Registrasi</h4>
            </div>
            <div class="card-body">
                <% if (error) { %>
                    <div class="alert alert-danger"><%= error %></div>
                <% } %>

                <form method="POST" action="/auth/register">
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

### 4. views/auth/login.ejs
```html
<%- include('../partials/header') %>

<div class="row justify-content-center">
    <div class="col-md-6">
        <div class="card">
            <div class="card-header">
                <h4 class="mb-0">Login</h4>
            </div>
            <div class="card-body">
                <% if (error) { %>
                    <div class="alert alert-danger"><%= error %></div>
                <% } %>

                <% if (message) { %>
                    <div class="alert alert-success"><%= message %></div>
                <% } %>

                <form method="POST" action="/auth/login">
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

### 5. views/dashboard.ejs
```html
<%- include('partials/header') %>

<div class="row">
    <div class="col-12">
        <h1>Dashboard</h1>
        <p>Selamat datang, <strong><%= user.username %></strong>!</p>
        
        <div class="row mt-4">
            <div class="col-md-6 col-lg-4 mb-3">
                <div class="card">
                    <div class="card-body">
                        <h5 class="card-title">Kelola Buku</h5>
                        <p class="card-text">Tambah, edit, dan hapus buku dalam koleksi Anda.</p>
                        <a href="/books" class="btn btn-primary">Lihat Buku</a>
                    </div>
                </div>
            </div>
            
            <div class="col-md-6 col-lg-4 mb-3">
                <div class="card">
                    <div class="card-body">
                        <h5 class="card-title">Tambah Buku Baru</h5>
                        <p class="card-text">Tambahkan buku baru ke dalam koleksi Anda.</p>
                        <a href="/books/create" class="btn btn-success">Tambah Buku</a>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>

<%- include('partials/footer') %>
```

### 6. views/books/index.ejs
```html
<%- include('../partials/header') %>

<div class="row">
    <div class="col-12">
        <div class="d-flex justify-content-between align-items-center mb-3">
            <h1>Daftar Buku</h1>
            <a href="/books/create" class="btn btn-primary">Tambah Buku</a>
        </div>

        <% if (typeof req !== 'undefined' && req.query.message) { %>
            <div class="alert alert-success"><%= req.query.message %></div>
        <% } %>

        <% if (typeof req !== 'undefined' && req.query.error) { %>
            <div class="alert alert-danger"><%= req.query.error %></div>
        <% } %>

        <% if (books.length === 0) { %>
            <div class="alert alert-info">
                <h4>Belum ada buku</h4>
                <p>Anda belum menambahkan buku apapun. <a href="/books/create">Tambah buku pertama</a> Anda sekarang!</p>
            </div>
        <% } else { %>
            <div class="row">
                <% books.forEach(book => { %>
                    <div class="col-md-6 col-lg-4 mb-4">
                        <div class="card h-100">
                            <div class="card-body">
                                <h5 class="card-title"><%= book.title %></h5>
                                <p class="card-text">
                                    <strong>Penulis:</strong> <%= book.author %><br>
                                    <% if (book.publication_year) { %>
                                        <strong>Tahun:</strong> <%= book.publication_year %><br>
                                    <% } %>
                                    <% if (book.isbn) { %>
                                        <strong>ISBN:</strong> <%= book.isbn %><br>
                                    <% } %>
                                    <small class="text-muted">Ditambahkan oleh: <%= book.username || 'Unknown' %></small>
                                </p>
                                <% if (book.description) { %>
                                    <p class="card-text">
                                        <%= book.description.length > 100 ? book.description.substring(0, 100) + '...' : book.description %>
                                    </p>
                                <% } %>
                            </div>
                            <div class="card-footer">
                                <div class="btn-group w-100" role="group">
                                    <a href="/books/<%= book.id %>" class="btn btn-outline-primary btn-sm">Detail</a>
                                    <% if (book.user_id === user.id) { %>
                                        <a href="/books/<%= book.id %>/edit" class="btn btn-outline-secondary btn-sm">Edit</a>
                                        <form method="POST" action="/books/<%= book.id %>/delete" class="d-inline" onsubmit="return confirm('Apakah Anda yakin ingin menghapus buku ini?')">
                                            <button type="submit" class="btn btn-outline-danger btn-sm">Hapus</button>
                                        </form>
                                    <% } %>
                                </div>
                            </div>
                        </div>
                    </div>
                <% }); %>
            </div>
        <% } %>
    </div>
</div>

<%- include('../partials/footer') %>
```

### 7. views/books/create.ejs
```html
<%- include('../partials/header') %>

<div class="row">
    <div class="col-md-8">
        <h1>Tambah Buku Baru</h1>

        <% if (error) { %>
            <div class="alert alert-danger"><%= error %></div>
        <% } %>

        <form method="POST" action="/books/create">
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
                        <label for="isbn" class="form-label">ISBN</label>
                        <input type="text" class="form-control" id="isbn" name="isbn">
                    </div>
                </div>
                <div class="col-md-6">
                    <div class="mb-3">
                        <label for="publication_year" class="form-label">Tahun Terbit</label>
                        <input type="number" class="form-control" id="publication_year" name="publication_year" min="1000" max="<%= new Date().getFullYear() %>">
                    </div>
                </div>
            </div>

            <div class="mb-3">
                <label for="description" class="form-label">Deskripsi</label>
                <textarea class="form-control" id="description" name="description" rows="4"></textarea>
            </div>

            <div class="mb-3">
                <button type="submit" class="btn btn-primary">Simpan Buku</button>
                <a href="/books" class="btn btn-secondary">Batal</a>
            </div>
        </form>
    </div>
</div>

<%- include('../partials/footer') %>
```

### 8. views/books/edit.ejs
```html
<%- include('../partials/header') %>

<div class="row">
    <div class="col-md-8">
        <h1>Edit Buku: <%= book.title %></h1>

        <% if (error) { %>
            <div class="alert alert-danger"><%= error %></div>
        <% } %>

        <form method="POST" action="/books/<%= book.id %>/edit">
            <div class="row">
                <div class="col-md-6">
                    <div class="mb-3">
                        <label for="title" class="form-label">Judul Buku *</label>
                        <input type="text" class="form-control" id="title" name="title" value="<%= book.title %>" required>
                    </div>
                </div>
                <div class="col-md-6">
                    <div class="mb-3">
                        <label for="author" class="form-label">Penulis *</label>
                        <input type="text" class="form-control" id="author" name="author" value="<%= book.author %>" required>
                    </div>
                </div>
            </div>

            <div class="row">
                <div class="col-md-6">
                    <div class="mb-3">
                        <label for="isbn" class="form-label">ISBN</label>
                        <input type="text" class="form-control" id="isbn" name="isbn" value="<%= book.isbn || '' %>">
                    </div>
                </div>
                <div class="col-md-6">
                    <div class="mb-3">
                        <label for="publication_year" class="form-label">Tahun Terbit</label>
                        <input type="number" class="form-control" id="publication_year" name="publication_year" value="<%= book.publication_year || '' %>" min="1000" max="<%= new Date().getFullYear() %>">
                    </div>
                </div>
            </div>

            <div class="mb-3">
                <label for="description" class="form-label">Deskripsi</label>
                <textarea class="form-control" id="description" name="description" rows="4"><%= book.description || '' %></textarea>
            </div>

            <div class="mb-3">
                <button type="submit" class="btn btn-primary">Update Buku</button>
                <a href="/books/<%= book.id %>" class="btn btn-secondary">Batal</a>
            </div>
        </form>
    </div>
</div>

<%- include('../partials/footer') %>
```

### 9. views/books/show.ejs
```html
<%- include('../partials/header') %>

<div class="row">
    <div class="col-md-8">
        <% if (typeof req !== 'undefined' && req.query.message) { %>
            <div class="alert alert-success"><%= req.query.message %></div>
        <% } %>

        <div class="card">
            <div class="card-header d-flex justify-content-between align-items-center">
                <h3 class="mb-0"><%= book.title %></h3>
                <% if (book.user_id === user.id) { %>
                    <div class="btn-group">
                        <a href="/books/<%= book.id %>/edit" class="btn btn-outline-primary btn-sm">Edit</a>
                        <form method="POST" action="/books/<%= book.id %>/delete" class="d-inline" onsubmit="return confirm('Apakah Anda yakin ingin menghapus buku ini?')">
                            <button type="submit" class="btn btn-outline-danger btn-sm">Hapus</button>
                        </form>
                    </div>
                <% } %>
            </div>
            <div class="card-body">
                <div class="row">
                    <div class="col-md-6">
                        <p><strong>Penulis:</strong> <%= book.author %></p>
                        <% if (book.isbn) { %>
                            <p><strong>ISBN:</strong> <%= book.isbn %></p>
                        <% } %>
                        <% if (book.publication_year) { %>
                            <p><strong>Tahun Terbit:</strong> <%= book.publication_year %></p>
                        <% } %>
                    </div>
                    <div class="col-md-6">
                        <p><strong>Ditambahkan:</strong> <%= new Date(book.created_at).toLocaleDateString('id-ID') %></p>
                        <% if (book.updated_at !== book.created_at) { %>
                            <p><strong>Diperbarui:</strong> <%= new Date(book.updated_at).toLocaleDateString('id-ID') %></p>
                        <% } %>
                    </div>
                </div>

                <% if (book.description) { %>
                    <hr>
                    <h5>Deskripsi</h5>
                    <p><%= book.description %></p>
                <% } %>
            </div>
        </div>

        <div class="mt-3">
            <a href="/books" class="btn btn-secondary">Kembali ke Daftar Buku</a>
        </div>
    </div>
</div>

<%- include('../partials/footer') %>
```

## Middleware

### middleware/auth.js
```javascript
// Middleware untuk memastikan user sudah login
const requireAuth = (req, res, next) => {
  if (!req.session.user) {
    return res.redirect('/auth/login');
  }
  next();
};

// Middleware untuk redirect user yang sudah login
const redirectIfAuthenticated = (req, res, next) => {
  if (req.session.user) {
    return res.redirect('/dashboard');
  }
  next();
};

module.exports = {
  requireAuth,
  redirectIfAuthenticated
};
```

## Main App

### app.js
```javascript
const express = require('express');
const session = require('express-session');
const path = require('path');
require('dotenv').config();

// Import models untuk inisialisasi database
const { initializeDatabase } = require('./models/index');

// Import routes
const authRoutes = require('./routes/auth');
const bookRoutes = require('./routes/books');

// Import controllers
const AuthController = require('./controllers/authController');

// Import middleware
const { requireAuth } = require('./middleware/auth');

const app = express();
const PORT = process.env.PORT || 3000;

// Inisialisasi database
initializeDatabase();

// Middleware
app.use(express.urlencoded({ extended: true }));
app.use(express.json());
app.use(express.static(path.join(__dirname, 'public')));

// Session configuration
app.use(session({
  secret: process.env.SESSION_SECRET || 'your-secret-key',
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
  if (req.session.user) {
    res.redirect('/dashboard');
  } else {
    res.redirect('/auth/login');
  }
});

app.get('/dashboard', requireAuth, AuthController.showDashboard);

// Auth routes
app.use('/auth', authRoutes);

// Book routes
app.use('/books', bookRoutes);

// 404 handler
app.use((req, res) => {
  res.status(404).render('404', { 
    title: 'Halaman Tidak Ditemukan',
    user: req.session.user || null
  });
});

// Error handler
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).render('500', { 
    title: 'Terjadi Kesalahan Server',
    user: req.session.user || null
  });
});

app.listen(PORT, () => {
  console.log(`Server berjalan di http://localhost:${PORT}`);
});
```

## File Tambahan

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

footer {
  margin-top: auto;
}

html, body {
  height: 100%;
}

#root {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}

.btn-group .btn {
  border-radius: 0;
}

.btn-group .btn:first-child {
  border-radius: 0.25rem 0 0 0.25rem;
}

.btn-group .btn:last-child {
  border-radius: 0 0.25rem 0.25rem 0;
}

.alert {
  border-radius: 0.5rem;
}
```

### public/js/main.js
```javascript
// Konfirmasi untuk tombol hapus
document.addEventListener('DOMContentLoaded', function() {
  // Auto-hide alerts after 5 seconds
  const alerts = document.querySelectorAll('.alert');
  alerts.forEach(alert => {
    if (alert.classList.contains('alert-success') || alert.classList.contains('alert-info')) {
      setTimeout(() => {
        alert.style.opacity = '0';
        setTimeout(() => {
          alert.remove();
        }, 300);
      }, 5000);
    }
  });

  // Form validation
  const forms = document.querySelectorAll('form');
  forms.forEach(form => {
    form.addEventListener('submit', function(e) {
      const requiredFields = form.querySelectorAll('[required]');
      let isValid = true;

      requiredFields.forEach(field => {
        if (!field.value.trim()) {
          isValid = false;
          field.classList.add('is-invalid');
        } else {
          field.classList.remove('is-invalid');
        }
      });

      if (!isValid) {
        e.preventDefault();
      }
    });
  });
});
```

## Menjalankan Aplikasi

### 1. Persiapan Database
Pastikan MySQL sudah berjalan dan buat database:
```sql
CREATE DATABASE nodejs_ejs_app;
```

### 2. Install Dependencies
```bash
npm install
```

### 3. Setup Environment
Buat file `.env` dan atur konfigurasi database sesuai dengan setup MySQL Anda.

### 4. Jalankan Aplikasi
```bash
# Development mode
npm run dev

# Production mode
npm start
```

### 5. Akses Aplikasi
Buka browser dan akses `http://localhost:3000`

## Fitur Aplikasi

### 1. Autentikasi
- **Registrasi**: User dapat mendaftar dengan username, email, dan password
- **Login**: User dapat login dengan email dan password
- **Logout**: User dapat logout dan mengakhiri session
- **Session Management**: Session otomatis expire setelah 24 jam

### 2. CRUD Buku
- **Create**: Tambah buku baru dengan informasi lengkap
- **Read**: Lihat daftar semua buku dan detail buku
- **Update**: Edit informasi buku (hanya pemilik buku)
- **Delete**: Hapus buku (hanya pemilik buku)

### 3. Keamanan
- Password di-hash menggunakan bcrypt
- Session-based authentication
- Authorization check untuk edit/delete buku
- Input validation pada semua form

### 4. User Experience
- Responsive design menggunakan Bootstrap
- Alert messages untuk feedback user
- Konfirmasi sebelum menghapus data
- Form validation di client-side dan server-side

## Tips Pengembangan

1. **Database**: Pastikan MySQL service berjalan sebelum menjalankan aplikasi
2. **Environment**: Selalu gunakan file `.env` untuk konfigurasi sensitif
3. **Error Handling**: Semua operasi database dibungkus dalam try-catch
4. **Security**: Jangan pernah menyimpan password dalam bentuk plain text
5. **Session**: Gunakan session secret yang kuat di production
6. **Validation**: Selalu validasi input baik di client maupun server side

Panduan ini memberikan fondasi yang solid untuk membangun aplikasi web dengan Node.js dan EJS. Anda dapat mengembangkannya lebih lanjut dengan menambahkan fitur seperti upload file, pagination, search, atau integrasi dengan service lain.