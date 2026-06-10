---
title: "Sharing and Feedback"
date: 2026-06-10
weight: 7
chapter: false
pre: " <b> 7. </b> "
---

### Honest thoughts

Full disclosure: this is a personal project I built to learn AWS seriously — and it turned out to be way more fun than expected.

I started as a HUTECH student with a Flutter app whose Express + MongoDB backend ran on my own PC — turn the PC off and the app dies. After this project, my app runs on Amazon's infrastructure, the server is permanently off, and the monthly bill costs less than a pack of instant noodles.

![Just a chill guy HUTECH](/images/7-Feedback/nh.jpg?width=350px)

*My mental state throughout the project: just a chill guy.*

Most memorable moment? Seeding 240 movies into DynamoDB at midnight and getting slapped with `ValidationException: Item size has exceeded the maximum allowed size`:

![Me after debugging](/images/7-Feedback/meo.png?width=400px)

*Me, 1 AM, after meeting the 400KB limit.*

But it passed. And the feeling of opening the app on a real phone, with the Node server completely OFF, watching the movie list stream in from Singapore:

![SIUUU](/images/7-Feedback/cr7.jpg?width=400px)

*When `GET /api/movies` returns 200 OK from Lambda. SIUUU.*

### Satisfaction level

**Very satisfied.** Especially when opening the Billing page:

![Stunned](/images/7-Feedback/ngao.png?width=350px)

*My face seeing the total AWS bill: ~$0.02/month.*

Serverless truly means "pay for what you use" — and a student doesn't use much.

### Points to improve (mine)

- Design the DynamoDB data model **before** migrating instead of fixing on the fly — would have saved me 3 re-seeds.
- Bookmarks/comments still live on the old backend — next goal is moving them to AWS too.
- Set the Budget alert **before** deploying, not after remembering it exists.

![Sigma](/images/7-Feedback/sigma.jpg?width=400px)

*The energy when deciding to kill off the 24/7 Express server.*

### Would I recommend this to friends? Why?

**Yes, 100%.** If you're an IT student with an existing personal project (any app or website), migrating it to AWS serverless is the most hands-on way to learn cloud I've ever tried — beats 10 hours of tutorials. You hit real errors, read real logs, and end up with a real product to show off.

![Showing off the project](/images/7-Feedback/rua-shopping.png?width=350px)

*Me, taking my freshly-deployed AWS project out to show everyone.*
