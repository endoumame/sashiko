# Review Report Template

## Instructions

When regressions are found, create `review-inline.txt` following this template.

## Formatting Rules

1. **Plain text only** - no markdown, no backticks for code
2. **Wrap at 78 characters** - except code snippets
3. **No line numbers** - use function/method names and file paths
4. **No dramatic language** - factual descriptions only
5. **Suitable for GitHub PR comment** or code review tool

## Template

```
Subject: Re: [PATCH] <original subject line>

I found potential issues in this patch:

=== Issue 1: <Brief description> ===

File: <filename>
Class/Function: <ClassName::methodName()> or <function_name()>
Severity: <low|medium|high|critical>

The change introduces <describe the issue>.

Current code:

    <code snippet showing the problem>
    <use indentation, no backticks>

The issue is that <explain why this is wrong>.

This can be triggered when <describe the condition>.

Suggested fix:

    <code snippet showing fix if known>

---

=== Issue 2: <Brief description> ===

[Repeat format for each issue]

---

Analysis notes:
- <Any relevant context>
- <Call paths traced>
- <Assumptions verified>
- <PHP version considerations>
```

## Example

```
Subject: Re: [PATCH] Add user profile update endpoint

I found potential issues in this patch:

=== Issue 1: SQL injection via raw query ===

File: app/Services/UserService.php
Class/Function: UserService::searchUsers()
Severity: critical

The change constructs a raw SQL query by concatenating user input
from the search parameter.

Current code:

    public function searchUsers(string $query): Collection
    {
        return DB::select(
            "SELECT * FROM users WHERE name LIKE '%" . $query . "%'"
        );
    }

The issue is that $query comes from user input via the controller
and is directly concatenated into the SQL string without
parameterization or escaping.

This can be triggered by any authenticated user sending a crafted
search query like: ' OR 1=1 --

Suggested fix:

    public function searchUsers(string $query): Collection
    {
        return DB::select(
            "SELECT * FROM users WHERE name LIKE ?",
            ['%' . $query . '%']
        );
    }

---

=== Issue 2: Missing null check on optional relation ===

File: app/Http/Controllers/ProfileController.php
Class/Function: ProfileController::show()
Severity: high

The change accesses the profile relation without checking for null.
The User model defines profile() as a hasOne relation which can
return null for users without a profile.

Current code:

    public function show(User $user)
    {
        $bio = $user->profile->bio;
        return view('profile.show', compact('bio'));
    }

The issue is that $user->profile can be null for users who have
not created a profile, causing "Attempt to read property on null"
error.

This can be triggered when viewing any user without a profile.

Suggested fix:

    $bio = $user->profile?->bio ?? '';

---

Analysis notes:
- Traced call path from Route to Controller to Service
- Verified $query parameter originates from Request::input()
- Confirmed User::profile() is hasOne (nullable)
- Checked no global scope forces profile creation
```

## Checklist Before Writing Report

- [ ] Issue verified against false-positive-guide.md
- [ ] Code path is actually reachable
- [ ] Concrete evidence provided (code snippets)
- [ ] No speculative issues included
- [ ] Formatting rules followed
- [ ] Framework-specific patterns considered

## File Creation

Create the file in the current directory:
```
./review-inline.txt
```

Verify the file exists after creation.
