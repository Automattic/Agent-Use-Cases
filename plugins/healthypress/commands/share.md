---
description: Give a care team member access to the private health site, or change and revoke that access — private sites only
allowed-tools: mcp__wpcom__wpcom-mcp-site, mcp__wpcom__wpcom-mcp-user-management, mcp__plugin_healthypress_wpcom__wpcom-mcp-site, mcp__plugin_healthypress_wpcom__wpcom-mcp-user-management, mcp__wpcom__authenticate, mcp__wpcom__complete_authentication, mcp__plugin_healthypress_wpcom__authenticate, mcp__plugin_healthypress_wpcom__complete_authentication, AskUserQuestion, Skill, Bash
arguments:
  - name: action
    description: What to do — "invite", "list", "change", "revoke". If omitted, you will be asked.
    required: false
  - name: email
    description: Email address of the person, for invite / change / revoke.
    required: false
---

# Share with Your Care Team

Manages who else can see the health site. Load `wpcom-mcp-operations` before step 2.

## Step 0.5: Connect to the MCP server

If the only `wpcom` tools available are `authenticate` and `complete_authentication`, the server
isn't authorized yet. Call `authenticate`, open the returned URL in the user's browser with Bash
(`open` / `xdg-open` / `start`), tell them the grant is account-wide, and wait for them to approve.
See `wpcom-mcp-operations` for the full handshake, including the fallback when the callback doesn't
land. Don't send the user off to configure anything — do it for them.

## Step 1: The private-site gate

Read the site status. **If the site is not Private, STOP:**

> `<site>` is currently `<status>`, not Private. I'm not going to add people to a site whose privacy
> posture I can't confirm — and the read-only Viewer role only exists on private sites at all. Run
> `/healthypress:setup` first.

Do not list users, do not invite anyone. Then STOP.

## Step 2: Show the current state

List site users and pending invites. Show, for each: name or email, role, and whether the invite is
accepted or still pending. If there are pending invites older than a week, point them out — a
forgotten pending invite is an open door.

If `action` wasn't given, ask with `AskUserQuestion` what they want to do: invite someone · change a
role · revoke access · cancel or resend a pending invite · nothing, just looking.

## Step 3: Invite — explain the role honestly, first

The chosen posture for HealthyPress is **trusted full access: invite care team as Editor.** An Editor
can read the entire private timeline, which is the point.

Before sending any invite, say this in plain language and get an explicit yes:

> I'll invite `<email>` as an **Editor**. What that means concretely:
>
> - They can **read every record on the site**, including private posts — the whole timeline, not a
>   subset. There's no way to share part of it.
> - They can also **edit and trash your records**, including ones they didn't create. WordPress has
>   no read-only-private role, so full read comes bundled with full write. Trashed posts are
>   recoverable for 30 days.
> - They can upload media and edit the Health Summary page.
> - They cannot change site settings, change privacy, or add other users. That stays with you.
>
> The alternative is **Viewer**, which is read-only — but a Viewer sees only *published* content, so
> on this site they'd see the Health Summary page and none of the individual private records.
> That's the right choice for someone who should see the summary but not your whole history.

Offer both with `AskUserQuestion`: **Editor** (full read, can also edit) · **Viewer** (read-only,
Health Summary only) · **Cancel**.

Then send the invite for the chosen role. Confirm what was sent, to which address, in which role,
and that it's pending until they accept. If the invite fails, show the error and STOP — do not retry
with a different role.

## Step 4: Change a role

Show the current role, state what the new role can and cannot do using the same plain-language
framing as step 3, get an explicit yes, then change it and read the role back.

Downgrading Editor → Viewer means they immediately lose access to every individual record and keep
only the published Health Summary page. Say that before doing it, so it isn't a surprise when they call.

## Step 5: Revoke

Removing a user or cancelling a pending invite. Confirm the exact person first — an email address
typo here removes the wrong person.

After revoking, read the user list back and confirm they're gone. Then say what revocation does
*not* undo:

> `<email>` no longer has access. Two things that don't reverse: anything they already read or
> exported, and any copy they made. If they were an Editor, check `/healthypress:review` for records
> they edited or trashed.

## Step 6: Report

Print the resulting user list with roles and pending invites — the same shape as step 2, so the
before/after is comparable.

Once per run, whether or not anything changed:

> Sharing this site shares health information that has **no legal privacy protection** — HIPAA
> covers providers and insurers, not your own website. Anyone you invite can copy anything they can
> see. Invite the smallest number of people who need it, and re-run `/healthypress:share list`
> occasionally to check who still has access.

---

This command manages access to a site. It does not record, interpret, or advise on health
information.
