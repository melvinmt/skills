# Animations & Micro-Interactions

Patterns for adding polish to Astro sites. Applicable to any static site generator with minor adjustments.

## CSS-Only Entry Animations

### Fade-in on Load

```css
@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(24px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.hero-title {
  animation: fadeInUp 0.8s cubic-bezier(0.16, 1, 0.3, 1) both;
}

.hero-subtitle {
  animation: fadeInUp 0.8s cubic-bezier(0.16, 1, 0.3, 1) 0.15s both;
}

.hero-cta {
  animation: fadeInUp 0.8s cubic-bezier(0.16, 1, 0.3, 1) 0.3s both;
}
```

Use staggered delays (0.15s increments) for sequential elements.

### Scale-in for Cards

```css
@keyframes scaleIn {
  from {
    opacity: 0;
    transform: scale(0.95);
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}
```

---

## Scroll-Triggered Animations

### IntersectionObserver Pattern

```astro
---
// In your Astro component
---
<div class="animate-on-scroll">
  <slot />
</div>

<script>
  const observer = new IntersectionObserver(
    (entries) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) {
          entry.target.classList.add('visible');
          observer.unobserve(entry.target);
        }
      });
    },
    { threshold: 0.15, rootMargin: '0px 0px -50px 0px' }
  );

  document.querySelectorAll('.animate-on-scroll').forEach((el) => {
    observer.observe(el);
  });
</script>

<style>
  .animate-on-scroll {
    opacity: 0;
    transform: translateY(20px);
    transition: opacity 0.6s cubic-bezier(0.16, 1, 0.3, 1),
                transform 0.6s cubic-bezier(0.16, 1, 0.3, 1);
  }

  .animate-on-scroll.visible {
    opacity: 1;
    transform: translateY(0);
  }
</style>
```

### Staggered Children

```astro
<script>
  const staggerObserver = new IntersectionObserver(
    (entries) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) {
          const children = entry.target.querySelectorAll('.stagger-item');
          children.forEach((child, index) => {
            child.style.transitionDelay = `${index * 0.1}s`;
            child.classList.add('visible');
          });
          staggerObserver.unobserve(entry.target);
        }
      });
    },
    { threshold: 0.1 }
  );

  document.querySelectorAll('.stagger-container').forEach((el) => {
    staggerObserver.observe(el);
  });
</script>
```

Use this for feature grids, pricing cards, team members, etc.

---

## Hover & Interactive States

### Button Hover

```css
.btn-primary {
  transition: transform 0.2s ease, box-shadow 0.2s ease, background-color 0.2s ease;
}

.btn-primary:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 25px -5px rgba(0, 0, 0, 0.2);
}

.btn-primary:active {
  transform: translateY(0);
  box-shadow: 0 2px 10px -3px rgba(0, 0, 0, 0.2);
}
```

### Card Hover

```css
.card {
  transition: transform 0.3s cubic-bezier(0.16, 1, 0.3, 1),
              box-shadow 0.3s cubic-bezier(0.16, 1, 0.3, 1);
}

.card:hover {
  transform: translateY(-4px);
  box-shadow: 0 20px 40px -15px rgba(0, 0, 0, 0.15);
}
```

### Link Underline Animation

```css
.animated-link {
  position: relative;
  text-decoration: none;
}

.animated-link::after {
  content: '';
  position: absolute;
  bottom: -2px;
  left: 0;
  width: 0;
  height: 2px;
  background: currentColor;
  transition: width 0.3s cubic-bezier(0.16, 1, 0.3, 1);
}

.animated-link:hover::after {
  width: 100%;
}
```

---

## Navigation

### Sticky Header with Blur

```css
.navbar {
  position: sticky;
  top: 0;
  z-index: 50;
  backdrop-filter: blur(12px);
  background: rgba(255, 255, 255, 0.8);
  transition: background-color 0.3s ease, box-shadow 0.3s ease;
}

.navbar.scrolled {
  background: rgba(255, 255, 255, 0.95);
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}
```

```javascript
window.addEventListener('scroll', () => {
  document.querySelector('.navbar')
    .classList.toggle('scrolled', window.scrollY > 20);
});
```

### Smooth Scroll

```css
html {
  scroll-behavior: smooth;
}
```

---

## Parallax (Lightweight)

```css
.parallax-bg {
  background-attachment: fixed;
  background-position: center;
  background-size: cover;
}
```

For more control, use a simple JS parallax:

```javascript
window.addEventListener('scroll', () => {
  const scrolled = window.scrollY;
  document.querySelector('.parallax-element').style.transform =
    `translateY(${scrolled * 0.3}px)`;
});
```

---

## Reduced Motion

Always respect the user's preference:

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

Place this at the end of your global CSS file.

---

## Easing Reference

| Name | Value | Feel |
|------|-------|------|
| Smooth out | `cubic-bezier(0.16, 1, 0.3, 1)` | Snappy start, gentle end |
| Bounce | `cubic-bezier(0.34, 1.56, 0.64, 1)` | Slight overshoot |
| Gentle | `cubic-bezier(0.4, 0, 0.2, 1)` | Material Design standard |
| Sharp | `cubic-bezier(0.4, 0, 0.6, 1)` | Quick, decisive |

Default to "Smooth out" for most animations. Use "Bounce" sparingly for playful elements.
