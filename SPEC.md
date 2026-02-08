# SPEC: Valentine's Day Proposal Website

## Intent Crystal

```
surface_request: "cute simple valentines day website asking her to be my valentine"
underlying_need: romantic gesture that's personal, playful, and shareable via link
success_looks_like: she opens a link on her phone, laughs at the No button, says yes, sees photos of you two, gets excited about the date plan
constraints: simple (no framework), mobile-first, deployable as static site, ready before Feb 14
```

## Architecture

**Single-page app** — one `index.html` with 3 screens toggled via JS. No routing, no build step, no framework.

```
valentine/
├── index.html          ← single file: HTML + embedded <style> + <script>
├── images/             ← placeholder photos (swap in real ones later)
│   ├── photo1.jpg
│   ├── photo2.jpg
│   └── ... (5-8 slots)
├── SPEC.md
└── README.md           ← (optional: deployment instructions)
```

### External Dependencies (CDN only)
- **canvas-confetti** (~17KB) — heart-shaped confetti explosion
- **Google Fonts** — Pacifico (headers) + Quicksand (body)
- No other libraries. Everything else is vanilla CSS/JS.

## Screens

### Screen 1: The Proposal

**Layout**: Centered card on a romantic gradient background with floating hearts

| Element | Behavior |
|---------|----------|
| **Floating hearts background** | 15-20 CSS-animated heart elements, staggered delays, float upward continuously |
| **Typewriter text** | "Will you be my Valentine?" types out letter by letter (CSS `steps()` animation) |
| **Yes button** | Pink/rose, rounded. Grows via `transform: scale()` each time No is clicked (+0.15 per click). Also increases font size. |
| **No button** | Starts same size as Yes. Each click: (1) text changes to next phrase, (2) button shrinks by 0.1 scale, (3) repositions to random spot within viewport. After all phrases exhausted, button fades out entirely. |

**No Button Phrase Progression** (12 escalating messages):
1. "No"
2. "Are you sure?"
3. "Really sure?"
4. "Pookie please?"
5. "Don't do this to me :("
6. "I'm gonna cry..."
7. "You're breaking my heart!"
8. "I'll be so sad..."
9. "Pretty please?"
10. "Just say yes!"
11. "I'll buy you food..."
12. "Fine, I'll cry now"

**On "Yes" click**:
- Heart confetti explosion (canvas-confetti with heart shapes, pink/red colors)
- Brief celebration animation (1-2 seconds)
- Smooth fade transition to Screen 2

**Easter egg**: If she clicks Yes WITHOUT ever clicking No → show special message: "I knew you couldn't resist!" before transitioning.

### Screen 2: Celebration + Photos

**Layout**: Full-width celebration screen

| Element | Behavior |
|---------|----------|
| **"YAY!" header** | Large animated text, fades/scales in with bounce |
| **Personalized message** | Short sweet text below (e.g., "You just made me the happiest person!") — placeholder, user customizes |
| **"Reasons I love you"** | Scrollable list of 5-10 short reasons, each fades in on scroll. Placeholder content user swaps. |
| **Photo gallery** | CSS grid of polaroid-style photos. White border, slight random rotation (-5 to 5deg), caption below each. 5-8 placeholder slots. |
| **Photo lightbox** | Click any photo to view fullscreen with dark overlay. Pure CSS/JS, no library. |
| **"See our plans" button** | Transitions to Screen 3 |

**Polaroid photo styling**:
- White border (thicker on bottom for caption)
- Slight rotation via `transform: rotate(Xdeg)` with random -5 to 5deg
- Subtle box-shadow
- Hover: straighten rotation + slight scale up
- Caption text below in handwriting-style font

### Screen 3: Valentine's Day Itinerary

**Layout**: Vertical timeline with alternating left/right items

| Element | Behavior |
|---------|----------|
| **Header** | "Our Valentine's Day" or similar |
| **Timeline** | Vertical center line with items alternating sides |
| **Each item** | Time badge (e.g., "6:00 PM"), activity emoji, activity name, short description |
| **Scroll animation** | Items fade in as user scrolls (IntersectionObserver) |

**Itinerary structure** (user will provide real content):
```
[time] [emoji] [activity]
[short description]
```

Template with 4-6 placeholder items. User fills in their actual plans.

## Design System

### Color Palette
- **Primary**: `#ff6b8a` (soft rose/pink)
- **Secondary**: `#ff2d55` (deeper red/pink)
- **Background**: `#fff0f3` (very light pink) → `#ffe0e6` (gradient)
- **Text**: `#4a2040` (dark plum)
- **White**: `#ffffff` (polaroid borders, card backgrounds)
- **Accent**: `#ffd700` (gold, for special moments)

