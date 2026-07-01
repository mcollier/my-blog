---
title: "Events"
date: 2025-06-25T09:00:00-04:00
author: "Michael S. Collier"
description: "Speaking engagements at user groups, conferences, and community events."
slug: "events"
---

## Upcoming Events

{{< event-card
title="Building AI Agents with the Microsoft Agent Framework"
role="speaking"
location="Southfield, MI (In-person)"
date="2026-07-15"
registrationUrl="https://www.meetup.com/midotnet/events/314122553/"
registrationText="Register for the event"
status="upcoming"
>}}
TODO
{{< /event-card >}}

{{< event-card
title="AI Agent Workflows with Azure Functions (and Friends)"
role="speaking"
location="Mason, OH (In-person)"
date="2026-07-24"
registrationUrl="https://www.cincydeliver.org/Home/Index"
registrationText="Event details and registration"
status="upcoming"
>}}
TODO
{{< /event-card >}}

{{< event-card
title="AI Agent Workflows with Azure Functions (and Friends)"
role="speaking"
location="Grand Rapids, MI (In-person)"
date="2026-08-14"
registrationUrl="https://www.BeerCityCode.com"
registrationText="Event details and registration"
status="upcoming"
>}}
TODO
{{< /event-card >}}

## Past Events

{{< event-card
title="Microsoft Build //localhost:Columbus"
role="organizing"
location="Columbus, OH, USA (Hybrid)"
date="2026-06-20"
registrationText="Registration closed"
registrationUrl="https://developer.microsoft.com/en-us/reactor/events/27247/"
relatedBlogUrl=""
relatedBlogText=""
status="past"
>}}
Organizer and speaker, delivering a session on *From CLI to PR: Automating the path to merged code*.
{{< /event-card >}}

{{< event-card
title="Local Azure Meetup - Secretless Connectivity in Practice"
role="attending"
location="Cincinnati, OH, USA (In-person)"
date="2026-05-16"
registrationText="Registration closed"
status="past"
>}}
Demonstrated managed identity patterns and secretless connections across common Azure services.
{{< /event-card >}}

---

**Maintainer Notes**

- Add new engagements to **Upcoming Events**.
- After an event date has passed, move it to **Past Events**.
- Keep each event entry as an `event-card` shortcode with: title, role, location, date, registration info, notes.
- Use one of these role values: **Speaking**, **Organizing**, or **Attending**.
- Add `relatedBlogUrl` and optional `relatedBlogText` only when there is a related post.
- If there is no related post, omit those fields and the card will hide that row.
- Prefer ISO date format (`YYYY-MM-DD`) for consistency and easy sorting.
