# RALLY — Color Theme Selection Rationale

## 1. Brand direction

**Brand concept:** RALLY — Pickleball & Badminton Court Reservation  
**Brand personality:** Modern, energetic, welcoming, and premium  
**Design direction:** A premium sports-club experience supported by a clean, easy-to-use reservation and queue interface.

The selected palette combines **forest green**, **vibrant lime**, and **warm off-white**. Each color has a distinct role: forest green establishes the brand, lime brings energy and draws attention to important actions, and warm off-white keeps the experience comfortable and readable.

---

## 2. Selected color palette

| Color | Hex code | Tailwind Token | Role |
| :--- | :--- | :--- | :--- |
| **Deep Forest Green** | `#173F32` | `rally-forest` | Primary brand color |
| **Court Green** | `#24764E` | `rally-court` | Secondary green for interactive & available states |
| **Electric Lime** | `#C8F169` | `rally-lime` | Accent, selected slot, and call-to-action color |
| **Warm Ivory** | `#F4F1E7` | `rally-ivory` | Main page background |
| **Soft White** | `#FFFFFF` | `rally-surface` | Cards, forms, and content surfaces |
| **Charcoal** | `#252B28` | `rally-charcoal` | Main text color |

### Deep Forest Green — `#173F32`

**Purpose:** Establish the brand's identity and provide a strong visual foundation.

Why it was selected:
- Green connects naturally with outdoor recreation, courts, and the broader visual language of sport.
- A deep shade feels grounded, dependable, and professional.
- It creates a premium club atmosphere rather than the appearance of a generic booking template.
- It contrasts effectively with the brighter lime accent.

Recommended use:
- Logo and brand mark
- Navigation bar and footer
- Major headings on light backgrounds
- Dark hero sections
- Selected navigation or interface elements

### Court Green — `#24764E`

**Purpose:** Provide a more active, medium-green shade for secondary emphasis.

Why it was selected:
- It maintains the green brand identity while being lighter and more flexible than the primary forest green.
- It can distinguish interactive states without relying on the lime accent everywhere.
- It creates a visual bridge between the dark primary color and the light background.

Recommended use:
- Secondary buttons
- Links and active navigation states
- Availability indicators, when the status is genuinely available
- Icons and small interface highlights

### Electric Lime — `#C8F169`

**Purpose:** Add energy and direct attention to important actions.

Why it was selected:
- Its brightness gives the palette a lively, athletic character.
- Against forest green, it creates a clear visual contrast.
- It makes key actions easier to spot when used selectively.
- It helps the brand feel contemporary and distinctive.

Recommended use:
- Primary "Book a Court" / "Confirm Court Booking" button
- Selected date or time slot
- Important calls to action (e.g., "Promote Next in Queue")
- Small highlights, badges, and decorative accents

**Usage note:** Lime should be an accent, not the dominant page color. If too many elements use it, the interface loses visual hierarchy. Use dark text (`#173F32` or `#252B28`) on lime backgrounds for readability.

### Warm Ivory — `#F4F1E7`

**Purpose:** Create a calm, comfortable canvas for the website.

Why it was selected:
- It is softer in appearance than pure white.
- It balances the intensity of forest green and lime.
- It gives the interface a warm, welcoming, and refined character.
- It provides a clean surface for reservation forms, schedules, and court information.

Recommended use:
- Main page background
- Booking and reservation pages
- Section backgrounds
- Large content areas behind cards

### Soft White — `#FFFFFF`

**Purpose:** Separate important content into clear, readable surfaces.

Recommended use:
- Court cards
- Booking forms and queue drawers
- Dialogs and popovers
- Dashboard panels

### Charcoal — `#252B28`

**Purpose:** Provide readable text without making every text element appear as visually heavy as pure black.

Recommended use:
- Body text
- Form labels
- Supporting information
- Text on light backgrounds

---

## 3. Suggested color distribution

Use the following ratio as a starting point for the overall interface:

| Color | Suggested share | Function |
| :--- | :---: | :--- |
| **Forest green (`#173F32`)** | 60% | Brand foundation and major sections |
| **Warm off-white (`#F4F1E7`)** | 30% | Page backgrounds and breathing room |
| **Vibrant lime (`#C8F169`)** | 10% | Calls to action and selected highlights |

This is a visual guideline, not a strict measurement. White cards and charcoal text can be layered into the interface as needed.

The main principle is to let forest green establish the identity, let off-white provide space, and reserve lime for moments that deserve attention.

---

## 4. How the palette supports the user experience

| User experience goal | Color contribution |
| :--- | :--- |
| **Recognizable brand identity** | Forest green (`#173F32`) creates a consistent visual foundation |
| **Clear calls to action** | Electric lime (`#C8F169`) makes primary actions stand out |
| **Comfortable browsing** | Warm ivory (`#F4F1E7`) reduces the starkness of an all-white interface |
| **Readable information** | Charcoal text (`#252B28`) and white surfaces (`#FFFFFF`) create clear content hierarchy |
| **Consistent interaction states** | Court green (`#24764E`) supports secondary and active interface elements |

The palette is intended to make the website feel like a professional pickleball & badminton club with a convenient digital reservation experience—not simply an administrative booking system.

---

## 5. Example component styling

| Component | Suggested treatment |
| :--- | :--- |
| **Main navigation** | Forest green (`#173F32`) background with light text, or ivory background with forest-green text |
| **Primary button** | Electric lime (`#C8F169`) background with forest-green (`#173F32`) text |
| **Secondary button** | Forest-green (`#173F32`) or Court-green (`#24764E`) background with white text, or green outline |
| **Main page** | Warm ivory (`#F4F1E7`) background |
| **Court card / Panel** | Soft white (`#FFFFFF`) surface with charcoal (`#252B28`) text |
| **Selected time slot** | Electric lime (`#C8F169`) background with dark (`#173F32`) text |
| **Available status** | Court green (`#24764E`), paired with a clear `"Available"` label/icon |
| **Booked / Waitlist Queue status** | Amber, paired with a `"Booked · Join Queue"` or `"Waitlisted (#N)"` label |
| **Unavailable / Walk-In Blocked status** | Muted gray, paired with an `"Unavailable"` or `"Walk-In Blocked"` label |
| **Footer** | Forest green (`#173F32`) background with light text |

Do not communicate booking status through color alone. Include text labels or icons so that status remains understandable to users with color-vision differences.

---

## 6. Accessibility and implementation considerations

- Maintain sufficient contrast between text and its background. In particular, use dark text (`#173F32` or `#252B28`) on lime (`#C8F169`) and light backgrounds.
- Check actual color-contrast ratios against WCAG guidance before release; do not assume that a visually strong combination automatically passes.
- Avoid using lime for long paragraphs or small text.
- Use consistent colors for the same actions and states throughout the website.
- Provide visible hover, focus, disabled, and selected states.
- Test the palette on desktop and mobile screens under different brightness conditions.

---

## 7. Final recommendation

The selected palette is:

- **Deep Forest Green (`#173F32`)** — the foundation of the brand.
- **Court Green (`#24764E`)** — secondary interaction and status color.
- **Electric Lime (`#C8F169`)** — energy and emphasis.
- **Warm Ivory (`#F4F1E7`)** — the primary background.
- **Soft White (`#FFFFFF`)** — cards and content surfaces.
- **Charcoal (`#252B28`)** — readable text.

Together, these colors create a balance of athletic energy, professional credibility, and visual comfort. They support the RALLY identity while keeping the most important task—finding and reserving a court or joining a queue—clear and easy to complete.
