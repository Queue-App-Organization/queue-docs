# Users

## Primary Users

### Customer

The customer is a walk-in diner, usually on mobile, often arriving during a peak meal period with limited patience and limited context.

Customer goals:

- Join the waitlist without downloading an app.
- Know estimated wait time.
- Leave the crowded entrance area without losing their place.
- Receive WhatsApp updates.
- Tell the restaurant they are on the way.
- Cancel easily if plans change.
- Trust that the queue is fair.

Customer frustrations:

- No clear estimate.
- Being asked to stand near the host counter.
- Missing a call.
- Losing position because communication was unclear.
- Being told "10 minutes" repeatedly when the real wait is much longer.

Key customer requirement:

> Scan -> enter name, phone, party size -> join.

No account creation should be required.

### Host / Staff

The host or floor staff member runs the waitlist during live service. This is the highest-pressure user and the most important operational screen.

Staff goals:

- See all waiting parties quickly.
- Add walk-ins manually.
- Understand party size and wait duration.
- See table availability and cleaning status.
- Call the right party.
- Mark arrived, seated, skipped, cancelled, or no-show.
- Assign a party to a table.
- Override recommendations when needed.
- Work quickly during rush periods.

Staff frustrations:

- Switching between paper, WhatsApp, phone calls, and mental tracking.
- Customers repeatedly asking for updates.
- Hard-to-read dashboards during rush.
- Unclear table readiness.
- Large parties blocking smaller table turnover.

Key staff requirement:

> The staff dashboard must stay fast, dense, and operational during Saturday dinner rush.

### Restaurant Owner / Manager

The owner or manager does setup, reviews performance, and decides whether the product is worth paying for.

Owner goals:

- Set up the restaurant in under 10 minutes.
- Define operating hours and tables.
- Print or display QR codes.
- Monitor current waiting demand.
- Understand average wait, no-shows, cancellations, and peak periods.
- Improve table utilization.
- Reduce lost parties.

Owner frustrations:

- Tools that require staff retraining.
- Complex enterprise features before basic operations work.
- No data about why customers leave.
- Lack of visibility into busy periods.

Key owner requirement:

> The product must prove operational value before expanding into platform features.

## Secondary Users

### Restaurant Admin

Manages staff access, permissions, operating hours, sections, table inventory, WhatsApp templates, and billing settings.

### Product Owner / Founder

Owns market focus, product boundaries, roadmap tradeoffs, pricing decisions, and human approval gates.

### AI Product Organization

An internal AI-assisted team can plan and implement the application through shared state in Jira, GitHub, documentation, analytics, and CI/CD.

Initial recommended AI roles:

- Product Manager / Orchestrator.
- UX / Product Designer.
- Full-Stack Engineer.
- QA Engineer.
- DevOps / Platform Engineer.

Later roles:

- Restaurant Domain / Business Analyst.
- Analytics.
- Separate frontend and backend engineers when the codebase justifies it.

## User Priorities

| User | Highest priority | Product implication |
| --- | --- | --- |
| Customer | Accurate, low-friction waiting | Mobile web + WhatsApp + clear ETA |
| Staff | Fast operational control | Dense dashboard + simple queue/table actions |
| Owner | Better operations | Analytics + setup + table utilization |
| Founder | Focused product wedge | Avoid POS, loyalty, ordering, and enterprise sprawl early |
