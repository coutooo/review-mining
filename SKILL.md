---
name: review-mining
description: Turns App Store and Play Store reviews into a ranked, buildable backlog. Fetches reviews for an app, clusters the complaints, weights them by how much revenue they threaten, and writes each one as a task an agent can actually implement. Use when deciding what to build next, when reviews are piling up, or when a rating has dropped.
---

# Review mining

Your users already wrote your roadmap. It is sitting in your reviews, unsorted.

This turns that into a ranked backlog.

## Step 1. Collect

Fetch reviews for the app. Take the most recent 200, or everything since the last release, whichever is larger.

Keep for each: rating, date, app version, country, and the text.

If you were given an App Store link, use it. If you were given a bundle id, resolve it first. Do not proceed on a guess about which app this is.

### Where the reviews come from

Apple exposes reviews as public JSON. No key, no auth.

```
https://itunes.apple.com/{country}/rss/customerreviews/page={n}/id={appId}/sortby=mostrecent/json
```

`{appId}` is the digits after `id` in the App Store URL. `{country}` is a two letter code, lowercase, `us` by default.

Pages go from 1 to 10, fifty reviews each, so five hundred per country is the ceiling. If you need more, walk the countries the app actually sells in rather than trying to page past ten.

Each entry gives `im:rating`, `im:version`, `title`, `content`, `author` and `updated`. Reviews only appear here once Apple has surfaced them, so the newest day or two is usually thin. Do not treat that as a drop in review volume.

If a page returns an empty `feed.entry`, the app has no reviews in that storefront. Say so and move to the next country rather than reporting zero overall.

## Step 2. Separate signal from noise

Discard, but count:
- Reviews with no text
- Reviews about price alone, with no other complaint
- Reviews that are clearly about a different app

Report the counts. A wall of "too expensive" with nothing else is itself a finding, and it is not a bug.

## Step 3. Cluster the complaints

Group by the underlying cause, not by the words used. "It logged me out", "lost all my data" and "had to sign in again" are usually one bug.

For each cluster give:
- A one line name for the problem
- How many reviews it appears in
- The rating spread of those reviews
- Whether it started at a specific app version
- Two verbatim quotes, unedited

The version correlation is the most valuable column. A cluster that starts at one version is a regression with a known cause, not a feature request.

## Step 4. Rank by revenue at risk, not by count

Order the clusters by:

1. **Crashes and data loss.** Always first. Nothing else matters while these exist.
2. **Blocks on the paying path.** Anything between a user and giving you money. Broken restore purchases, a paywall that will not dismiss, a subscription that does not unlock.
3. **Onboarding failures.** Complaints from users on their first session. These never become paying users, so they are invisible in your revenue data and lethal.
4. **Missing features asked for repeatedly.** Only counts if the same thing appears at least five times.
5. **Everything else.**

A one star review from someone who could not sign up costs you more than a three star from someone who has been paying for a year.

## Step 5. Write it as work, not as feedback

For each cluster in the top ten, output a task an agent can pick up cold:

- What is happening, in behavioural terms
- Which files or subsystems are most likely responsible, if the repo is available
- How to reproduce it, inferred from the reviews
- What "fixed" looks like
- A suggested reply to the reviewers once it ships

Never output "improve stability" or "polish onboarding". If you cannot say what to change, say the cluster needs reproduction first and say what information is missing.

## Step 6. The reply list

Reviewers who reported something you have now fixed are the cheapest rating recovery available. Most developers never reply.

Output the list of review ids worth replying to, grouped by cluster, with a short reply for each. Say what shipped, in one sentence, without marketing language.

## Output format

1. Counts: total reviews read, discarded, clustered
2. The cluster table, ranked
3. The top ten as tasks
4. The reply list
5. One paragraph: if you only did one thing this week, this is it

## Rules

- Never invent a review or a quote. Quote verbatim or do not quote.
- Never rank by count alone. Ten people annoyed by a colour matter less than two who lost data.
- If the reviews are overwhelmingly positive, say so and stop. Do not manufacture problems to fill a report.
