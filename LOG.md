# Debug log

Your notes. One entry per bug you fixed, using the template below.

This file is read as carefully as your code. A correct fix you cannot
explain counts for little; a bug you could not fix but investigated
honestly still counts for something.

Delete the example before you submit.



## Example — delete this

### CC-99 — "The cart total is wrong"

**Reproduced:** Added 2 dosas at Rs. 60 each. The cart showed
Rs. 119.99999 instead of Rs. 130. Happened every time, on any dish with
a price ending in .50.

**Cause:** The total was being added up with plain floating point and
never rounded, so 0.1 + 0.2 style errors showed up on screen. The
rounding helper existed but this one place was not using it.

**Fix:** Ran the total through the existing rounding helper instead of
adding a new one, so every price on screen goes through the same path.

**Checked:** Cart, checkout and the order screen all show Rs. 130 now.
Prices without decimals still show without a trailing .00.

**Time:** about 40 minutes, most of it working out that the cart and the
order screen round in different places.



### CC-01 — "The search suggestions are behind everything"

**Reproduced:** Typed "momo" into the search bar. The dropdown appeared, but items past the first suggestion were covered by the category bar below it. Clicking anywhere on the lower suggestions didn't select anything because the category bar was sitting in front.

**Cause:** `.search-wrap` had `z-index: 1`, creating a stacking context with priority 1. The `.cat-tabs` bar right below it had `z-index: 40`. Even though `.suggest-box` had `z-index: 100`, it was trapped inside `.search-wrap`'s stacking context, so the category tabs were painted above it and intercepted mouse clicks.

**Fix:** Changed `.search-wrap`'s z-index to 50 in `frontend/style.css` so its stacking context sits above `.cat-tabs` (40).

**Checked:** Typed various dish names in the search bar. The suggestion dropdown renders cleanly above the category strip, hover styles trigger on all items, and clicking any suggestion populates the search and opens the dish.

**Time:** about 15 minutes.
 
 
 
### CC-02 — "Can't read anything in dark mode"

**Reproduced:** Toggled the theme switch to dark mode. The card backgrounds turned dark brown/black (`#201913`), but the dish names and prices became nearly unreadable, while the meta info in grey remained visible.

**Cause:** `.dish-body` had `color: #2b2118;` explicitly hardcoded. In light mode this matched `--ink`, but in dark mode `--card` is `#201913` and `--ink` is `#f2e8df`. Because of the hardcoded color, `.dish-name` and `.dish-price` stayed dark brown on the almost identical dark card background.

**Fix:** Replaced the hardcoded hex with `color: var(--ink);` on `.dish-body` in `frontend/style.css` so it uses the theme variable.

**Checked:** Toggled between light and dark modes. Dish titles and prices are crisp and legible in both themes.

**Time:** about 10 minutes.
 
 
 
### CC-03 — "The menu is wider than my phone"

**Reproduced:** Resized the viewport down to mobile width (around 500–600px). The menu grid stretched horizontally beyond the viewport, creating an annoying horizontal scrollbar. The right-hand column was pushed half off-screen, cutting off the Add to Cart buttons on those dishes.

**Cause:** Inside the `@media (max-width: 760px)` breakpoint, `.name-btn` was set to `width: max-content;`. In CSS Grid, tracks sized with `1fr` resolve with a minimum size of `auto`. When dishes have lengthy titles (such as "Paneer Butter Masala with Butter Naan"), `max-content` prevented the button from wrapping and forced the entire grid column to expand to fit the text, overflowing the screen width.

**Fix:** Removed `.name-btn { width: max-content; }` from the mobile media query in `frontend/style.css`, letting dish names wrap naturally as intended by `.dish-name { overflow-wrap: break-word; }`.

**Checked:** Tested viewport widths from 360px up to 760px. Dish cards stay completely inside their columns, long dish names wrap cleanly across multiple lines, and no horizontal scrollbar appears.

**Time:** about 25 minutes.
 
 
 
### CC-04 — "The buttons don't work on my tablet"

**Reproduced:** Set browser viewport width to 800px (tablet range: 761px–900px). Clicked the "Add to Cart" button and the star favorite icon on various dish cards. Neither button reacted, yet on desktop (>900px) and phone (<760px) both worked normally.

**Cause:** In `frontend/style.css`, `@media (min-width: 761px) and (max-width: 900px)` injected pseudo-elements `.dish-card::after` (positioned over the bottom 58px with z-index 3) and `.img-wrap::after` (positioned over the top-right 52x52px with z-index 1). Because `pointer-events: none` was omitted, these invisible overlay elements sat right on top of the Add button and favorite star, capturing all click and touch events.

**Fix:** Added `pointer-events: none;` to `.dish-card::after, .img-wrap::after` under the tablet media query block in `frontend/style.css`.

**Checked:** At 768px and 820px tablet widths, clicked "Add to Cart" and the favorite star. Clicks register immediately, incrementing cart quantities and toggling favorite state.

**Time:** about 20 minutes.
 
 
 
### CC-05 — "The category bar scrolls away on my phone"

**Reproduced:** Opened the site on a mobile screen size (<=480px) and scrolled down through the dish list. The sticky filter and category bar scrolled away out of view instead of remaining pinned at the top like it did on desktop.

**Cause:** The `@media (max-width: 480px)` stylesheet set `.view { overflow-x: hidden; }`. Setting any overflow other than `visible` on an ancestor element creates a new clipping/scroll context that breaks standard `position: sticky` on descendant elements (here `.filters`).

