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
