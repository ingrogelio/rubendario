# Plan de Mejoras UX/UI - Proyecto Rubén Darío

## Resumen Ejecutivo
Implementar mejoras completas de interactividad, accesibilidad y experiencia de usuario en el sitio web interactivo sobre Rubén Darío.

---

## 1. Modo Oscuro/Claro

### Implementación
- Toggle switch en la navegación con icono de sol/luna
- Persistencia en localStorage
- Transición suave de 0.3s entre modos
- Variables CSS para colores que cambien dinámicamente

### Colores Modo Oscuro
```css
--bg-dark: #121212
--surface-dark: #1e1e1e
--text-dark: #e0e0e0
--primary-dark: #bb86fc
--secondary-dark: #03dac6
```

---

## 2. Micro-interacciones

### Timeline Items
```css
.timeline-item:hover .timeline-content {
  transform: translateY(-8px) scale(1.02);
  box-shadow: 0 20px 60px rgba(233,69,96,0.25);
}

.timeline-item::before {
  transition: all 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275);
}

.timeline-item:hover::before {
  transform: translateX(-50%) scale(1.5);
  box-shadow: 0 0 30px rgba(233,69,96,0.8);
}
```

### Botones con Efecto Ripple
```javascript
function createRipple(event) {
  const button = event.currentTarget;
  const circle = document.createElement('span');
  const diameter = Math.max(button.clientWidth, button.clientHeight);
  const radius = diameter / 2;
  
  circle.style.width = circle.style.height = `${diameter}px`;
  circle.style.left = `${event.clientX - button.getBoundingClientRect().left - radius}px`;
  circle.style.top = `${event.clientY - button.getBoundingClientRect().top - radius}px`;
  circle.classList.add('ripple');
  
  const ripple = button.getElementsByClassName('ripple')[0];
  if (ripple) ripple.remove();
  button.appendChild(circle);
}
```

### Animaciones de Entrada
```javascript
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      entry.target.classList.add('fade-in-up');
    }
  });
}, { threshold: 0.1 });
```

---

## 3. Accesibilidad Mejorada

### ARIA Labels
```html
<!-- Navigation -->
<nav role="navigation" aria-label="Navegación principal">
  <ul role="menubar">
    <li role="menuitem"><a href="#hero" aria-current="page">Inicio</a></li>
  </ul>
</nav>

<!-- Buttons -->
<button 
  aria-label="Navegar a página anterior" 
  aria-describedby="btnPrevDesc"
  id="btnPrev">
  ← Anterior
</button>
<span id="btnPrevDesc" class="sr-only">Ir a la página anterior del libro</span>
```

### Focus States
```css
*:focus-visible {
  outline: 3px solid var(--gold);
  outline-offset: 2px;
}

*:focus:not(:focus-visible) {
  outline: none;
}
```

### Prefers Reduced Motion
```css
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

---

## 4. Indicadores de Progreso

### Progreso de Lectura del Libro
```html
<div class="reading-progress" role="progressbar" aria-valuenow="25" aria-valuemin="0" aria-valuemax="100">
  <div class="reading-progress-bar" style="width: 25%"></div>
</div>
<span class="reading-progress-text">Página 3 de 12</span>
```

### Progreso de Scroll
```javascript
function updateReadingProgress() {
  const scrollHeight = document.documentElement.scrollHeight - window.innerHeight;
  const progress = (window.scrollY / scrollHeight) * 100;
  readingProgressBar.style.width = `${progress}%`;
}
```

---

## 5. Tooltips Informativos

```html
<div class="tooltip" data-tooltip="Ir al inicio">
  <button class="tooltip-trigger" aria-describedby="tooltip-1">🏠</button>
  <span id="tooltip-1" class="tooltip-content" role="tooltip">Ir al inicio</span>
</div>
```

```css
.tooltip {
  position: relative;
}

.tooltip-content {
  visibility: hidden;
  opacity: 0;
  transform: translateY(5px);
  transition: all 0.3s ease;
}

.tooltip:hover .tooltip-content,
.tooltip:focus-within .tooltip-content {
  visibility: visible;
  opacity: 1;
  transform: translateY(0);
}
```

---

## 6. Opciones de Tamaño de Fuente

```html
<div class="font-size-controls" role="group" aria-label="Tamaño de fuente">
  <button onclick="setFontSize('small')" aria-label="Fuente pequeña">A-</button>
  <button onclick="setFontSize('medium')" aria-label="Fuente media" class="active">A</button>
  <button onclick="setFontSize('large')" aria-label="Fuente grande">A+</button>
</div>
```

```javascript
function setFontSize(size) {
  const sizes = { small: '14px', medium: '16px', large: '18px' };
  document.documentElement.style.setProperty('--base-font-size', sizes[size]);
  localStorage.setItem('fontSize', size);
}
```

---

## 7. Búsqueda en Línea de Tiempo

```html
<div class="timeline-search">
  <input 
    type="text" 
    id="timelineSearch" 
    placeholder="Buscar eventos..." 
    aria-label="Buscar en línea de tiempo"
  >
  <div class="search-results" id="searchResults"></div>