**Fix:** Removed `.view { overflow-x: hidden; }` from the mobile media query in `frontend/style.css`. The underlying overflow cause had already been solved by fixing dish title button wrapping in CC-03.

**Checked:** Tested scrolling through the menu on mobile screen widths (375px, 412px, 480px). The category and filter bar sticks reliably to the top of the viewport while browsing dishes.

**Time:** about 15 minutes.
 
 
 
### CC-06 — "I ordered more than they had"

**Reproduced:** Checked a dish with only 2 items in stock (e.g. Samosa). In the UI or via direct API call, requested a quantity of 5. The order passed validation and was accepted with status 201, leaving negative stock on the server.

**Cause:** In `backend/logic/validation.js`, `validateLine` verified that the dish was served, not disabled, and not sold out (`dish.stock <= 0`), but never checked whether the requested `qty` was greater than `dish.stock`. Additionally, `reserveStock` blindly decremented stock without checking if there was enough left, and `setQty` on the frontend only checked `MAX_PER_DISH` (10).

**Fix:** In `backend/logic/validation.js`, added a check in `validateLine` rejecting any line where `qty > Number(dish.stock)`. Added a check in `reserveStock` verifying `dish.stock >= item.qty`. Also added a stock check in `frontend/js/state.js` inside `setQty` to stop users before submitting.

**Checked:** Attempted to add and place orders for quantities exceeding stock. The UI shows "Only 2 available" and the backend returns a 400 validation error if a request attempts to order more than available stock.

**Time:** about 30 minutes.
 
 
 
### CC-07 — "Cancelling makes it worse"

**Reproduced:** Placed an order for 1 dish when stock was 5 (stock dropped to 4). Immediately cancelled the order. Instead of restoring to 5, the stock dropped further down to 3.

**Cause:** In `backend/logic/validation.js`, `releaseStock` had a sign error on line 125: `stock: dish.stock - line.qty`. It subtracted the order line quantity from the current stock instead of adding it back.

**Fix:** Changed subtraction to addition (`dish.stock + line.qty`) in `releaseStock`.

**Checked:** Placed an order and then cancelled it; checked the dish stock on `/api/menu/:id`. Stock correctly restored by the exact ordered quantity.

**Time:** about 10 minutes.
 
 
 
### CC-08 — "An old coupon still works"

**Reproduced:** Added items to the cart and applied coupon `FRESHERS24`. The coupon granted Rs. 30 discount and was accepted at quote and checkout despite having expired on 2024-09-30.

**Cause:** In `backend/routes/coupons.routes.js`, the `GET /api/coupons` route filtered out expired coupons using `new Date(c.expiresAt) > now`. However, `pricing.applyCoupon` in `backend/logic/pricing.js` only checked `usesLeft`, `minOrder`, `vegOnly`, and `slots`. It never compared `coupon.expiresAt` with `now`.

**Fix:** Added an expiration check in `applyCoupon` in `backend/logic/pricing.js`: if `coupon.expiresAt && new Date(coupon.expiresAt) <= now`, return `{ valid: false, discount: 0, reason: 'This coupon has expired' }`.

**Checked:** Tested quote with `FRESHERS24`; received error "This coupon has expired" with 0 discount. Valid active coupons (`BYTE10`, `FLAT50`) continue to apply discounts properly.

**Time:** about 15 minutes.
 
 
 
### CC-09 — "The menu shows more dishes than it should"

**Reproduced:** Loaded the homepage. Instead of loading the initial batch of 5 dishes and showing a "Load more" button / infinite scroll sentinel, the page rendered the entire menu list of all 37 dishes at once on the first request.

**Cause:** In `backend/logic/search.js`, `paginate` computed `const items = list.slice(start, start + perPage);` correctly, but then returned `items: list,` in the response payload. The slice was discarded and the unpaged full list was always returned.

**Fix:** Changed `items: list` to `items` in `paginate` in `backend/logic/search.js`.

**Checked:** Fetched `/api/menu?limit=5&page=1` and `/api/menu?limit=5&page=2`. Page 1 returns only the first 5 dishes and page 2 returns the next 5 dishes. Verified in the UI that 5 cards render initially and additional dishes load progressively on scroll / "Load more".

**Time:** about 15 minutes.
 
 
 
### CC-10 — "Sorting by price is backwards"

**Reproduced:** Selected "Price: low to high" from the sort dropdown. The most expensive dishes (Thali / Paneer Butter Masala at Rs. 140–160) appeared first. Selected "Price: high to low" and the cheapest dish (Chai at Rs. 15) appeared first. Both sorting options were inverted.

**Cause:** In `backend/logic/search.js`, the `SORTERS` dictionary had inverted comparators: `'price-asc'` used `(a, b) => b.price - a.price` (which sorts descending), while `'price-desc'` used `(a, b) => a.price - b.price` (which sorts ascending).

**Fix:** Swapped the comparator implementations in `backend/logic/search.js` so `'price-asc'` calculates `a.price - b.price` and `'price-desc'` calculates `b.price - a.price`.

**Checked:** Tested both sort options via the frontend dropdown and verified via `smoke.js` test checking `sort by price works`. Low-to-high returns cheapest items first and high-to-low returns most expensive first.

**Time:** about 10 minutes.



## Could not fix

For anything you investigated but did not solve. Say what you tried and
where you got to. This is worth marks — leaving it blank when you got
stuck is not.

### CC-0X — "<the complaint>"

**What I tried:**

**Where I got to:**

**What I would try next:**



## Extra credit

Anything not on the bug log: a problem you found yourself, a test you
wrote, or a fix you are unsure about. Same format, plus one line on how
you noticed it.
