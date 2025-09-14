# Dokumentasi Lengkap Bootstrap 5 dan CSS Native

## Daftar Isi
1. [Pengenalan Bootstrap 5](#pengenalan-bootstrap-5)
2. [CSS Native Fundamentals](#css-native-fundamentals)
3. [Layout System](#layout-system)
4. [Typography](#typography)
5. [Colors](#colors)
6. [Spacing](#spacing)
7. [Components](#components)
8. [Utilities](#utilities)
9. [CSS Selectors](#css-selectors)
10. [CSS Properties](#css-properties)

---

## Pengenalan Bootstrap 5

Bootstrap 5 adalah framework CSS yang paling populer untuk mengembangkan website responsif dan mobile-first. Versi 5 menghilangkan dependensi jQuery dan menggunakan vanilla JavaScript.

### Instalasi Bootstrap 5

```html
<!-- CSS -->
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">

<!-- JavaScript Bundle -->
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
```

---

## CSS Native Fundamentals

### CSS Syntax
```css
selector {
    property: value;
}
```

### CSS Box Model
- **Content**: Konten aktual elemen
- **Padding**: Ruang di dalam elemen, di sekitar konten
- **Border**: Garis yang mengelilingi padding dan konten
- **Margin**: Ruang di luar elemen

---

## Layout System

### Bootstrap Grid System

#### Container
```html
<div class="container">       <!-- Fixed width container -->
<div class="container-fluid"> <!-- Full width container -->
<div class="container-sm">    <!-- Small breakpoint container -->
<div class="container-md">    <!-- Medium breakpoint container -->
<div class="container-lg">    <!-- Large breakpoint container -->
<div class="container-xl">    <!-- Extra large breakpoint container -->
<div class="container-xxl">   <!-- Extra extra large breakpoint container -->
```

#### Grid System
```html
<div class="container">
  <div class="row">
    <div class="col-12">12 kolom penuh</div>
    <div class="col-6">6 kolom</div>
    <div class="col-6">6 kolom</div>
  </div>
</div>
```

#### Responsive Columns
- `col-sm-*`: Small devices (≥576px)
- `col-md-*`: Medium devices (≥768px)
- `col-lg-*`: Large devices (≥992px)
- `col-xl-*`: Extra large devices (≥1200px)
- `col-xxl-*`: Extra extra large devices (≥1400px)

### CSS Native Layout

#### Display Properties
```css
.element {
    display: block;        /* Elemen blok */
    display: inline;       /* Elemen inline */
    display: inline-block; /* Kombinasi inline dan block */
    display: flex;         /* Flexbox */
    display: grid;         /* CSS Grid */
    display: none;         /* Menyembunyikan elemen */
}
```

#### Flexbox
```css
.flex-container {
    display: flex;
    flex-direction: row | column;
    justify-content: flex-start | center | flex-end | space-between | space-around;
    align-items: stretch | flex-start | center | flex-end;
    flex-wrap: nowrap | wrap;
}
```

#### CSS Grid
```css
.grid-container {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    grid-template-rows: auto;
    grid-gap: 20px;
}
```

---

## Typography

### Bootstrap Typography

#### Headings
```html
<h1 class="display-1">Display 1</h1>
<h1 class="display-2">Display 2</h1>
<h1 class="display-3">Display 3</h1>
<h1 class="display-4">Display 4</h1>
<h1 class="display-5">Display 5</h1>
<h1 class="display-6">Display 6</h1>
```

#### Text Utilities
- `text-start`: Teks rata kiri
- `text-center`: Teks rata tengah
- `text-end`: Teks rata kanan
- `text-uppercase`: Huruf kapital semua
- `text-lowercase`: Huruf kecil semua
- `text-capitalize`: Huruf kapital di awal kata

#### Font Weight
- `fw-bold`: Font tebal
- `fw-bolder`: Font lebih tebal
- `fw-normal`: Font normal
- `fw-light`: Font tipis
- `fw-lighter`: Font lebih tipis

### CSS Native Typography

```css
.text-styling {
    font-family: 'Arial', sans-serif;
    font-size: 16px;
    font-weight: bold | normal | 100-900;
    font-style: italic | normal;
    text-align: left | center | right | justify;
    text-decoration: none | underline | line-through;
    text-transform: uppercase | lowercase | capitalize;
    line-height: 1.5;
    letter-spacing: 2px;
    word-spacing: 5px;
}
```

---

## Colors

### Bootstrap Color System

#### Text Colors
- `text-primary`: Biru utama
- `text-secondary`: Abu-abu
- `text-success`: Hijau
- `text-danger`: Merah
- `text-warning`: Kuning
- `text-info`: Biru muda
- `text-light`: Putih/terang
- `text-dark`: Hitam/gelap
- `text-muted`: Abu-abu redup

#### Background Colors
- `bg-primary`, `bg-secondary`, `bg-success`, `bg-danger`
- `bg-warning`, `bg-info`, `bg-light`, `bg-dark`

### CSS Native Colors

```css
.color-examples {
    /* Named colors */
    color: red;
    color: blue;
    
    /* Hex colors */
    color: #ff0000;
    color: #0066cc;
    
    /* RGB */
    color: rgb(255, 0, 0);
    
    /* RGBA (dengan transparansi) */
    color: rgba(255, 0, 0, 0.5);
    
    /* HSL */
    color: hsl(0, 100%, 50%);
    
    /* HSLA */
    color: hsla(0, 100%, 50%, 0.5);
}
```

---

## Spacing

### Bootstrap Spacing

Format: `{property}{sides}-{size}` atau `{property}{sides}-{breakpoint}-{size}`

#### Properties
- `m`: margin
- `p`: padding

#### Sides
- `t`: top
- `b`: bottom
- `s`: start (left)
- `e`: end (right)
- `x`: horizontal (left dan right)
- `y`: vertical (top dan bottom)
- tanpa huruf: semua sisi

#### Sizes
- `0`: 0
- `1`: 0.25rem
- `2`: 0.5rem
- `3`: 1rem
- `4`: 1.5rem
- `5`: 3rem
- `auto`: auto

```html
<div class="m-3">Margin 1rem di semua sisi</div>
<div class="px-4">Padding horizontal 1.5rem</div>
<div class="mt-5">Margin top 3rem</div>
```

### CSS Native Spacing

```css
.spacing-examples {
    /* Margin */
    margin: 10px;                    /* Semua sisi */
    margin: 10px 20px;              /* Vertikal horizontal */
    margin: 10px 20px 15px 25px;    /* Top right bottom left */
    margin-top: 10px;
    margin-right: 20px;
    margin-bottom: 15px;
    margin-left: 25px;
    
    /* Padding - sama seperti margin */
    padding: 10px;
    padding-top: 10px;
}
```

---

## Components

### Bootstrap Components

#### Buttons
```html
<button class="btn btn-primary">Primary</button>
<button class="btn btn-secondary">Secondary</button>
<button class="btn btn-success">Success</button>
<button class="btn btn-danger">Danger</button>
<button class="btn btn-warning">Warning</button>
<button class="btn btn-info">Info</button>
<button class="btn btn-light">Light</button>
<button class="btn btn-dark">Dark</button>

<!-- Button Sizes -->
<button class="btn btn-primary btn-lg">Large</button>
<button class="btn btn-primary">Default</button>
<button class="btn btn-primary btn-sm">Small</button>

<!-- Button States -->
<button class="btn btn-primary" disabled>Disabled</button>
<button class="btn btn-outline-primary">Outline</button>
```

#### Cards
```html
<div class="card" style="width: 18rem;">
  <img src="..." class="card-img-top" alt="...">
  <div class="card-body">
    <h5 class="card-title">Card title</h5>
    <p class="card-text">Some quick example text.</p>
    <a href="#" class="btn btn-primary">Go somewhere</a>
  </div>
</div>
```

#### Navigation
```html
<nav class="navbar navbar-expand-lg navbar-light bg-light">
  <div class="container-fluid">
    <a class="navbar-brand" href="#">Navbar</a>
    <button class="navbar-toggler" type="button" data-bs-toggle="collapse">
      <span class="navbar-toggler-icon"></span>
    </button>
    <div class="collapse navbar-collapse">
      <ul class="navbar-nav">
        <li class="nav-item">
          <a class="nav-link active" href="#">Home</a>
        </li>
        <li class="nav-item">
          <a class="nav-link" href="#">Features</a>
        </li>
      </ul>
    </div>
  </div>
</nav>
```

#### Forms
```html
<form>
  <div class="mb-3">
    <label for="email" class="form-label">Email</label>
    <input type="email" class="form-control" id="email">
  </div>
  <div class="mb-3">
    <label for="password" class="form-label">Password</label>
    <input type="password" class="form-control" id="password">
  </div>
  <div class="mb-3 form-check">
    <input type="checkbox" class="form-check-input" id="check">
    <label class="form-check-label" for="check">Check me out</label>
  </div>
  <button type="submit" class="btn btn-primary">Submit</button>
</form>
```

#### Modal
```html
<!-- Button trigger modal -->
<button type="button" class="btn btn-primary" data-bs-toggle="modal" data-bs-target="#exampleModal">
  Launch demo modal
</button>

<!-- Modal -->
<div class="modal fade" id="exampleModal" tabindex="-1">
  <div class="modal-dialog">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title">Modal title</h5>
        <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
      </div>
      <div class="modal-body">
        Modal body text goes here.
      </div>
      <div class="modal-footer">
        <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Close</button>
        <button type="button" class="btn btn-primary">Save changes</button>
      </div>
    </div>
  </div>
</div>
```

---

## Utilities

### Bootstrap Utilities

#### Display
- `d-none`: Menyembunyikan elemen
- `d-block`: Display block
- `d-inline`: Display inline
- `d-inline-block`: Display inline-block
- `d-flex`: Display flexbox
- `d-grid`: Display grid

#### Position
- `position-static`: Posisi static
- `position-relative`: Posisi relative
- `position-absolute`: Posisi absolute
- `position-fixed`: Posisi fixed
- `position-sticky`: Posisi sticky

#### Flexbox Utilities
- `justify-content-start`, `justify-content-center`, `justify-content-end`
- `justify-content-between`, `justify-content-around`, `justify-content-evenly`
- `align-items-start`, `align-items-center`, `align-items-end`
- `flex-row`, `flex-column`
- `flex-wrap`, `flex-nowrap`

#### Border
- `border`: Border semua sisi
- `border-top`, `border-end`, `border-bottom`, `border-start`
- `border-0`: Menghilangkan border
- `rounded`: Border radius
- `rounded-circle`: Border radius 50%
- `rounded-pill`: Border radius penuh

#### Shadow
- `shadow-none`: Tanpa shadow
- `shadow-sm`: Shadow kecil
- `shadow`: Shadow default
- `shadow-lg`: Shadow besar

---

## CSS Selectors

### Basic Selectors
```css
/* Element selector */
h1 { color: blue; }

/* Class selector */
.my-class { color: red; }

/* ID selector */
#my-id { color: green; }

/* Universal selector */
* { margin: 0; }

/* Attribute selector */
[type="text"] { border: 1px solid black; }
```

### Combination Selectors
```css
/* Descendant selector */
div p { color: blue; }

/* Child selector */
div > p { color: red; }

/* Adjacent sibling */
h1 + p { margin-top: 0; }

/* General sibling */
h1 ~ p { color: gray; }
```

### Pseudo-classes
```css
/* Link states */
a:link { color: blue; }
a:visited { color: purple; }
a:hover { color: red; }
a:active { color: orange; }

/* Form states */
input:focus { border-color: blue; }
input:disabled { opacity: 0.5; }

/* Structural pseudo-classes */
p:first-child { margin-top: 0; }
p:last-child { margin-bottom: 0; }
tr:nth-child(odd) { background-color: #f2f2f2; }
```

### Pseudo-elements
```css
/* Before and after */
p::before { content: "🔸 "; }
p::after { content: " ✓"; }

/* First letter and line */
p::first-letter { font-size: 2em; }
p::first-line { font-weight: bold; }
```

---

## CSS Properties

### Layout Properties
```css
.layout-properties {
    /* Width and Height */
    width: 100px | 50% | auto;
    height: 200px | 100vh | auto;
    max-width: 500px;
    min-width: 200px;
    max-height: 300px;
    min-height: 100px;
    
    /* Position */
    position: static | relative | absolute | fixed | sticky;
    top: 10px;
    right: 20px;
    bottom: 30px;
    left: 40px;
    z-index: 10;
    
    /* Float */
    float: left | right | none;
    clear: left | right | both | none;
    
    /* Overflow */
    overflow: visible | hidden | scroll | auto;
    overflow-x: hidden;
    overflow-y: scroll;
}
```

### Visual Properties
```css
.visual-properties {
    /* Background */
    background-color: #ff0000;
    background-image: url('image.jpg');
    background-repeat: no-repeat | repeat | repeat-x | repeat-y;
    background-position: center | top left | 50% 50%;
    background-size: cover | contain | 100px 200px;
    background-attachment: scroll | fixed;
    
    /* Border */
    border: 1px solid black;
    border-width: 1px;
    border-style: solid | dashed | dotted | double;
    border-color: red;
    border-radius: 5px;
    
    /* Shadow */
    box-shadow: 2px 2px 5px rgba(0,0,0,0.3);
    text-shadow: 1px 1px 2px black;
    
    /* Opacity and Visibility */
    opacity: 0.5;
    visibility: visible | hidden;
}
```

### Animation Properties
```css
.animation-properties {
    /* Transition */
    transition: all 0.3s ease;
    transition-property: width, height;
    transition-duration: 0.5s;
    transition-timing-function: ease | linear | ease-in | ease-out;
    transition-delay: 0.2s;
    
    /* Transform */
    transform: translate(50px, 100px);
    transform: rotate(45deg);
    transform: scale(1.5);
    transform: skew(20deg, 10deg);
    transform-origin: center | top left | 50% 50%;
    
    /* Animation */
    animation: slide-in 2s ease-in-out;
    animation-name: slide-in;
    animation-duration: 2s;
    animation-timing-function: ease-in-out;
    animation-delay: 1s;
    animation-iteration-count: infinite | 3;
    animation-direction: normal | reverse | alternate;
    animation-fill-mode: forwards | backwards | both;
}

@keyframes slide-in {
    from { transform: translateX(-100%); }
    to { transform: translateX(0); }
}
```

---

## Responsive Design

### Bootstrap Breakpoints
- `xs`: <576px
- `sm`: ≥576px
- `md`: ≥768px
- `lg`: ≥992px
- `xl`: ≥1200px
- `xxl`: ≥1400px

### CSS Media Queries
```css
/* Mobile First Approach */
.element {
    width: 100%;
}

@media (min-width: 576px) {
    .element {
        width: 50%;
    }
}

@media (min-width: 768px) {
    .element {
        width: 33.333%;
    }
}

@media (min-width: 992px) {
    .element {
        width: 25%;
    }
}

/* Desktop First Approach */
.element {
    width: 25%;
}

@media (max-width: 991.98px) {
    .element {
        width: 33.333%;
    }
}

@media (max-width: 767.98px) {
    .element {
        width: 50%;
    }
}

@media (max-width: 575.98px) {
    .element {
        width: 100%;
    }
}
```

---

## Best Practices

### Bootstrap Best Practices
1. **Gunakan Grid System**: Manfaatkan sistem grid untuk layout responsif
2. **Utility Classes**: Gunakan utility classes untuk styling cepat
3. **Customization**: Override variabel SCSS untuk customization
4. **Semantic HTML**: Gunakan elemen HTML yang semantik
5. **Accessibility**: Perhatikan accessibility dengan ARIA attributes

### CSS Native Best Practices
1. **Organization**: Organisir CSS dengan metodologi seperti BEM
2. **Reusability**: Buat classes yang dapat digunakan kembali
3. **Performance**: Minimalkan CSS dan gunakan shorthand properties
4. **Maintainability**: Gunakan naming convention yang konsisten
5. **Browser Compatibility**: Test di berbagai browser

### Example BEM Methodology
```css
/* Block */
.card { }

/* Element */
.card__title { }
.card__content { }
.card__button { }

/* Modifier */
.card--featured { }
.card__button--primary { }
```

---

## Troubleshooting

### Common Bootstrap Issues
1. **CSS tidak load**: Periksa CDN link atau file path
2. **JavaScript tidak bekerja**: Pastikan Bootstrap JS sudah di-include
3. **Grid tidak responsif**: Periksa viewport meta tag
4. **Custom CSS tertimpa**: Gunakan specificity yang lebih tinggi

### Common CSS Issues
1. **Margin collapse**: Gunakan padding atau border untuk mencegah
2. **Z-index tidak bekerja**: Pastikan elemen memiliki position selain static
3. **Float clearing**: Gunakan clearfix untuk clear float
4. **Flexbox tidak support**: Gunakan fallback untuk browser lama

---

## Kesimpulan

Dokumentasi ini mencakup semua aspek penting dari Bootstrap 5 dan CSS Native. Bootstrap 5 menyediakan framework yang powerful untuk development yang cepat, sementara CSS Native memberikan kontrol penuh atas styling. Kombinasi keduanya memungkinkan Anda untuk membuat website yang responsive, accessible, dan maintainable.

Ingatlah untuk selalu:
- Test di berbagai device dan browser
- Mengoptimalkan performance
- Mempertahankan code quality
- Mengikuti best practices
- Selalu update dengan versi terbaru

Happy coding! 🚀