# AI App Launch Checklist

You built a prototype with Claude, Cursor, or another AI tool, and it works on your laptop. This is the list I go through before an app like that meets real users and real money.

It's not about the AI part. The AI usually did its job. It's about everything around it: accounts, payments, secrets, deploys, and knowing when something breaks. That's where most AI-built apps stall, and I wrote more about why in [Your AI App Is 90% Done. The Last 10% Is the Hard Part.](https://bonniedoone.ai/blog/ai-app-to-production-last-mile)

You don't need every box checked on day one. But you should know which ones you're skipping and why.

## Contents

1. [Accounts and auth](#1-accounts-and-auth)
2. [Your users' data](#2-your-users-data)
3. [Payments](#3-payments)
4. [Secrets and config](#4-secrets-and-config)
5. [Calling AI models](#5-calling-ai-models)
6. [Deploys and domains](#6-deploys-and-domains)
7. [Knowing when it breaks](#7-knowing-when-it-breaks)
8. [Backups and recovery](#8-backups-and-recovery)
9. [Abuse and limits](#9-abuse-and-limits)
10. [Privacy and the boring legal stuff](#10-privacy-and-the-boring-legal-stuff)

---

## 1. Accounts and auth

- [ ] **Use a real auth provider** (Clerk, Auth0, Supabase Auth, Cognito, or your framework's built-in auth). Rolling your own login is where security bugs hide.
- [ ] **Password reset works end to end**, including the email actually arriving. Test it with an address that isn't yours.
- [ ] **Sessions expire and logout really logs out.** A shared computer shouldn't stay signed in forever.
- [ ] **Every API route checks who's calling.** The UI hiding a button isn't protection; the server has to say no.
- [ ] **Admin pages are locked down** to specific accounts, not just "anyone logged in".

## 2. Your users' data

- [ ] **Every query is scoped to the current user or team.** If you can change an ID in the URL and see someone else's data, that's the bug to fix first.
- [ ] **The database isn't reachable from the public internet**, or if it has to be, it's behind strong credentials and an IP allowlist.
- [ ] **File uploads are private by default.** Public storage buckets are one of the most common leaks I see.
- [ ] **You've decided what happens when someone deletes their account**, and the app actually does it.

## 3. Payments

- [ ] **Charges use idempotency keys**, so a double click or a retried request can't charge someone twice.
- [ ] **Webhooks are your source of truth** for what happened with a payment, not the response in the browser.
- [ ] **Webhook signatures are verified.** Otherwise anyone can tell your app a payment succeeded. ([Here's a Go example](https://github.com/BonnieDooneConsulting/stripe-webhooks-go).)
- [ ] **Failed renewals are handled**: what does the customer see, and when do they lose access?
- [ ] **Refunds and cancellations work** from your side, not just from the Stripe dashboard.
- [ ] **You tested with Stripe's test cards** for declines and 3D Secure, not just the one that always works.

More on this in [5 Stripe Integration Mistakes That Cost Startups Money](https://bonniedoone.ai/blog/stripe-integration-mistakes).

## 4. Secrets and config

- [ ] **No API keys in the code or the git history.** If one ever was, rotate it. Deleting the line doesn't remove it from history.
- [ ] **No secret keys in the browser or mobile app.** Anything shipped to a device can be read. Calls that need a secret go through your server.
- [ ] **Production and test keys are separate**, and production keys live only in production.
- [ ] **Config comes from environment variables or a secrets manager**, so the same build runs everywhere.

## 5. Calling AI models

- [ ] **Model calls happen on your server**, never directly from the client with your key.
- [ ] **There's a spending cap** at the provider and a per-user limit in your app. One runaway loop or one abusive user can run up a real bill overnight.
- [ ] **Timeouts and retries are set.** Model APIs are sometimes slow or briefly unavailable, and your app should say so politely instead of hanging.
- [ ] **You handle bad or empty output.** Validate anything you parse from a model before you act on it.
- [ ] **User input can't quietly change your instructions** in ways that matter (prompt injection). Don't let model output trigger actions like sending email or deleting data without checks.
- [ ] **You log enough to debug** (model, latency, token counts, errors) without logging people's private content unless you need to and have said so.
- [ ] **Switching models is a config change**, not a rewrite. Prices and models change often.

## 6. Deploys and domains

- [ ] **One command (or one push) deploys the app.** If deploying means remembering ten steps, someone will forget one.
- [ ] **HTTPS everywhere**, with certificates that renew on their own.
- [ ] **Staging or preview environments exist** so you can try changes somewhere that isn't production.
- [ ] **Database migrations run as part of the deploy** and you know how to roll one back.
- [ ] **"Works on my machine" is tested away**: the production build runs from a clean checkout with only the documented environment variables.

## 7. Knowing when it breaks

- [ ] **Errors are reported somewhere you'll see them** (Sentry, Honeybadger, or your platform's error tracking).
- [ ] **Logs are kept and searchable** for at least a couple of weeks.
- [ ] **An uptime check pings the app** and alerts you, not your customers, when it's down.
- [ ] **You get alerted on payment and webhook failures** specifically. Those are the ones that cost money.

## 8. Backups and recovery

- [ ] **Automated database backups are on.**
- [ ] **You've actually restored one.** A backup you've never restored is a guess.
- [ ] **You know who can access production** and you can remove someone quickly.

## 9. Abuse and limits

- [ ] **Sign-up and login are rate limited**, and sign-up has some bot protection.
- [ ] **Expensive endpoints are rate limited per user**, especially anything that calls a model.
- [ ] **Uploads have size and type limits.**
- [ ] **Free tiers have limits** that make abuse boring.

## 10. Privacy and the boring legal stuff

- [ ] **A privacy policy and terms of service exist** and match what the app really does, including which AI providers see user data.
- [ ] **You know where your data lives** and which third parties process it.
- [ ] **Cookie and tracking consent** is handled if you have users where that's required.
- [ ] **Users can export or delete their data** if they ask.

I'm not a lawyer, and this section is a reminder to think about these things, not legal advice.

---

## Using this list

Copy it into an issue, a doc, or your project board and work through it. If you want to suggest something I missed, open an issue or a pull request.

## License

[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Share it and adapt it, just link back here.

---

Built by [Bonnie Doone Consulting](https://bonniedoone.ai). Stuck between a working prototype and a live product? [That's exactly what I help with](https://bonniedoone.ai/services/ai-app-to-production).
