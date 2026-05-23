# Feature to Code

Use this when the PM asks "where is [feature X] implemented?" — checkout, login, search, notifications, the export button, a specific screen, an API endpoint, etc.

## Goal

Take a feature described in product language and find the code that implements it, showing the PM enough evidence that they can confirm, dig deeper, or hand it to an engineer.

## The core technique: search the words users see

The most reliable way to find a feature in code is to **search for strings the user would see in the UI** — button labels, page titles, error messages, URL paths. These are almost always either literal strings in the code or keys in a translation file, and either way they lead you straight to the implementation.

### Step 1 — Get the search terms from the PM

If the PM says "the checkout flow", the actual searchable strings are things like:
- Button labels: "Checkout", "Place order", "Pay now"
- URL paths: `/checkout`, `/cart`, `/payment`
- Error messages they remember: "Your card was declined"
- Component names they may have heard: "CheckoutModal", "PaymentForm"

Ask the PM for 2–3 specific strings if they haven't already given them. Phrasing: "Do you have a button label or URL for that? That's the fastest way to pin it down."

### Step 2 — Search the obvious places first

```bash
# Search for the exact UI string across source files, excluding noise
grep -rni "place order" \
  --include="*.tsx" --include="*.jsx" --include="*.ts" --include="*.js" \
  --include="*.py" --include="*.rb" --include="*.go" --include="*.java" \
  --include="*.kt" --include="*.swift" --include="*.vue" --include="*.svelte" \
  --include="*.html" --include="*.json" \
  --exclude-dir=node_modules --exclude-dir=.git --exclude-dir=dist --exclude-dir=build \
  . 2>/dev/null | head -20
```

For URL paths, search for the path string with a slash:
```bash
grep -rn "/checkout" --include="*.tsx" --include="*.ts" --include="*.jsx" --include="*.js" \
  --exclude-dir=node_modules --exclude-dir=.git . 2>/dev/null | head -20
```

### Step 3 — If strings are externalized to i18n files

Many apps externalize UI strings to JSON/YAML translation files (`en.json`, `messages.yml`, `locales/`). If your first search returns hits only in translation files, that's actually progress — find the **key** in the translation file, then search for the key in the source code.

```bash
# Found a translation file hit like "checkout.placeOrder": "Place Order"
# Now search for the key
grep -rn "checkout.placeOrder\|checkout\\.place_order" \
  --include="*.tsx" --include="*.ts" --include="*.jsx" --include="*.js" \
  --exclude-dir=node_modules . 2>/dev/null | head -20
```

### Step 4 — Follow the breadcrumbs upward

You've found a component or function. Now expand outward:

1. **What file is this in?** The path itself tells you a lot — `src/features/checkout/components/PlaceOrderButton.tsx` means there's likely a whole `checkout/` directory with the surrounding logic.
2. **Who calls this?** Search for the component name or function name across the repo:
   ```bash
   grep -rn "PlaceOrderButton\|<PlaceOrderButton" --include="*.tsx" --include="*.jsx" \
     --exclude-dir=node_modules . 2>/dev/null
   ```
3. **What does this call?** `view` the file and note its imports and the API/service calls it makes. Those are the next layer of the feature.

A feature usually has 3–5 layers worth describing to the PM:
- The **UI** (button, form, screen) the user interacts with
- The **client-side logic** (state management, validation, routing)
- The **API call** (the request that goes to the backend)
- The **backend handler** (the route + controller that receives the request)
- The **data layer** (the database table or model that gets read/written)

Trace as many of these as the PM needs.

## Mapping by feature type

### "Where is [screen / page]?"

URL is the fastest lead. Convert the URL to a file path using the framework's convention:
- Next.js app router: `/settings/billing` → `app/settings/billing/page.tsx`
- Next.js pages router: `/settings/billing` → `pages/settings/billing.tsx`
- React Router: search for the route string in JSX `<Route path="/settings/billing">`
- Rails: search `routes.rb` for the path, find the controller, then the view
- Django: search `urls.py` for the path, follow to the view function

### "Where is [button / form element]?"

Search for the label text. If the result is a translation key, follow up with the key search. If there are many results, narrow by the surrounding context (which page is this button on?) and search within that subdirectory.

### "Where is [API endpoint]?"

Search for the path string:
```bash
grep -rn '"/api/v1/orders\|"/orders' --include="*.ts" --include="*.js" --include="*.py" \
  --exclude-dir=node_modules . 2>/dev/null | head -20
```
Or search the HTTP method + path together for the route definition:
```bash
grep -rn "POST.*orders\|@app.post.*orders\|router.post.*orders" \
  --include="*.ts" --include="*.py" --exclude-dir=node_modules . 2>/dev/null
```

### "Where is [background job / scheduled task / webhook]?"

These don't have URLs the user sees. Look for:
- Cron / scheduler config: `crontab`, `cron.yaml`, `schedule.rb`, `@scheduled` annotations
- Job queues: search for `enqueue`, `perform_later`, `Queue`, `Worker`
- Webhook handlers: usually a route under `/webhooks/` or `/hooks/`

### "Where is [feature that touches the database]?"

Start from the schema:
```bash
# Common schema locations
find . \( -name "schema.prisma" -o -name "schema.rb" -o -name "models.py" \
  -o -path "*/migrations/*" -o -name "*.sql" \) \
  -not -path "*/node_modules/*" 2>/dev/null | head -20
```
Find the relevant table/model, then search for the model name across the codebase to find every place it's used.

## How to present the answer

PMs want a short narrative answer with evidence, not a file dump. A good response looks like:

> **The checkout flow lives in `src/features/checkout/`.** The main entry point is the `CheckoutPage` component (`src/features/checkout/CheckoutPage.tsx`), which is rendered at the `/checkout` route. The user fills out a form, hits "Place Order", which calls `submitOrder()` in `src/features/checkout/api.ts`. That hits the backend at `POST /api/v1/orders`, handled by `OrdersController.create` in `server/controllers/orders.rb`. Orders are stored in the `orders` table, with line items in `order_items`.
>
> Three files to look at if you want the full picture: `CheckoutPage.tsx` (the screen), `api.ts` (the request), `OrdersController.rb` (the backend).
>
> One thing worth knowing: I noticed the file `LegacyCheckoutPage.tsx` also exists but isn't referenced anywhere — looks like dead code from a previous version. Want me to confirm?

Lead with the answer in one sentence. Then the trace. Then the three best files. Then any oddities you noticed.

## Gotchas

- **Components can be reused across features.** A `Button` component isn't the checkout feature — it's a UI primitive. Don't follow generic imports down infinite rabbit holes. Stop when you've explained the feature; if the PM wants more, they'll ask.
- **Server-rendered strings.** If a string doesn't appear in frontend code, it may be coming from the backend (e.g., error messages in an API response). Search backend code too.
- **Dynamic strings.** Sometimes UI text is composed: `"Welcome, " + userName + "!"`. The full string won't exist anywhere. Search for the most distinctive sub-string ("Welcome,").
- **Conditional rendering and feature flags.** A feature may be implemented but gated. After locating it, check whether it's wrapped in a flag check (see `shipped-vs-planned.md`).
- **Mobile platforms diverge.** A feature may have separate iOS and Android implementations. If you find it in only one platform's code, check the other.
