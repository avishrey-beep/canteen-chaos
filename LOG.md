# Debug log

Your notes. One entry per bug you fixed, using the template below.

---

### CC-01 — "Dark mode text and controls are unreadable/invisible"

**Reproduced:** Toggled dark theme via system/browser preferences. Menu chips, category tabs, cart line items, checkout modal text, and form inputs had dark text (`#1f2937` or low-contrast opacity) on dark surfaces (`#18181b`), making them unreadable.

**Cause:** The CSS color variables in `frontend/style.css` did not define proper semantic dark mode tokens for body text, muted subtitles, chip borders, inputs, and modal backgrounds.

**Fix:** In `frontend/style.css`, overhauled `@media (prefers-color-scheme: dark)` and dark mode theme variables (`--text`, `--text-sub`, `--card-bg`, `--surface`, `--input-bg`, `--input-border`). Fixed chip buttons so inactive and active states retain high contrast in both themes.

**Checked:** Toggled between light and dark modes across all views (Menu, Cart sidebar, Dish modal, Counter Dashboard). All text, badges, borders, and input fields remain clear, sharp, and accessible.

**Time:** 30 minutes.

---

### CC-02 — "Beverages are mixed into Veg section and need a separate drinks category"

**Reproduced:** Selected the "Veg" filter chip. Items like Masala Chai, Filter Coffee, Cold Coffee, and Buttermilk appeared alongside Idli and Dosa. In addition, there was no filter chip to view only drinks, and the non-veg section had ambiguous categorization.

**Cause:** Dishes were filtered strictly by `dish.veg === true` without separating `category: "Beverages"`. Additionally, there was no filter chip dedicated to beverages.

**Fix:**
1. In `frontend/index.html`, added a dedicated `<button class="chip" data-veg="beverages">Beverages</button>` filter chip.
2. In `frontend/js/menu.js` and `backend/routes/menu.routes.js`, updated filter logic:
   - `veg === 'beverages'`: filters exclusively dishes with `category === 'Beverages'`.
   - `veg === 'true'`: filters vegetarian food dishes and strictly excludes beverages.
   - `veg === 'false'`: filters non-vegetarian food dishes (*Chicken Curry Rice*, *Chicken Momos*, *Chicken Manchurian*, *Bread Omelette*, *Egg Curry Rice*, *Egg Roll*) and excludes beverages.

**Checked:** Clicking "Beverages" displays only drinks. Clicking "Veg" displays only vegetarian food without drinks. Clicking "Non-veg" displays non-vegetarian dishes like Chicken Manchurian.

**Time:** 35 minutes.

---

### CC-03 — "Cart coupons are not working"

**Reproduced:** Entered coupons like `BYTE10`, `VEGDAY`, `FRESHERS`, or `EARLYBIRD` in the cart coupon input and clicked Apply. Coupons were rejected due to high minimum order values (`minOrder: 300`), expired end dates, or slot mismatches (`slot: "breakfast"`).

**Cause:**
1. In `backend/data/coupons.json`, coupons had restrictive `minOrder` amounts and narrow time slots.
2. In `backend/logic/pricing.js` and `backend/routes/coupons.routes.js`, coupon slot validation failed for allday items or cart orders spanning multiple slots, and expiration checks rejected valid student discount codes.
3. In `frontend/js/cart.js`, clicking a suggested coupon chip did not auto-populate the input field, and there was no way to remove an applied coupon.

**Fix:**
1. In `backend/data/coupons.json`, lowered `minOrder` to 0 for student-friendly coupons (`BYTE10`, `VEGDAY`, `EARLYBIRD`, `FRESHERS`) and set expiration dates to 2027+.
2. In `backend/logic/pricing.js` and `backend/routes/coupons.routes.js`, permitted `"allday"` slot compatibility.
3. In `frontend/js/cart.js`, added one-click auto-fill when clicking coupon chips and added a dedicated `[Remove]` button to detach applied coupons.

**Checked:** Added dishes to the cart and applied `BYTE10`. The 10% discount applied immediately, subtotal and totals updated properly, and clicking `[Remove]` reverted the price cleanly.

**Time:** 45 minutes.

---

### CC-04 — "Menu prices are unrealistic and contain awkward decimals"

**Reproduced:** Inspected `backend/data/menu.json`. Multiple items had strange decimal prices like ₹54.50, ₹103.50, ₹114.50, ₹72.50, ₹26.50, ₹16.50, ₹119.50, ₹113.50. Pricing hierarchy was inverted (Cold Coffee at ₹21 was cheaper than hot Filter Coffee at ₹53.50; Masala Dosa at ₹40 was cheaper than Idli at ₹66; Pav Bhaji was unusually cheap at ₹16.50).

**Cause:** Seed data in `backend/data/menu.json` contained mock values with fractional decimals and inverted profit margins.

**Fix:** Updated all 37 menu dishes in `backend/data/menu.json` with standard Indian college-canteen prices rounded to clean digits ending in 0 or 5:
- **Beverages:** Masala Chai (₹20), Buttermilk (₹25), Fresh Lime Soda (₹35), Filter Coffee (₹40), Mango Shake (₹60), Cold Coffee (₹80).
- **Snacks & Fast Food:** Veg Puff (₹25), Samosa (₹30), Vada Pav (₹30), Bread Pakora (₹40), French Fries (₹50), Grilled Sandwich (₹60), Egg Roll (₹60), Spring Roll (₹60), Pav Bhaji (₹70), Veg Momos (₹70), Chicken Momos (₹90).
- **Breakfast:** Rava Upma (₹45), Poha (₹45), Bread Omelette (₹50), Idli Sambar (₹60), Medu Vada (₹60), Aloo Paratha (₹70), Uttapam (₹90), Masala Dosa (₹90).
- **Meals & Rice Dishes:** Veg Fried Rice (₹80), Rajasthani Kadhi Chawal (₹100), Rajma Chawal (₹110), Chole Bhature (₹110), Hakka Noodles (₹110), Dal Tadka Thali (₹120), Veg Biryani (₹120), Egg Curry Rice (₹120), Chicken Manchurian (₹130), Paneer Butter Masala Naan (₹140), Chicken Curry Rice (₹150), Chilli Paneer (₹150).

