# GE 8 – AI Retrospective Analysis & Jira Tasks

An n8n workflow that collects sprint retrospective feedback through a web form, uses AI to analyze it, and automatically creates Jira tasks for each actionable improvement it finds.

- **Workflow ID:** `EbPvYCclv08TFX8k`
- **Status:** Not published (draft)
- **Jira project:** Golden Eagle
- **Issue type created:** Task

---

## How it works

```
Retrospective Form  →  Analyze Feedback (OpenAI)  →  Split Actionable Items  →  Create Jira Task
```

1. **Retrospective Form** – A team member submits sprint feedback.
2. **Analyze Feedback** – OpenAI (`gpt-4o-mini`) reviews the answers and returns structured JSON:
   - overall sentiment (`positive`, `neutral`, `negative`, `mixed`)
   - morale score
   - summary
   - feedback categories
   - actionable improvement items (title, description, priority, category)
3. **Split Actionable Items** – Turns each actionable item into its own record and adds the sprint, sentiment, morale and summary.
4. **Create Jira Task** – Creates one Jira Task per actionable item in the **Golden Eagle** project.

If the AI finds no actionable items, no Jira tasks are created.

---

## Form fields

| Field | Type | Required |
|---|---|---|
| Your Name | Text | Yes |
| Sprint / Iteration | Text (e.g. "Sprint 24") | Yes |
| What went well? | Long text | No |
| What didn't go well? | Long text | No |
| Suggestions for improvement | Long text | No |
| How would you rate team morale this sprint? | Dropdown (1 – Very low … 5 – Excellent) | Yes |

### Form URL

- **Production (after publishing):** https://infoagile.app.n8n.cloud/form/4ef2501b-245c-4c81-8699-4bd7acf39096
- **Testing:** open the workflow in n8n and use the form trigger's test URL. Test submissions create real Jira tasks.

---

## Jira task format

**Summary**
```
[Retro] <action item title>
```

**Description**
```
<action item description>

----
Sprint: <sprint>
Category: <category>
Suggested priority: <High | Medium | Low>
Team sentiment: <sentiment> (morale <score>/10)

Generated automatically from sprint retrospective feedback.
```

> The AI's suggested priority is written into the description only. The Jira *Priority* field itself is not set.

---

## Requirements

| Service | Credential | Used by |
|---|---|---|
| OpenAI | OpenAI API credential | Analyze Feedback |
| Jira Software Cloud | `Jira SW Cloud account 2` | Create Jira Task |

### Jira credential setup
- **Email:** your Atlassian account email
- **API token:** created at https://id.atlassian.com → Security → API tokens (not your password)
- **Domain:** `https://<your-site>.atlassian.net` (no trailing path)

The Atlassian account must have **Create issues** permission in the Golden Eagle project.

---

## Setup & usage

1. Make sure both credentials are connected and their tests pass.
2. In **Create Jira Task**, confirm Project = *Golden Eagle* and Issue Type = *Task*.
3. Submit a test response through the form and check that tasks appear in Jira.
4. Publish the workflow to activate the production form URL.
5. Share the production form link with the team after each sprint.

---

## Troubleshooting

| Error | Likely cause | Fix |
|---|---|---|
| `The target project doesn't exist or you don't have permission to create issues in it` | Jira credential is unauthorized (wrong email/token) or the account lacks project permission | Regenerate the API token, update the credential, check project permissions |
| `Parameter "Project" is required` / `"Issue Type" is required` | Fields were cleared, usually after a credential change | Re-select Project and Issue Type in Create Jira Task |
| Credential test returns `Invalid URL` | Domain is malformed | Use `https://<your-site>.atlassian.net` |
| No Jira tasks created | AI returned no actionable items | Expected behavior. Add more specific feedback or suggestions |

---

## Known limitations

- **Morale scale mismatch:** the form rates morale 1–5, but the AI scores it on a 1–10 scale, and the Jira description shows `/10`.
- Each form submission is analyzed on its own. Feedback is not combined across team members.
- Jira Priority, assignee, labels and sprint fields are not set automatically.
