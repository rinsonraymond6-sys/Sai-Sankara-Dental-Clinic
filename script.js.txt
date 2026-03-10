<script src="script.js"></script>
/**
 * Sai Sankara Dental Clinic — Main JavaScript
 * Handles: Navigation, Hero Scroll, Review Slider,
 *          Scroll Animations, Back-to-Top, Active Nav
 */

'use strict';

// ============================================================
// DOM READY
// ============================================================
document.addEventListener('DOMContentLoaded', () => {
  initAnnouncement();
  initNavigation();
  initScrollBehavior();
  initReviewSlider();
  initScrollAnimations();
  initBackToTop();
  initActiveNavHighlight();
  initCurrentDayHighlight();
});


// ============================================================
// 1. ANNOUNCEMENT BAR HEIGHT OFFSET
// ============================================================
function initAnnouncement() {
  const announcementBar = document.querySelector('.announcement-bar');
  const header = document.getElementById('site-header');

  if (!announcementBar || !header) return;

  function updateHeaderOffset() {
    const barHeight = announcementBar.offsetHeight;
    header.style.top = barHeight + 'px';

    // Also update hero padding-top if present
    const hero = document.querySelector('.hero');
    if (hero) {
      const headerH = parseInt(getComputedStyle(document.documentElement).getPropertyValue('--header-height')) || 76;
      hero.style.paddingTop = (barHeight + headerH + 48) + 'px';
    }
  }

  updateHeaderOffset();
  window.addEventListener('resize', updateHeaderOffset);
}


// ============================================================
// 2. NAVIGATION — MOBILE MENU & SCROLL TRANSPARENCY
// ============================================================
function initNavigation() {
  const hamburger   = document.getElementById('hamburger');
  const mobileMenu  = document.getElementById('mobile-menu');
  const mobileLinks = document.querySelectorAll('.mobile-link');
  const header      = document.getElementById('site-header');

  if (!hamburger || !mobileMenu) return;

  // Toggle mobile menu
  hamburger.addEventListener('click', () => {
    const isOpen = mobileMenu.classList.contains('open');
    if (isOpen) {
      closeMobileMenu();
    } else {
      openMobileMenu();
    }
  });

  // Close on link click
  mobileLinks.forEach(link => {
    link.addEventListener('click', () => {
      closeMobileMenu();
    });
  });

  // Close on ESC key
  document.addEventListener('keydown', (e) => {
    if (e.key === 'Escape') closeMobileMenu();
  });

  function openMobileMenu() {
    mobileMenu.classList.add('open');
    hamburger.classList.add('active');
    hamburger.setAttribute('aria-expanded', 'true');
    document.body.style.overflow = 'hidden';
  }

  function closeMobileMenu() {
    mobileMenu.classList.remove('open');
    hamburger.classList.remove('active');
    hamburger.setAttribute('aria-expanded', 'false');
    document.body.style.overflow = '';
  }

  // Header scroll state
  function handleHeaderScroll() {
    if (!header) return;
    if (window.scrollY > 50) {
      header.classList.add('scrolled');
    } else {
      header.classList.remove('scrolled');
    }
  }

  window.addEventListener('scroll', handleHeaderScroll, { passive: true });
  handleHeaderScroll(); // run on load
}


// ============================================================
// 3. SMOOTH SCROLL OFFSET (account for sticky header)
// ============================================================
function initScrollBehavior() {
  document.querySelectorAll('a[href^="#"]').forEach(link => {
    link.addEventListener('click', function (e) {
      const targetId = this.getAttribute('href');
      if (targetId === '#') return;

      const target = document.querySelector(targetId);
      if (!target) return;

      e.preventDefault();

      const header      = document.getElementById('site-header');
      const announcement = document.querySelector('.announcement-bar');
      const headerH     = header      ? header.offsetHeight      : 76;
      const announcementH = announcement ? announcement.offsetHeight : 36;
      const totalOffset = headerH + announcementH + 16;

      const targetTop = target.getBoundingClientRect().top + window.scrollY - totalOffset;

      window.scrollTo({
        top: targetTop,
        behavior: 'smooth'
      });
    });
  });
}


