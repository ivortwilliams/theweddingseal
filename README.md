# The Wedding Seal

A wedding invitation, from a stationery house that does not exist, for a wedding that is not
happening, between Thomas and the mother of his friend Matt.

Live at **https://theweddingseal.com**.

## What's here

| Path | What it is |
|---|---|
| `email/invite.html` | The invitation email. Tables and inline styles only, no comments. |
| `email/invite.txt` | The plain-text part of the same email. |
| `public/index.html` | The landing page: the envelope that opens, the invitation, the RSVP form. |
| `server.js` | Serves the page, takes the RSVP, emails it to Thomas via Resend. |

## Why the envelope is on the web and not in the email

Gmail strips `<style>` blocks unreliably, removes all JavaScript, and ignores CSS animation.
An envelope that opens cannot run inside the message. So the email is a static invitation
card with an **Open your invitation** button, and the animation lives on the landing page —
which is exactly what Paperless Post and Greenvelope do, for exactly this reason.

## Deliverability notes

The domain is new, so it has no sending reputation. What we do about that:

- SPF, DKIM and DMARC are all set before the first send.
- Resend's click and open tracking are **off**. Both rewrite links through a tracking
  subdomain, which reads as bulk mail.
- The From address is a person, not `noreply@`.
- Every send uses `invitations@theweddingseal.com`. Splitting a new domain's tiny volume
  across several From addresses splits the reputation with it.
- `email/invite.html` is the sendable body and nothing else. No HTML comments: comments
  travel with the message, so notes about the message belong here instead.
- `email/invite.txt` is the plain-text part, and it says what the HTML says. A missing or
  mismatched text part is a bulk signal on its own.
- No `List-Unsubscribe` header. It helps genuine bulk mail and hurts a message that is
  meant to read as one person writing to one person.

### What got us marked as spam, 2026-09-08

The first send, 08:37, reached the Gmail **inbox**. A revision at 09:12 went to **spam**,
with Gmail giving "similar to messages that were identified as spam in the past" — its
content classifier, not an authentication failure. DKIM, SPF and DMARC were verified and
unchanged across both.

Two things differed, and we could not separate them without more sends:

1. **The copy.** The revision wrapped the button in "TAP BELOW" with directional arrows,
   "Your invitation is sealed. Open it.", and "the RSVP takes ten seconds", and rewrote the
   preheader into the same register. That is the voice of bulk marketing.
2. **The pattern.** The domain was ninety minutes old and had sent six messages, including
   two identical invitations five minutes apart, plus RSVP notifications from a second
   address.

The button is still gold and still the obvious thing to press. The urgency copy around it
is gone. Prominence was never the problem.

Rules for the next send:

- One message. No repeats of the same subject to the same address.
- Leave the domain quiet beforehand. A day of silence beats an hour.
- Test on a fresh recipient rather than resending to an address that has already filed
  this sender under spam. Gmail keeps per-recipient history, and marking "not spam"
  only trains that one account.

## Configuration

| Variable | Purpose |
|---|---|
| `RESEND_API_KEY` | Sends the RSVP notification. Secret. |
| `NOTIFY_EMAIL` | Where RSVPs land. |
| `NOTIFY_FROM` | From address on the notification. |
| `ADMIN_TOKEN` | Guards `/admin`, which lists every response. Secret. |

`/admin?token=…` reads a file on the container's ephemeral disk, so it empties on redeploy.
The notification email is the durable record. The token itself lives only in the App Platform
secret, never in this repo.

## Where it runs

DigitalOcean App Platform app `theweddingseal`, Sydney, `apps-s-1vcpu-0.5gb`, deployed from this
repository. Deliberately a separate app from the `tastyradio` droplet, which also serves
`radio.truthseekersbyo.com` and must not be disturbed.
