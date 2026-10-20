---
name: plain
description: Use when the user types /plain, asks for something explained plainly or restated, says replies are too long, verbose, confusing, or full of jargon, file names, or ids, or asks to keep all later turns short. Also use when handing a technical finding to a non-technical audience.
---

# Plain

Plain language for a smart reader who has not seen the work happen and will not look anything up.

## Scope

- Bare `/plain`, or text supplied: rewrite the previous assistant message (or the supplied text) once.
- The user says "from now on", "all turns", "for this conversation", or similar: restate the previous message now, then apply the recipe to every later message in the session, including final reports and messages after tool runs. Length creeps back over a session, so check each message against the recipe before sending.

## Recipe

Every message is one paragraph of short sentences, built in this order:

1. **Answer.** First sentence: the verdict, the state of things, or the direct reply.
2. **Support.** Up to three sentences. Decide the ask first, then keep only the facts the answer or the ask depends on. Each fact gives a reason to trust the answer or says what the reader does about it.
3. **Ask.** Last sentence: the one decision or action that falls to the reader, or "Nothing needed from you."

Five sentences at most for a restatement or a report; three for anything else.

Things are named by what they do. Write "the invoice publisher's message channel", not `acme.invoice-publisher.upsert`. A label (story id, PR number, flag, endpoint, file path, queue or exchange name, error string) appears only when the reader must copy it into a command or search box next, and then it follows the plain description, once.

Names of people, numbers, dates, and who-owns-what stay; they are usually the point. Evidence trails, rejected alternatives, and the order Claude found things in are cut.

## Before sending

Check each message (not the text of a draft inside it) against four tests, and rewrite until all pass:

1. No line starts with a list marker or a heading. The message is one paragraph, even when the material it draws on is a list.
2. The first sentence is the answer itself, not a sentence announcing one.
3. Every backticked name, path, or id is something the reader will copy next. The rest are described by what they do.
4. The last sentence is the ask.

## When the reader is lost

If the user says "confused", "missing half the story", "walk me through", or "assume I know nothing", the problem is missing context, not length. Open with what the thing is in plain words, then why it exists, what happened, what is next. Define each unfamiliar term at first use. Six sentences at most, ask included, still one paragraph.

## Drafts for approval

The draft is the text itself, in the format its destination expects, as short as it can be while keeping the point. One sentence after it asks for approval.

## Examples

Restating a status report. Before (220 words, abridged): "Preflight done. Story 1234 Ready for PROD; `newRoleNames` on in TST, off in PRD (`apps/billing-api/prd/values.yaml:41`); `POST /invoices/resync-all` required post-deploy else `invoiceChanged` hits the old router…"

> The release is not ready to cut. Two checks are open: whether the invoice publisher's message channel exists in production, and whether the reviewer's approval on the billing PR survived the force-push. Everything else checks out. Should I run those two checks before drafting release notes?

Reporting after a tool run. Before: "Exchange `acme.orders.sync` exists, queue `orders-api.sync.v2` bound, 3 consumers, 0 msgs/hr; flag `ordersSync.enabled` false in `apps/orders/prd/values.yaml`; `ha-all` policy mirrors the queue."

> The order-sync channel is set up and has a consumer, but nothing flows because its switch is still off in production. Should I draft the change that turns it on?