// ============================================================
// 4. REVIEWS SLIDER
// ============================================================
function initReviewSlider() {
  const slider    = document.getElementById('reviews-slider');
  const prevBtn   = document.getElementById('slider-prev');
  const nextBtn   = document.getElementById('slider-next');
  const dotsWrap  = document.getElementById('slider-dots');

  if (!slider || !prevBtn || !nextBtn) return;

  const cards        = slider.querySelectorAll('.review-card');
  const totalCards   = cards.length;
  let currentIndex   = 0;
  let cardsVisible   = getCardsVisible();
  let autoPlayTimer  = null;

  function getCardsVisible() {
    if (window.innerWidth <= 768) return 1;
    if (window.innerWidth <= 1024) return 2;
    return 3;
  }

  function maxIndex() {
    return Math.max(0, totalCards - cardsVisible);
  }

  // Build dots
  function buildDots() {
    dotsWrap.innerHTML = '';
    const count = maxIndex() + 1;
    for (let i = 0; i < count; i++) {
      const dot = document.createElement('button');
      dot.classList.add('slider-dot');
      dot.setAttribute('aria-label', `Go to review ${i + 1}`);
      if (i === 0) dot.classList.add('active');
      dot.addEventListener('click', () => goTo(i));
      dotsWrap.appendChild(dot);
    }
  }

  function updateDots() {
    const dots = dotsWrap.querySelectorAll('.slider-dot');
    dots.forEach((dot, i) => {
      dot.classList.toggle('active', i === currentIndex);
    });
  }

  function goTo(index) {
    cardsVisible    = getCardsVisible();
    currentIndex    = Math.max(0, Math.min(index, maxIndex()));
    const cardWidth = cards[0].offsetWidth + 24; // gap = 1.5rem = 24px
    slider.style.transform = `translateX(-${currentIndex * cardWidth}px)`;
    updateDots();
    updateButtons();
  }

  function updateButtons() {
    prevBtn.disabled = currentIndex === 0;
    nextBtn.disabled = currentIndex >= maxIndex();
    prevBtn.style.opacity = prevBtn.disabled ? '0.35' : '1';
    nextBtn.style.opacity = nextBtn.disabled ? '0.35' : '1';
  }

  function next() {
    if (currentIndex < maxIndex()) {
      goTo(currentIndex + 1);
    } else {
      goTo(0); // loop back
    }
  }

  function prev() {
    if (currentIndex > 0) {
      goTo(currentIndex - 1);
    } else {
      goTo(maxIndex()); // loop to end
    }
  }

  // Auto-play
  function startAutoPlay() {
    stopAutoPlay();
    autoPlayTimer = setInterval(next, 5000);
  }

  function stopAutoPlay() {
    if (autoPlayTimer) {
      clearInterval(autoPlayTimer);
      autoPlayTimer = null;
    }
  }

  // Event listeners
  prevBtn.addEventListener('click', () => { prev(); stopAutoPlay(); startAutoPlay(); });
  nextBtn.addEventListener('click', () => { next(); stopAutoPlay(); startAutoPlay(); });

  // Touch/swipe support
  let touchStartX = 0;
  let touchEndX   = 0;

  slider.addEventListener('touchstart', (e) => {
    touchStartX = e.changedTouches[0].clientX;
  }, { passive: true });

  slider.addEventListener('touchend', (e) => {
    touchEndX = e.changedTouches[0].clientX;
    const diff = touchStartX - touchEndX;
    if (Math.abs(diff) > 50) {
      if (diff > 0) next();
      else prev();
      stopAutoPlay();
      startAutoPlay();
    }
  }, { passive: true });

  // Pause on hover
  slider.addEventListener('mouseenter', stopAutoPlay);
  slider.addEventListener('mouseleave', startAutoPlay);

  // Resize handler
  let resizeTimer;
  window.addEventListener('resize', () => {
    clearTimeout(resizeTimer);
    resizeTimer = setTimeout(() => {
      const newVisible = getCardsVisible();
      if (newVisible !== cardsVisible) {
        cardsVisible = newVisible;
        buildDots();
      }
      goTo(currentIndex);
    }, 150);
  });

  // Initialise
  buildDots();
  goTo(0);
  startAutoPlay();
}