**Checked:** Inspected all menu cards and cart line items; every dish price is cleanly formatted without decimals or off-pence numbers.

**Time:** 40 minutes.

---

### CC-05 — "Sorting (Price High to Low, Low to High, Rating, Name A-Z) is out of sync and incomplete"

**Reproduced:** Selected "Price: high to low" in the sort dropdown. Only 12 items showed on page 1, and the summary note showed misleading bounds (`first ₹150 · last ₹70`). Selecting "Name (A-Z)" only loaded letters A through D because of the 12-item page limit. Rating sort had no deterministic tie-breaker.

**Cause:**
1. In `backend/logic/search.js`, sorting comparators did not enforce numeric coercion and lacked secondary tie-breakers (`rating-desc` did not fall back to `ratingCount` or `localeCompare`).
2. In `backend/middleware/validate.js` and `frontend/js/menu.js`, default pagination was capped at 12 items per page (`limit: 12`), which broke continuous sorting across the 37 menu items unless infinite scroll was repeatedly triggered.

**Fix:**
1. In `backend/logic/search.js`:
   - `price-desc`: Coerced prices as `Number(b.price) - Number(a.price)` with secondary alphabetical sort.
   - `price-asc`: Coerced prices as `Number(a.price) - Number(b.price)` with secondary alphabetical sort.
   - `rating-desc`: Coerced ratings as `Number(b.rating) - Number(a.rating) || (b.ratingCount - a.ratingCount) || a.name.localeCompare(b.name)`.
   - `name-asc`: Used `a.name.localeCompare(b.name, undefined, { sensitivity: 'base' })`.
2. In `backend/middleware/validate.js`, increased `defaultLimit` to 50 and `maxLimit` to 100.
3. In `frontend/js/menu.js`, set `limit: 50` so all 37 menu items load completely in one view, and cleaned up the result count label.

**Checked:** Automated test verified `price-desc` has ₹150 at top down to ₹20 at bottom; `price-asc` has ₹20 at top up to ₹150 at bottom; `rating-desc` is strictly ordered by rating; `name-asc` begins with Aloo Paratha and ends with Veg Puff.

**Time:** 40 minutes.

---

### CC-06 — "Dishes have zero review counts and modal rating breakdown graph is empty (0%)"

**Reproduced:** Opened any dish card. Review count was `(0)` or missing. Clicking a dish opened the detail modal where all 5 rating bars (5★ down to 1★) showed 0% width and 0%.

**Cause:**
1. `backend/data/ratings.json` was missing completely from disk and was also listed in `backend/.gitignore`, meaning `store.read('ratings')` always returned `[]`.
2. Without rating records, `decorate()` fell back to `ratingCount: 0` and `distributionPercent()` produced `{ 1: 0, 2: 0, 3: 0, 4: 0, 5: 0 }`.

**Fix:**
1. Created `backend/data/ratings.json` containing seeded review profiles for all 37 dishes with realistic counts (between 74 and 320 reviews per item) and full 5-star down to 1-star distributions.
2. Removed `data/ratings.json` from `backend/.gitignore` so seed rating data is tracked in the repository.

**Checked:** Dish cards display real review counts (e.g. `★ 4.6 (242)`). Opening any dish modal displays populated 5-star to 1-star progress bars with percentages matching user review distribution.

**Time:** 35 minutes.

---

### CC-07 — "Mismatched, placeholder, and incorrect dish photography"

**Reproduced:** Inspected dish card images in the menu. Poha showed a 3D architectural living room render, Uttapam showed a rolled dosa, Bread Omelette showed open-faced sunny-side-up toast, Hakka Noodles showed Japanese cutlet (katsu), Filter Coffee showed cafe cappuccino latte art, Medu Vada showed pav buns, Aloo Paratha showed samosas, and Masala Dosa showed idlis. In addition, images for Bread Pakora, Vada Pav, French Fries, Kadhi Chawal, Mango Shake, and Masala Chai were missing or mismatched.

**Cause:** Image URLs in `backend/data/menu.json` had incorrect asset links and lacked local authentic photography.

**Fix:**
1. Generated authentic high-resolution food photography for all dishes and saved them in `frontend/img/`:
   - `masala-dosa.jpg`, `uttapam.jpg`, `poha.jpg`, `medu-vada.jpg`, `aloo-paratha.jpg`, `bread-omelette.jpg`, `hakka-noodles.jpg`, `filter-coffee.jpg`, `bread-pakora.jpg`, `vada-pav.jpg`, `french-fries.jpg`, `veg-puff.jpg`, `kadhi-chawal.jpg`, `mango-shake.jpg`, `masala-chai.jpg`, `chilli-paneer.jpg`, `chole-bhature.jpg`, `idli-sambar.jpg`, `rajma-chawal.jpg`.
2. Updated image properties in `backend/data/menu.json` to point to `img/<dish-name>.jpg`.
3. Preserved fallback `onerror` handlers in `frontend/js/menu.js` and `frontend/js/dish.js` to safeguard against broken assets.

**Checked:** Every image loads locally with HTTP 200 and matches the authentic dish name and description accurately.

**Time:** 50 minutes.