</div>
```

```javascript
function searchTimeline(query) {
  const items = document.querySelectorAll('.timeline-item');
  items.forEach(item => {
    const text = item.textContent.toLowerCase();
    if (text.includes(query.toLowerCase())) {
      item.style.display = '';
      item.classList.add('highlight');
    } else {
      item.style.display = 'none';
      item.classList.remove('highlight');
    }
  });
}
```

---

## 8. Sistema de Marcadores

```html
<button 
  class="bookmark-btn" 
  onclick="toggleBookmark()" 
  aria-pressed="false"
  aria-label="Marcar página como favorita">
  <svg>⭐</svg>
</button>
<div class="bookmarks-panel" id="bookmarksPanel" hidden>
  <h3>Marcadores</h3>
  <ul id="bookmarksList"></ul>
</div>
```

```javascript
function toggleBookmark() {
  const bookmarks = JSON.parse(localStorage.getItem('bookmarks') || '[]');
  const currentPage = currentBookPage;
  
  if (bookmarks.includes(currentPage)) {
    const index = bookmarks.indexOf(currentPage);
    bookmarks.splice(index, 1);
  } else {
    bookmarks.push(currentPage);
  }
  
  localStorage.setItem('bookmarks', JSON.stringify(bookmarks));
  updateBookmarksUI();
}
```

---

## 9. Animaciones de Transición

```css
.book-page {
  transition: opacity 0.4s ease, transform 0.4s ease;
}

.book-page.fade-out {
  opacity: 0;
  transform: translateX(-20px);
}

.book-page.fade-in {
  opacity: 1;
  transform: translateX(0);
}
```

```javascript
function animatePageTransition(direction) {
  const currentPageEl = document.querySelector('.book-page.active');
  currentPageEl.classList.add('fade-out');
  
  setTimeout(() => {
    currentPageEl.classList.remove('active', 'fade-out');
    showBookPage(nextPage);
    const newPageEl = document.querySelector('.book-page.active');
    newPageEl.classList.add('fade-in');
    setTimeout(() => newPageEl.classList.remove('fade-in'), 400);
  }, 400);
}
```

---

## 10. Loading States para Imágenes

```html
<img 
  src="..." 
  alt="..." 
  loading="lazy"
  onload="this.classList.add('loaded')"
  onerror="this.classList.add('error')"
>
<div class="image-loader">
  <div class="spinner"></div>
</div>
```

```css
img {
  opacity: 0;
  transition: opacity 0.5s ease;
}

img.loaded {
  opacity: 1;
}

.image-loader {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 200px;
}

.spinner {
  border: 3px solid rgba(0,0,0,0.1);
  border-top-color: var(--secondary);
  border-radius: 50%;
  width: 40px;
  height: 40px;
  animation: spin 1s linear infinite;
}
```

---

## 11. Keyboard Shortcuts Expandidos

```javascript
const shortcutsHelp = {
  'ArrowRight': { action: 'nextBookPage', description: 'Página siguiente' },
  'ArrowLeft': { action: 'prevBookPage', description: 'Página anterior' },
  'ArrowDown': { action: 'scrollToSection', description: 'Sección siguiente' },
  'ArrowUp': { action: 'scrollToPrevSection', description: 'Sección anterior' },
  'Home': { action: 'goToStart', description: 'Ir al inicio' },
  'End': { action: 'goToEnd', description: 'Ir al final' },
  '?': { action: 'toggleHelp', description: 'Mostrar ayuda' },
  'd': { action: 'toggleDarkMode', description: 'Modo oscuro' },
  'f': { action: 'toggleSearch', description: 'Buscar' },
  'b': { action: 'toggleBookmarks', description: 'Marcadores' },
  '+': { action: 'increaseFont', description: 'Aumentar fuente' },
  '-': { action: 'decreaseFont', description: 'Disminuir fuente' }
};
```

---

## 12. Mobile Gestures

```javascript
let touchStartX = 0;
let touchEndX = 0;

document.addEventListener('touchstart', (e) => {
  touchStartX = e.changedTouches[0].screenX;
}, false);

document.addEventListener('touchend', (e) => {
  touchEndX = e.changedTouches[0].screenX;
  handleSwipe();
}, false);

function handleSwipe() {
  const swipeThreshold = 50;
  const diff = touchStartX - touchEndX;
  
  if (Math.abs(diff) > swipeThreshold) {
    if (diff > 0) {
      nextBookPage(); // Swipe left
    } else {
      prevBookPage(); // Swipe right
    }
  }
}
```

---

## Archivo de Plan

**Creado:** 2026-02-04  
**Modo:** Architect  
**Archivos principales:** index.html, portada.html, portada_1.html, libro_ruber_dario.html  
**Siguiente paso:** Cambiar a Code mode para implementar mejoras