// ============================================================
// 5. SCROLL ANIMATIONS (Intersection Observer)
// ============================================================
function initScrollAnimations() {
  const elements = document.querySelectorAll('[data-animate]');
  if (!elements.length) return;

  const observer = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        const el    = entry.target;
        const delay = parseInt(el.getAttribute('data-delay') || '0');

        setTimeout(() => {
          el.classList.add('animated');
        }, delay);

        observer.unobserve(el);
      }
    });
  }, {
    threshold: 0.12,
    rootMargin: '0px 0px -40px 0px'
  });

  elements.forEach(el => observer.observe(el));
}


// ============================================================
// 6. BACK TO TOP BUTTON
// ============================================================
function initBackToTop() {
  const btn = document.getElementById('back-to-top');
  if (!btn) return;

  function toggleVisibility() {
    if (window.scrollY > 400) {
      btn.classList.add('visible');
    } else {
      btn.classList.remove('visible');
    }
  }

  btn.addEventListener('click', () => {
    window.scrollTo({ top: 0, behavior: 'smooth' });
  });

  window.addEventListener('scroll', toggleVisibility, { passive: true });
  toggleVisibility();
}


// ============================================================
// 7. ACTIVE NAV HIGHLIGHT (Scroll Spy)
// ============================================================
function initActiveNavHighlight() {
  const navLinks = document.querySelectorAll('.main-nav a');
  const sections = document.querySelectorAll('section[id], .hero[id]');

  if (!navLinks.length || !sections.length) return;

  const observer = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        const id = entry.target.getAttribute('id');
        navLinks.forEach(link => {
          link.classList.remove('active');
          if (link.getAttribute('href') === `#${id}`) {
            link.classList.add('active');
          }
        });
      }
    });
  }, {
    rootMargin: '-30% 0px -65% 0px'
  });

  sections.forEach(section => observer.observe(section));
}


// ============================================================
// 8. HIGHLIGHT TODAY'S CLINIC HOURS
// ============================================================
function initCurrentDayHighlight() {
  const hoursRows = document.querySelectorAll('.hours-row');
  if (!hoursRows.length) return;

  // Day index: 0=Sun, 1=Mon ... 6=Sat
  const dayNames  = ['Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday'];
  const today     = dayNames[new Date().getDay()];

  hoursRows.forEach(row => {
    const dayCell = row.querySelector('.day');
    if (dayCell && dayCell.textContent.trim() === today) {
      row.style.background  = 'rgba(201,168,76,0.08)';
      row.style.borderRadius = '6px';
      row.style.paddingLeft  = '0.5rem';
      row.style.paddingRight = '0.5rem';

      const dayEl = row.querySelector('.day');
      if (dayEl) {
        dayEl.style.color  = 'rgba(201,168,76,0.9)';
        dayEl.style.fontWeight = '800';
      }

      // Add "Today" badge
      const badge = document.createElement('span');
      badge.textContent = 'Today';
      badge.style.cssText = `
        font-size: 0.65rem;
        font-weight: 700;
        letter-spacing: 0.08em;
        text-transform: uppercase;
        background: rgba(201,168,76,0.2);
        color: rgba(224,192,112,0.9);
        border: 1px solid rgba(201,168,76,0.25);
        padding: 0.1rem 0.45rem;
        border-radius: 999px;
        margin-left: 0.5rem;
        vertical-align: middle;
      `;
      if (dayEl) dayEl.appendChild(badge);
    }
  });
}


// ============================================================
// 9. LAZY-LOAD IMAGES (progressive enhancement)
// ============================================================
(function initLazyImages() {
  if (!('IntersectionObserver' in window)) return;

  const images = document.querySelectorAll('img');

  const imgObserver = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        const img = entry.target;
        img.style.opacity = '0';
        img.style.transition = 'opacity 0.5s ease';

        img.addEventListener('load', () => {
          img.style.opacity = '1';
        }, { once: true });

        // If already loaded (cached)
        if (img.complete) img.style.opacity = '1';

        imgObserver.unobserve(img);
      }
    });
  }, { rootMargin: '200px' });

  images.forEach(img => imgObserver.observe(img));
})();