### Typography
- **Headers**: `'Pacifico', cursive` (Google Fonts)
- **Body**: `'Quicksand', sans-serif` (Google Fonts)
- **Captions**: `'Dancing Script', cursive` (for photo captions)

### Spacing & Layout
- Mobile-first: base design at 375px width
- Max content width: 600px centered
- Generous padding (20-40px)
- Rounded corners everywhere (12-20px border-radius)

## Technical Implementation Notes

### Floating Hearts (CSS-only)
- 15-20 `<div class="heart">` elements absolutely positioned
- CSS `@keyframes float` — translateY from bottom to top, slight horizontal sway
- Staggered `animation-delay` (0s to 8s range)
- `animation-duration` varies (6s to 12s)
- Hearts created as CSS shapes (::before + ::after circles + rotated square)
- Various sizes (10px to 30px) and opacities (0.3 to 0.8)

### Typewriter Effect (CSS-only)
```css
animation: typing 3s steps(25), blink-caret 0.75s step-end infinite;
```
- `overflow: hidden; white-space: nowrap; border-right: 3px solid;`
- Width animates from 0 to 100%

### No Button Repositioning (mobile-friendly)
- On click (not hover — touch devices don't have hover)
- `position: absolute` within a container
- Random x/y within viewport bounds (clamped to avoid off-screen)
- CSS `transition: all 0.3s ease` for smooth movement
- After phrase 12: `opacity: 0; pointer-events: none;`

### Yes Button Growth
```javascript
let noClickCount = 0;
const yesBtn = document.getElementById('yes-btn');

function handleNoClick() {
  noClickCount++;
  const scale = 1 + (noClickCount * 0.15);
  yesBtn.style.transform = `scale(${scale})`;
}
```

### Confetti (canvas-confetti CDN)
```javascript
// Heart-shaped confetti burst on Yes click
confetti({
  particleCount: 150,
  spread: 80,
  origin: { y: 0.6 },
  colors: ['#ff6b8a', '#ff2d55', '#ff69b4', '#ffd700'],
  shapes: ['heart'],
  scalar: 2
});
```

### Screen Transitions
- All 3 screens in DOM, only one visible at a time
- `display: none` → `display: flex` with opacity transition
- 0.5s fade transition between screens

### Scroll-triggered Animations (Screen 3)
```javascript
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      entry.target.classList.add('visible');
    }
  });
}, { threshold: 0.2 });
```

## Responsiveness

- **Mobile (375px)**: Single column, full-width cards, stacked timeline
- **Tablet (768px)**: Slightly wider cards, 2-column photo grid
- **Desktop (1024px+)**: Centered max-width container, alternating timeline

## Done Criteria

1. All 3 screens render correctly on mobile Safari + Chrome
2. No button phrase progression works through all 12 messages
3. Yes button grows visibly with each No click
4. No button repositions on tap (mobile) and shrinks
5. Heart confetti fires on Yes click
6. Easter egg message shows when Yes clicked without any No clicks
7. "Reasons I love you" section displays with fade-in
8. Photo gallery renders with polaroid styling and lightbox
9. Itinerary timeline renders with scroll animations
10. Page loads in <2s on 4G (compressed images, CDN assets)
11. All placeholder content is clearly marked for easy customization

## Customization Points (marked in code with comments)

- `PHRASES` array — No button messages
- `REASONS` array — "Reasons I love you" list
- `ITINERARY` array — date plan items with time/emoji/activity/description
- `CELEBRATION_MESSAGE` — text shown after Yes
- `EASTER_EGG_MESSAGE` — text for instant Yes
- Photo images — swap files in `images/` folder
- Photo captions — update in HTML
- Color variables — CSS custom properties at top of `<style>`

## Decisions Made

| Decision | Choice | Why |
|----------|--------|-----|
| Single file vs multi-file | Single `index.html` with embedded CSS/JS | Simplest deployment, zero routing, one file to share |
| Framework | None (vanilla HTML/CSS/JS) | Overkill for 3 screens. Zero build step. |
| Confetti library | canvas-confetti CDN | 17KB, heart shapes, battle-tested |
| Photo lightbox | Custom (pure CSS/JS) | ~30 lines of code, no library needed for <10 photos |
| Music | Cut | Autoplay blocked by browsers, awkward in public. Not worth the UX friction. |
| GIF changes on No | Cut | Cleaner without external GIF dependencies. The text progression is funny enough. |
| No button behavior | Shrink + reposition + fade out | Combined approach: shrinks AND moves on each click, fades after all phrases |
| Hosting | GitHub Pages | Free, instant, clean URL |

## Assumptions

1. She opens this on her phone (iPhone Safari or Android Chrome)
2. User will swap placeholder photos later (5-8 slots)
3. User will provide real itinerary content (has the plan)
4. No custom domain needed — `username.github.io/valentine` works
5. Ready before Feb 14, 2026
