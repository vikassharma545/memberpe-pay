# One-tap UPI payments

`index.html` is a single static page that turns an `https://` link into a UPI
app handoff. Publish it once and members pay in one tap instead of copying a
UPI ID by hand.

## Why this page exists

Telegram clients only open a whitelist of URL schemes — `http`, `https`, `tg`
and a few others. `upi://` is not on it, so a `upi://` link is ignored when
tapped, on **every** platform:

| Where the link is | What happens |
|---|---|
| Inline keyboard button | Bot API rejects it: *"Unsupported URL protocol"* |
| HTML text link in a message | API accepts it, every client ignores the tap |
| Pasted into Chrome | **Works** — Chrome hands `upi://` to the UPI app |
| Encoded in a QR code | **Works** — UPI apps parse the scheme when scanning |

The last two are the opening. Telegram opens `https://` normally, so a page at
an `https://` URL can receive the tap and hand off to `upi://` itself — the same
handoff Chrome already performs.

## Publishing it (GitHub Pages, free)

1. Create a GitHub account if you don't have one.
2. Create a **public** repository — e.g. `memberpe-pay`.
3. Upload `index.html` to it (drag and drop on github.com works).
4. Go to **Settings → Pages**, set *Source* to `Deploy from a branch`, branch
   `main`, folder `/ (root)`, and **Save**.
5. Wait ~1 minute. Your page is at:
   `https://<your-username>.github.io/memberpe-pay/`
6. Test it before wiring it in — open this in your phone's browser:
   ```
   https://<your-username>.github.io/memberpe-pay/?pa=yourupi@okaxis&pn=Your+Name&am=1.00&tn=TEST
   ```
   You should see a ₹1 payment card. Tapping **Pay with UPI app** must open
   GPay/PhonePe.
7. Put the base URL in `.env` and restart the bot:
   ```
   PAY_PAGE_URL=https://<your-username>.github.io/memberpe-pay/
   ```

Cloudflare Pages works the same way if you prefer it.

## Without it

Leave `PAY_PAGE_URL` empty and the bot shows copy-the-UPI-ID plus a QR code
instead. That needs no hosting and works everywhere — it is just more taps.

## How the page behaves

- **Rebuilds the `upi://` URI itself** from `pa` / `pn` / `am` / `tn` rather
  than accepting a ready-made one, so a crafted link cannot smuggle extra UPI
  parameters into the payment.
- **Validates the UPI ID and amount** and shows an error instead of opening a
  UPI app with wrong details — a wrong payee or amount costs the payer real
  money.
- **Does not auto-redirect.** Mobile browsers block scheme handoffs the user
  did not initiate, and a silent failure leaves a blank page with no way
  forward. The payer taps a button.
- Always shows the UPI ID with a copy button, so the manual route survives even
  if the handoff fails on some device.
- No tracking. The only external request is Telegram's Mini App SDK
  (`telegram-web-app.js`), used to detect when the page is opened inside
  Telegram and bounce out to a real browser; the page works without it.
- **Cannot verify who made the link.** The parameters are unsigned, so a
  phisher can craft a MemberPe-looking link paying their own UPI ID. The page
  therefore tells the payer to compare the UPI ID with the one in their
  Telegram message from the bot, which is the only trustworthy source.

## Privacy note

The UPI ID, amount and reference travel in the URL, so they appear in the
payer's browser history. All three are already visible in the Telegram message,
so this leaks nothing new — but do not add anything sensitive to the query
string.
