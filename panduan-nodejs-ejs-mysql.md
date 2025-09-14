# Panduan Lengkap Node.js + EJS + MySQL

## Daftar Isi
1. [Setup Awal](#setup-awal)
2. [Konfigurasi Database](#konfigurasi-database)
3. [Setup Express dan EJS](#setup-express-dan-ejs)
4. [Struktur Folder](#struktur-folder)
5. [Konfigurasi Database Connection](#konfigurasi-database-connection)
6. [Model Database](#model-database)
7. [Middleware dan Session](#middleware-dan-session)
8. [Implementasi Registrasi](#implementasi-registrasi)
9. [Implementasi Login](#implementasi-login)
10. [CRUD Books (dengan Foreign Key ke User)](#crud-books)
11. [Testing dan Deployment](#testing-dan-deployment)

## 1. Setup Awal

### Prasyarat
- Node.js versi 16 atau lebih baru
- MySQL Server
- Text Editor (VS Code recommended)

### Inisialisasi Project
```bash
mkdir bookstore-app
cd bookstore-app
npm init -y
```

### Install Dependencies
```bash
# Dependencies utama
npm install express ejs mysql2 bcryptjs express-session connect-mysql express-validator dotenv

# Development dependencies
npm install --save-dev nodemon
```

### Update package.json
```json
{
  "scripts": {
    "start": "node app.js",
    "dev": "nodemon app.js"
  }
}
```

## 2. Konfigurasi Database

### Buat Database MySQL
```sql
CREATE DATABASE bookstore_db;
USE bookstore_db;

-- Tabel Users
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    full_name VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Tabel Books (dengan foreign key ke users)
CREATE TABLE books (
    id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    author VARCHAR(100) NOT NULL,
    description TEXT,
    price DECIMAL(10,2) NOT NULL,
    stock INT DEFAULT 0,
    user_id INT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

## 3. Setup Express dan EJS

### File .env
```env
PORT=3000
DB_HOST=localhost
DB_USER=root
DB_PASSWORD=your_password
DB_NAME=bookstore_db
SESSION_SECRET=your_secret_key_here
```

### File app.js (Main Application)
```javascript
require('dotenv').config();
const express = require('express');
const session = require('express-session');
const MySQLStore = require('connect-mysql')(session);
const path = require('path');

const app = express();

// Import routes
const authRoutes = require('./routes/auth');
const bookRoutes = require('./routes/books');
const indexRoutes = require('./routes/index');

// Set EJS sebagai template engine
app.set('view engine', 'ejs');
app.set('views', path.join(__dirname, 'views'));

// Middleware
app.use(express.static('public'));
app.use(express.urlencoded({ extended: true }));
app.use(express.json());

// Session configuration
app.use(session({
    secret: process.env.SESSION_SECRET,
    resave: false,
    saveUninitialized: false,
    store: new MySQLStore({
        config: {
            user: process.env.DB_USER,
            password: process.env.DB_PASSWORD,
            database: process.env.DB_NAME,
            host: process.env.DB_HOST,
        }
    }),
    cookie: { maxAge: 60000 * 60 * 24 } // 24 hours
}));

// Global middleware untuk user session
app.use((req, res, next) => {
    res.locals.user = req.session.user || null;
    next();
});

// Routes
app.use('/', indexRoutes);
app.use('/auth', authRoutes);
app.use('/books', bookRoutes);

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
    console.log(`Server running on http://localhost:${PORT}`);
});
```

## 4. Struktur Folder

```
bookstore-app/
│
├── config/
│   └── database.js
├── middleware/
│   └── auth.js
├── models/
│   ├── User.js
│   └── Book.js
├── routes/
│   ├── index.js
│   ├── auth.js
│   └── books.js
├── views/
│   ├── layouts/
│   │   └── main.ejs
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
│       └── main.js
├── .env
├── app.js
└── package.json
```

## 5. Konfigurasi Database Connection

### File config/database.js
```javascript
const mysql = require('mysql2');

const pool = mysql.createPool({
    host: process.env.DB_HOST,
    user: process.env.DB_USER,
    password: process.env.DB_PASSWORD,
    database: process.env.DB_NAME,
    waitForConnections: true,
    connectionLimit: 10,
    queueLimit: 0
});

// Promisify untuk async/await
const promisePool = pool.promise();

module.exports = promisePool;
```

## 6. Model Database

### File models/User.js
```javascript
const db = require('../config/database');
const bcrypt = require('bcryptjs');

class User {
    static async create(userData) {
        const { username, email, password, full_name } = userData;
        const hashedPassword = await bcrypt.hash(password, 10);
        
        const [result] = await db.execute(
            'INSERT INTO users (username, email, password, full_name) VALUES (?, ?, ?, ?)',
            [username, email, hashedPassword, full_name]
        );
        
        return result.insertId;
    }

    static async findByEmail(email) {
        const [rows] = await db.execute(
            'SELECT * FROM users WHERE email = ?',
            [email]
        );
        return rows[0];
    }

    static async findByUsername(username) {
        const [rows] = await db.execute(
            'SELECT * FROM users WHERE username = ?',
            [username]
        );
        return rows[0];
    }

    static async findById(id) {
        const [rows] = await db.execute(
            'SELECT * FROM users WHERE id = ?',
            [id]
        );
        return rows[0];
    }

    static async verifyPassword(plainPassword, hashedPassword) {
        return await bcrypt.compare(plainPassword, hashedPassword);
    }
}

module.exports = User;
```

### File models/Book.js
```javascript
const db = require('../config/database');

class Book {
    static async create(bookData) {
        const { title, author, description, price, stock, user_id } = bookData;
        
        const [result] = await db.execute(
            'INSERT INTO books (title, author, description, price, stock, user_id) VALUES (?, ?, ?, ?, ?, ?)',
            [title, author, description, price, stock, user_id]
        );
        
        return result.insertId;
    }

    static async findAll() {
        const [rows] = await db.execute(`
            SELECT b.*, u.username as owner_name 
            FROM books b 
            JOIN users u ON b.user_id = u.id 
            ORDER BY b.created_at DESC
        `);
        return rows;
    }

    static async findByUserId(userId) {
        const [rows] = await db.execute(
            'SELECT * FROM books WHERE user_id = ? ORDER BY created_at DESC',
            [userId]
        );
        return rows;
    }

    static async findById(id) {
        const [rows] = await db.execute(`
            SELECT b.*, u.username as owner_name 
            FROM books b 
            JOIN users u ON b.user_id = u.id 
            WHERE b.id = ?
        `, [id]);
        return rows[0];
    }

    static async update(id, bookData) {
        const { title, author, description, price, stock } = bookData;
        
        const [result] = await db.execute(
            'UPDATE books SET title = ?, author = ?, description = ?, price = ?, stock = ? WHERE id = ?',
            [title, author, description, price, stock, id]
        );
        
        return result.affectedRows > 0;
    }

    static async delete(id) {
        const [result] = await db.execute('DELETE FROM books WHERE id = ?', [id]);
        return result.affectedRows > 0;
    }
}

module.exports = Book;
```

## 7. Middleware dan Session

### File middleware/auth.js
```javascript
const requireAuth = (req, res, next) => {
    if (!req.session.user) {
        return res.redirect('/auth/login');
    }
    next();
};

const requireGuest = (req, res, next) => {
    if (req.session.user) {
        return res.redirect('/');
    }
    next();
};

module.exports = { requireAuth, requireGuest };
```

## 8. Implementasi Registrasi

### File routes/auth.js
```javascript
const express = require('express');
const { body, validationResult } = require('express-validator');
const User = require('../models/User');
const { requireGuest, requireAuth } = require('../middleware/auth');

const router = express.Router();

// GET register page
router.get('/register', requireGuest, (req, res) => {
    res.render('auth/register', {
        title: 'Register',
        errors: [],
        oldInput: {}
    });
});

// POST register
router.post('/register', [
    body('username')
        .isLength({ min: 3 })
        .withMessage('Username minimal 3 karakter'),
    body('email')
        .isEmail()
        .withMessage('Email tidak valid'),
    body('password')
        .isLength({ min: 6 })
        .withMessage('Password minimal 6 karakter'),
    body('full_name')
        .notEmpty()
        .withMessage('Nama lengkap harus diisi')
], async (req, res) => {
    const errors = validationResult(req);
    
    if (!errors.isEmpty()) {
        return res.render('auth/register', {
            title: 'Register',
            errors: errors.array(),
            oldInput: req.body
        });
    }

    try {
        // Check if user already exists
        const existingUserByEmail = await User.findByEmail(req.body.email);
        const existingUserByUsername = await User.findByUsername(req.body.username);

        if (existingUserByEmail) {
            return res.render('auth/register', {
                title: 'Register',
                errors: [{ msg: 'Email sudah terdaftar' }],
                oldInput: req.body
            });
        }

        if (existingUserByUsername) {
            return res.render('auth/register', {
                title: 'Register',
                errors: [{ msg: 'Username sudah digunakan' }],
                oldInput: req.body
            });
        }

        // Create new user
        await User.create(req.body);
        res.redirect('/auth/login?success=1');

    } catch (error) {
        console.error(error);
        res.render('auth/register', {
            title: 'Register',
            errors: [{ msg: 'Terjadi kesalahan server' }],
            oldInput: req.body
        });
    }
});

// GET login page
router.get('/login', requireGuest, (req, res) => {
    const success = req.query.success === '1';
    res.render('auth/login', {
        title: 'Login',
        errors: [],
        success: success ? 'Registrasi berhasil! Silakan login.' : null
    });
});

// POST login
router.post('/login', [
    body('email').isEmail().withMessage('Email tidak valid'),
    body('password').notEmpty().withMessage('Password harus diisi')
], async (req, res) => {
    const errors = validationResult(req);
    
    if (!errors.isEmpty()) {
        return res.render('auth/login', {
            title: 'Login',
            errors: errors.array(),
            success: null
        });
    }

    try {
        const user = await User.findByEmail(req.body.email);
        
        if (!user) {
            return res.render('auth/login', {
                title: 'Login',
                errors: [{ msg: 'Email atau password salah' }],
                success: null
            });
        }

        const isValidPassword = await User.verifyPassword(req.body.password, user.password);
        
        if (!isValidPassword) {
            return res.render('auth/login', {
                title: 'Login',
                errors: [{ msg: 'Email atau password salah' }],
                success: null
            });
        }

        // Set session
        req.session.user = {
            id: user.id,
            username: user.username,
            email: user.email,
            full_name: user.full_name
        };

        res.redirect('/books');

    } catch (error) {
        console.error(error);
        res.render('auth/login', {
            title: 'Login',
            errors: [{ msg: 'Terjadi kesalahan server' }],
            success: null
        });
    }
});

// POST logout
router.post('/logout', requireAuth, (req, res) => {
    req.session.destroy((err) => {
        if (err) {
            console.error(err);
        }
        res.redirect('/');
    });
});

module.exports = router;
```

### File views/auth/register.ejs
```html
<%- include('../layouts/main', { title: title }) %>

<div class="container">
    <div class="row justify-content-center">
        <div class="col-md-6">
            <div class="card">
                <div class="card-header">
                    <h3>Registrasi</h3>
                </div>
                <div class="card-body">
                    <% if (errors.length > 0) { %>
                        <div class="alert alert-danger">
                            <ul class="mb-0">
                                <% errors.forEach(error => { %>
                                    <li><%= error.msg %></li>
                                <% }); %>
                            </ul>
                        </div>
                    <% } %>

                    <form method="POST" action="/auth/register">
                        <div class="mb-3">
                            <label for="username" class="form-label">Username</label>
                            <input type="text" class="form-control" id="username" name="username" 
                                   value="<%= oldInput.username || '' %>" required>
                        </div>

                        <div class="mb-3">
                            <label for="full_name" class="form-label">Nama Lengkap</label>
                            <input type="text" class="form-control" id="full_name" name="full_name" 
                                   value="<%= oldInput.full_name || '' %>" required>
                        </div>

                        <div class="mb-3">
                            <label for="email" class="form-label">Email</label>
                            <input type="email" class="form-control" id="email" name="email" 
                                   value="<%= oldInput.email || '' %>" required>
                        </div>

                        <div class="mb-3">
                            <label for="password" class="form-label">Password</label>
                            <input type="password" class="form-control" id="password" name="password" required>
                        </div>

                        <button type="submit" class="btn btn-primary w-100">Daftar</button>
                    </form>

                    <div class="text-center mt-3">
                        <p>Sudah punya akun? <a href="/auth/login">Login disini</a></p>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
```

### File views/auth/login.ejs
```html
<%- include('../layouts/main', { title: title }) %>

<div class="container">
    <div class="row justify-content-center">
        <div class="col-md-6">
            <div class="card">
                <div class="card-header">
                    <h3>Login</h3>
                </div>
                <div class="card-body">
                    <% if (success) { %>
                        <div class="alert alert-success">
                            <%= success %>
                        </div>
                    <% } %>

                    <% if (errors.length > 0) { %>
                        <div class="alert alert-danger">
                            <ul class="mb-0">
                                <% errors.forEach(error => { %>
                                    <li><%= error.msg %></li>
                                <% }); %>
                            </ul>
                        </div>
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
                        <p>Belum punya akun? <a href="/auth/register">Daftar disini</a></p>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
```

## 9. Implementasi Login

Login sudah diimplementasi di bagian routes/auth.js di atas.

## 10. CRUD Books (dengan Foreign Key ke User)

### File routes/books.js
```javascript
const express = require('express');
const { body, validationResult } = require('express-validator');
const Book = require('../models/Book');
const { requireAuth } = require('../middleware/auth');

const router = express.Router();

// Semua routes book memerlukan authentication
router.use(requireAuth);

// GET all books
router.get('/', async (req, res) => {
    try {
        const books = await Book.findAll();
        res.render('books/index', {
            title: 'Daftar Buku',
            books: books
        });
    } catch (error) {
        console.error(error);
        res.status(500).send('Server Error');
    }
});

// GET my books
router.get('/my-books', async (req, res) => {
    try {
        const books = await Book.findByUserId(req.session.user.id);
        res.render('books/index', {
            title: 'Buku Saya',
            books: books,
            isMyBooks: true
        });
    } catch (error) {
        console.error(error);
        res.status(500).send('Server Error');
    }
});

// GET create form
router.get('/create', (req, res) => {
    res.render('books/create', {
        title: 'Tambah Buku',
        errors: [],
        oldInput: {}
    });
});

// POST create book
router.post('/create', [
    body('title').notEmpty().withMessage('Judul harus diisi'),
    body('author').notEmpty().withMessage('Penulis harus diisi'),
    body('price').isFloat({ min: 0 }).withMessage('Harga harus berupa angka positif'),
    body('stock').isInt({ min: 0 }).withMessage('Stok harus berupa angka positif')
], async (req, res) => {
    const errors = validationResult(req);
    
    if (!errors.isEmpty()) {
        return res.render('books/create', {
            title: 'Tambah Buku',
            errors: errors.array(),
            oldInput: req.body
        });
    }

    try {
        const bookData = {
            ...req.body,
            user_id: req.session.user.id
        };
        
        await Book.create(bookData);
        res.redirect('/books/my-books');

    } catch (error) {
        console.error(error);
        res.render('books/create', {
            title: 'Tambah Buku',
            errors: [{ msg: 'Terjadi kesalahan server' }],
            oldInput: req.body
        });
    }
});

// GET single book
router.get('/:id', async (req, res) => {
    try {
        const book = await Book.findById(req.params.id);
        
        if (!book) {
            return res.status(404).send('Buku tidak ditemukan');
        }

        res.render('books/show', {
            title: book.title,
            book: book
        });
    } catch (error) {
        console.error(error);
        res.status(500).send('Server Error');
    }
});

// GET edit form
router.get('/:id/edit', async (req, res) => {
    try {
        const book = await Book.findById(req.params.id);
        
        if (!book) {
            return res.status(404).send('Buku tidak ditemukan');
        }

        // Check if user owns this book
        if (book.user_id !== req.session.user.id) {
            return res.status(403).send('Akses ditolak');
        }

        res.render('books/edit', {
            title: 'Edit Buku',
            book: book,
            errors: [],
            oldInput: book
        });
    } catch (error) {
        console.error(error);
        res.status(500).send('Server Error');
    }
});

// PUT update book
router.post('/:id/edit', [
    body('title').notEmpty().withMessage('Judul harus diisi'),
    body('author').notEmpty().withMessage('Penulis harus diisi'),
    body('price').isFloat({ min: 0 }).withMessage('Harga harus berupa angka positif'),
    body('stock').isInt({ min: 0 }).withMessage('Stok harus berupa angka positif')
], async (req, res) => {
    const errors = validationResult(req);
    
    try {
        const book = await Book.findById(req.params.id);
        
        if (!book) {
            return res.status(404).send('Buku tidak ditemukan');
        }

        // Check if user owns this book
        if (book.user_id !== req.session.user.id) {
            return res.status(403).send('Akses ditolak');
        }

        if (!errors.isEmpty()) {
            return res.render('books/edit', {
                title: 'Edit Buku',
                book: book,
                errors: errors.array(),
                oldInput: req.body
            });
        }

        await Book.update(req.params.id, req.body);
        res.redirect('/books/my-books');

    } catch (error) {
        console.error(error);
        res.status(500).send('Server Error');
    }
});

// DELETE book
router.post('/:id/delete', async (req, res) => {
    try {
        const book = await Book.findById(req.params.id);
        
        if (!book) {
            return res.status(404).send('Buku tidak ditemukan');
        }

        // Check if user owns this book
        if (book.user_id !== req.session.user.id) {
            return res.status(403).send('Akses ditolak');
        }

        await Book.delete(req.params.id);
        res.redirect('/books/my-books');

    } catch (error) {
        console.error(error);
        res.status(500).send('Server Error');
    }
});

module.exports = router;
```

### File views/books/index.ejs
```html
<%- include('../layouts/main', { title: title }) %>

<div class="container">
    <div class="d-flex justify-content-between align-items-center mb-4">
        <h1><%= title %></h1>
        <% if (typeof isMyBooks !== 'undefined' && isMyBooks) { %>
            <a href="/books/create" class="btn btn-primary">Tambah Buku</a>
        <% } %>
    </div>

    <% if (typeof isMyBooks === 'undefined') { %>
        <div class="mb-3">
            <a href="/books/my-books" class="btn btn-outline-primary">Buku Saya</a>
        </div>
    <% } else { %>
        <div class="mb-3">
            <a href="/books" class="btn btn-outline-secondary">Semua Buku</a>
        </div>
    <% } %>

    <% if (books.length === 0) { %>
        <div class="alert alert-info">
            <% if (typeof isMyBooks !== 'undefined' && isMyBooks) { %>
                Anda belum memiliki buku. <a href="/books/create">Tambah buku pertama</a>
            <% } else { %>
                Belum ada buku yang tersedia.
            <% } %>
        </div>
    <% } else { %>
        <div class="row">
            <% books.forEach(book => { %>
                <div class="col-md-4 mb-4">
                    <div class="card">
                        <div class="card-body">
                            <h5 class="card-title"><%= book.title %></h5>
                            <p class="card-text">
                                <strong>Penulis:</strong> <%= book.author %><br>
                                <strong>Harga:</strong> Rp <%= Number(book.price).toLocaleString('id-ID') %><br>
                                <strong>Stok:</strong> <%= book.stock %><br>
                                <% if (book.owner_name) { %>
                                    <strong>Pemilik:</strong> <%= book.owner_name %>
                                <% } %>
                            </p>
                            <div class="d-flex gap-2">
                                <a href="/books/<%= book.id %>" class="btn btn-info btn-sm">Detail</a>
                                <% if (typeof isMyBooks !== 'undefined' && isMyBooks) { %>
                                    <a href="/books/<%= book.id %>/edit" class="btn btn-warning btn-sm">Edit</a>
                                    <form method="POST" action="/books/<%= book.id %>/delete" class="d-inline">
                                        <button type="submit" class="btn btn-danger btn-sm" 
                                                onclick="return confirm('Yakin ingin menghapus buku ini?')">Hapus</button>
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
```

### File views/books/create.ejs
```html
<%- include('../layouts/main', { title: title }) %>

<div class="container">
    <div class="row justify-content-center">
        <div class="col-md-8">
            <div class="card">
                <div class="card-header">
                    <h3>Tambah Buku Baru</h3>
                </div>
                <div class="card-body">
                    <% if (errors.length > 0) { %>
                        <div class="alert alert-danger">
                            <ul class="mb-0">
                                <% errors.forEach(error => { %>
                                    <li><%= error.msg %></li>
                                <% }); %>
                            </ul>
                        </div>
                    <% } %>

                    <form method="POST" action="/books/create">
                        <div class="mb-3">
                            <label for="title" class="form-label">Judul Buku</label>
                            <input type="text" class="form-control" id="title" name="title" 
                                   value="<%= oldInput.title || '' %>" required>
                        </div>

                        <div class="mb-3">
                            <label for="author" class="form-label">Penulis</label>
                            <input type="text" class="form-control" id="author" name="author" 
                                   value="<%= oldInput.author || '' %>" required>
                        </div>

                        <div class="mb-3">
                            <label for="description" class="form-label">Deskripsi</label>
                            <textarea class="form-control" id="description" name="description" rows="4"><%= oldInput.description || '' %></textarea>
                        </div>

                        <div class="row">
                            <div class="col-md-6">
                                <div class="mb-3">
                                    <label for="price" class="form-label">Harga</label>
                                    <input type="number" step="0.01" class="form-control" id="price" name="price" 
                                           value="<%= oldInput.price || '' %>" required>
                                </div>
                            </div>
                            <div class="col-md-6">
                                <div class="mb-3">
                                    <label for="stock" class="form-label">Stok</label>
                                    <input type="number" class="form-control" id="stock" name="stock" 
                                           value="<%= oldInput.stock || '' %>" required>
                                </div>
                            </div>
                        </div>

                        <div class="d-flex gap-2">
                            <button type="submit" class="btn btn-primary">Simpan</button>
                            <a href="/books/my-books" class="btn btn-secondary">Batal</a>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
</div>
```

### File views/books/edit.ejs
```html
<%- include('../layouts/main', { title: title }) %>

<div class="container">
    <div class="row justify-content-center">
        <div class="col-md-8">
            <div class="card">
                <div class="card-header">
                    <h3>Edit Buku</h3>
                </div>
                <div class="card-body">
                    <% if (errors.length > 0) { %>
                        <div class="alert alert-danger">
                            <ul class="mb-0">
                                <% errors.forEach(error => { %>
                                    <li><%= error.msg %></li>
                                <% }); %>
                            </ul>
                        </div>
                    <% } %>

                    <form method="POST" action="/books/<%= book.id %>/edit">
                        <div class="mb-3">
                            <label for="title" class="form-label">Judul Buku</label>
                            <input type="text" class="form-control" id="title" name="title" 
                                   value="<%= oldInput.title || book.title %>" required>
                        </div>

                        <div class="mb-3">
                            <label for="author" class="form-label">Penulis</label>
                            <input type="text" class="form-control" id="author" name="author" 
                                   value="<%= oldInput.author || book.author %>" required>
                        </div>

                        <div class="mb-3">
                            <label for="description" class="form-label">Deskripsi</label>
                            <textarea class="form-control" id="description" name="description" rows="4"><%= oldInput.description || book.description || '' %></textarea>
                        </div>

                        <div class="row">
                            <div class="col-md-6">
                                <div class="mb-3">
                                    <label for="price" class="form-label">Harga</label>
                                    <input type="number" step="0.01" class="form-control" id="price" name="price" 
                                           value="<%= oldInput.price || book.price %>" required>
                                </div>
                            </div>
                            <div class="col-md-6">
                                <div class="mb-3">
                                    <label for="stock" class="form-label">Stok</label>
                                    <input type="number" class="form-control" id="stock" name="stock" 
                                           value="<%= oldInput.stock || book.stock %>" required>
                                </div>
                            </div>
                        </div>

                        <div class="d-flex gap-2">
                            <button type="submit" class="btn btn-primary">Update</button>
                            <a href="/books/my-books" class="btn btn-secondary">Batal</a>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
</div>
```

### File views/books/show.ejs
```html
<%- include('../layouts/main', { title: title }) %>

<div class="container">
    <div class="row justify-content-center">
        <div class="col-md-8">
            <div class="card">
                <div class="card-header d-flex justify-content-between align-items-center">
                    <h3><%= book.title %></h3>
                    <% if (user && book.user_id === user.id) { %>
                        <div>
                            <a href="/books/<%= book.id %>/edit" class="btn btn-warning btn-sm">Edit</a>
                            <form method="POST" action="/books/<%= book.id %>/delete" class="d-inline">
                                <button type="submit" class="btn btn-danger btn-sm" 
                                        onclick="return confirm('Yakin ingin menghapus buku ini?')">Hapus</button>
                            </form>
                        </div>
                    <% } %>
                </div>
                <div class="card-body">
                    <div class="row">
                        <div class="col-md-6">
                            <p><strong>Penulis:</strong> <%= book.author %></p>
                            <p><strong>Harga:</strong> Rp <%= Number(book.price).toLocaleString('id-ID') %></p>
                            <p><strong>Stok:</strong> <%= book.stock %></p>
                            <p><strong>Pemilik:</strong> <%= book.owner_name %></p>
                        </div>
                        <div class="col-md-6">
                            <p><strong>Ditambahkan:</strong> <%= new Date(book.created_at).toLocaleDateString('id-ID') %></p>
                            <% if (book.updated_at !== book.created_at) { %>
                                <p><strong>Diupdate:</strong> <%= new Date(book.updated_at).toLocaleDateString('id-ID') %></p>
                            <% } %>
                        </div>
                    </div>
                    
                    <% if (book.description) { %>
                        <div class="mt-3">
                            <h5>Deskripsi:</h5>
                            <p><%= book.description %></p>
                        </div>
                    <% } %>

                    <div class="mt-4">
                        <a href="/books" class="btn btn-secondary">Kembali ke Daftar Buku</a>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
```

## 11. Layout dan Routes Utama

### File views/layouts/main.ejs
```html
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title><%= title %> - Bookstore App</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="/css/style.css">
</head>
<body>
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
        <div class="container">
            <a class="navbar-brand" href="/">Bookstore App</a>
            
            <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav">
                <span class="navbar-toggler-icon"></span>
            </button>
            
            <div class="collapse navbar-collapse" id="navbarNav">
                <ul class="navbar-nav me-auto">
                    <li class="nav-item">
                        <a class="nav-link" href="/">Home</a>
                    </li>
                    <% if (user) { %>
                        <li class="nav-item">
                            <a class="nav-link" href="/books">Semua Buku</a>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link" href="/books/my-books">Buku Saya</a>
                        </li>
                    <% } %>
                </ul>
                
                <ul class="navbar-nav">
                    <% if (user) { %>
                        <li class="nav-item dropdown">
                            <a class="nav-link dropdown-toggle" href="#" id="navbarDropdown" role="button" data-bs-toggle="dropdown">
                                <%= user.full_name %>
                            </a>
                            <ul class="dropdown-menu">
                                <li>
                                    <form method="POST" action="/auth/logout" class="d-inline">
                                        <button type="submit" class="dropdown-item">Logout</button>
                                    </form>
                                </li>
                            </ul>
                        </li>
                    <% } else { %>
                        <li class="nav-item">
                            <a class="nav-link" href="/auth/login">Login</a>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link" href="/auth/register">Register</a>
                        </li>
                    <% } %>
                </ul>
            </div>
        </div>
    </nav>

    <main class="py-4">
        <%- body %>
    </main>

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/js/bootstrap.bundle.min.js"></script>
    <script src="/js/main.js"></script>
</body>
</html>
```

### File routes/index.js
```javascript
const express = require('express');
const router = express.Router();

router.get('/', (req, res) => {
    res.render('index', {
        title: 'Selamat Datang'
    });
});

module.exports = router;
```

### File views/index.ejs
```html
<%- include('layouts/main', { title: title }) %>

<div class="container">
    <div class="jumbotron bg-light p-5 rounded">
        <h1 class="display-4">Selamat Datang di Bookstore App</h1>
        <p class="lead">Platform manajemen buku sederhana menggunakan Node.js, EJS, dan MySQL.</p>
        
        <% if (!user) { %>
            <hr class="my-4">
            <p>Silakan login atau daftar untuk mulai mengelola koleksi buku Anda.</p>
            <div class="d-flex gap-2">
                <a class="btn btn-primary btn-lg" href="/auth/login" role="button">Login</a>
                <a class="btn btn-outline-primary btn-lg" href="/auth/register" role="button">Daftar</a>
            </div>
        <% } else { %>
            <hr class="my-4">
            <p>Halo, <%= user.full_name %>! Selamat datang kembali.</p>
            <div class="d-flex gap-2">
                <a class="btn btn-primary btn-lg" href="/books" role="button">Lihat Semua Buku</a>
                <a class="btn btn-outline-primary btn-lg" href="/books/my-books" role="button">Buku Saya</a>
            </div>
        <% } %>
    </div>
</div>
```

### File public/css/style.css
```css
body {
    background-color: #f8f9fa;
}

.jumbotron {
    margin-top: 2rem;
}

.card {
    box-shadow: 0 0.125rem 0.25rem rgba(0, 0, 0, 0.075);
    transition: box-shadow 0.15s ease-in-out;
}

.card:hover {
    box-shadow: 0 0.5rem 1rem rgba(0, 0, 0, 0.15);
}

.navbar-brand {
    font-weight: bold;
}

.alert {
    margin-bottom: 1rem;
}

.btn {
    border-radius: 0.375rem;
}

.form-control:focus {
    border-color: #0d6efd;
    box-shadow: 0 0 0 0.2rem rgba(13, 110, 253, 0.25);
}
```

### File public/js/main.js
```javascript
// Confirmation for delete actions
document.addEventListener('DOMContentLoaded', function() {
    const deleteButtons = document.querySelectorAll('button[onclick*="confirm"]');
    
    deleteButtons.forEach(button => {
        button.addEventListener('click', function(e) {
            if (!confirm('Yakin ingin menghapus data ini?')) {
                e.preventDefault();
            }
        });
    });
});

// Auto-hide alerts after 5 seconds
document.addEventListener('DOMContentLoaded', function() {
    const alerts = document.querySelectorAll('.alert-success');
    
    alerts.forEach(alert => {
        setTimeout(() => {
            alert.style.opacity = '0';
            setTimeout(() => {
                alert.remove();
            }, 300);
        }, 5000);
    });
});
```

## 12. Testing dan Deployment

### Testing Manual
1. Jalankan aplikasi:
```bash
npm run dev
```

2. Test flow lengkap:
   - Buka `http://localhost:3000`
   - Registrasi user baru
   - Login dengan user yang baru dibuat
   - Tambah beberapa buku
   - Edit buku
   - Hapus buku
   - Logout dan login kembali

### Environment Variables untuk Production
```env
NODE_ENV=production
PORT=3000
DB_HOST=your_production_db_host
DB_USER=your_production_db_user
DB_PASSWORD=your_production_db_password
DB_NAME=your_production_db_name
SESSION_SECRET=your_very_secure_session_secret
```

### Script Deployment (package.json)
```json
{
  "scripts": {
    "start": "node app.js",
    "dev": "nodemon app.js",
    "production": "NODE_ENV=production node app.js"
  }
}
```

## Kesimpulan

Panduan ini mencakup implementasi lengkap aplikasi web dengan Node.js, EJS, dan MySQL yang meliputi:

1. **Setup Project** - Konfigurasi awal dan dependencies
2. **Database Design** - Struktur tabel dengan foreign key relationship
3. **Authentication** - Registrasi dan login dengan session management
4. **CRUD Operations** - Full CRUD untuk entitas Books dengan ownership
5. **Security** - Password hashing, input validation, dan authorization
6. **UI/UX** - Responsive design dengan Bootstrap

**Fitur Utama:**
- User registration dan login
- Session management
- Book management (CRUD) dengan ownership
- Foreign key relationship antara User dan Book
- Input validation dan error handling
- Responsive UI

**Struktur Database:**
- Tabel `users` untuk authentication
- Tabel `books` dengan foreign key ke `users`
- Relationship one-to-many (satu user bisa punya banyak buku)

Aplikasi ini siap untuk pengembangan lebih lanjut dengan fitur tambahan seperti search, pagination, file upload, atau API endpoints.