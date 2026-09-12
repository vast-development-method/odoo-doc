# Forum, blog and public profiles

The deep specification of the community capabilities published on a site: the discussion forum with its
questions, answers, comments, votes, tags, moderation and complete reputation economy; the blog with its
posts, tags, comments, archive navigation and subscription feed; the public user profiles; and the public
contact directory.

The entity fields are in [entities.md](entities.md) Part 4, the state tables in
[state-machines.md](state-machines.md) §§7–10, the rules in
[business-rules.md](business-rules.md) Part 3 and Part 4, and the arithmetic in
[calculations.md](calculations.md) §§21–23.

---

## 1. Forums

### 1.1 What a forum is

A forum is one discussion space. It carries its own reputation table, so two forums of the same site can
demand different scores for the same operation, its own answering mode, its own privacy, its own guidelines
page and its own welcome banner.

| Setting | Values | Effect |
|---|---|---|
| Answering mode | questions, with one answer per participant; discussions, with several | In questions mode a participant may post only one answer per question (WS-607). |
| Privacy | public, signed in, some users | Who may reach the forum and its content. The authorised group is cleared when the privacy becomes public or signed in (WS-580). |
| Default ordering | newest, last updated, most voted, relevance, answered | The order applied when the visitor has not chosen one. |
| Relevance parameters | a vote exponent, 0.8 by default, and a time decay, 1.8 by default | The relevance formula of [calculations.md](calculations.md) §22. |
| Sharing options | on by default | After posting, the participant is offered the social sharing actions. |
| Guidelines | filled at creation from the shipped template | The page reached from the forum header. |
| Welcome message | the shipped block | The banner shown until the visitor dismisses it; its shipped text contains the heading `Welcome!`, the sentence `Share and discuss the best content and new marketing ideas, build your professional profile and become a better marketer together.`, a button labelled `Sign up` and a dismiss button labelled `Dismiss`. |

### 1.2 The listing

The question listing shows the active questions the visitor may view (WS-605), ten per page.

| Control | Values |
|---|---|
| Sort | The five orders of the default ordering setting. |
| Filter | All questions; unanswered, that is without any answer; solved, that is with an accepted answer; unsolved, that is without one. |
| Scope | Mine; followed, that is questions the participant follows; tagged, that is questions carrying a tag the participant follows; favourites. |
| Tag | One tag, reached through the tag address. |
| Search | The term, matched through the site search with approximate matching. |

The side panel shows the five most used tags, the unused tags for a moderator, the forum statistics — the
number of questions, of views, of answers and of favourites — and the moderation counters when the visitor
may moderate.

### 1.3 Counters

The forum reports the number of active and closed questions, the summed view counters of those questions,
the summed answer counts, the number of those questions that at least one participant marked as a favourite,
the number of posts waiting for validation and the number of flagged posts. The last two are what the
moderation panel shows.

---

## 2. Questions, answers and comments

### 2.1 The three kinds of contribution

| Contribution | Stored as | Notes |
|---|---|---|
| Question | a Forum Post with no parent | Carries the title, the content and the tags. |
| Answer | a Forum Post whose parent is the question | Its title is `Re: ` followed by the question title. |
| Comment | a message on the discussion thread of a post | Not a post; it carries no vote and no acceptance. |

### 2.2 Asking

The procedure is §39 of [workflows.md](workflows.md). Three parts are easy to get wrong in a rebuild:

* the tag text mixes existing tags, given as identifiers, and new tags, given as an underscore followed by a
  name; a new name reuses an existing tag of the same forum when one carries that name, and otherwise
  creates one only when the participant may create tags (WS-585);
* the content is rewritten before storage: links receive the no-follow marker below the no-follow threshold
  (WS-601), and a picture, a link or a background picture below the editor threshold refuses the write
  (WS-602);
* a question by an author below the validation threshold is created **pending**, awards nothing and is
  visible only to its author and to the moderators (WS-604).

### 2.3 Answering

The answering threshold applies. Answering a closed or archived question is refused (WS-599). Posting an
answer posts the shipped new-answer message on the question and refreshes the question's last activity
moment, which is what the "last updated" ordering reads.

### 2.4 Commenting and converting

Commenting has its own thresholds, distinguishing one's own posts from other people's. A comment may be
converted into an answer and an answer into a comment, each with its own threshold and its own message
(WS-587, WS-588). Deleting a comment has its own threshold (WS-589).

### 2.5 Voting

A vote is a row carrying one of the three values `1`, `0` and `-1`, unique per participant and post
(WS-597). A participant may not vote on their own post and may not change somebody else's vote (WS-595,
WS-596). Voting the same way twice withdraws the vote by storing the value `0`; the row stays so that the
uniqueness constraint keeps one row per participant and post.

Every create and every change moves the author's reputation by the difference between the award of the new
value and the award of the old value ([calculations.md](calculations.md) §21.2). A participant whose present
vote is the opposite one may always withdraw it, whatever their score (WS-594).

### 2.6 Accepting

Accepting an answer sets its acceptance flag, clears the flag on every other answer of the question — only
one answer may be accepted — awards the acceptance award to the answer's author and the acceptance bonus to
the accepting participant, unless the two are the same person, and sets the question's answered flag.
Withdrawing the acceptance reverses all of it. Deleting an accepted answer withdraws both awards (WS-610).

### 2.7 Favourites

A participant may mark a question as a favourite, which adds them to the favourite list and increases the
favourite counter. The counter feeds three of the shipped badges.

### 2.8 Related questions

The related questions of a question are the at most five questions with the highest tag similarity, ordered
by similarity descending and then by last activity descending. The measure and a worked example are
[calculations.md](calculations.md) §23. A question with no tag has no related questions.

### 2.9 Structured description for search engines

A question that has at least one answer publishes a question-and-answer page description containing the
question, its accepted answer when there is one and at most five suggested answers. Each entry carries the
vote count, the publication moment, the public address and the author, and the author carries a link to the
public profile when that profile is published. The question additionally carries its title, its plain
content and its answer count.

---

## 3. The reputation economy

### 3.1 What reputation is

Reputation is one integer score per user account, held by
[learning, surveys and gamification](../learning-surveys-and-gamification/README.md). The forum is the
largest contributor to it: it awards points for activity and spends them as permission. A rebuild that
separates the two — a "score" and a "role" — will not reproduce the behaviour, because every gate reads the
same number that the awards move.

### 3.2 The awards

The complete award table, the vote arithmetic with its worked example, the acceptance arithmetic and the
closing deductions are [calculations.md](calculations.md) §21. In summary, with the shipped defaults:

| Event | Beneficiary | Points |
|---|---|---|
| A question becomes active | its author | +2 |
| A question is up-voted | its author | +5 |
| A question is down-voted | its author | −2 |
| An answer is up-voted | its author | +10 |
| An answer is down-voted | its author | −2 |
| An answer is accepted | its author | +15 |
| An answer is accepted | the accepting participant | +2 |
| A post is marked offensive, or a question is closed as spam or offensive | its author | −100, multiplied by ten for a first question closed as spam |
| The account validates its address while its score is zero | the account | set to 3 |

Every amount is a field of the Forum record, so a forum may be tuned without touching another.

### 3.3 Where the points are visible

The public profile shows the score, the rank derived from it, the badges earned and the reputation history.
The post shows the author's score beside their name, and their biography only above the biography threshold
and only when their profile is published (WS-603).

---

## 4. The gates

Every forum operation compares the participant's score with a threshold of the forum. The comparison is
"the score is greater than or equal to the threshold"; an administrator passes every gate.

| Operation | Threshold field | Default | Refusal |
|---|---|---|---|
| Ask a question | ask | 3 | WS-581 |
| Answer | answer | 3 | WS-582 |
| Edit an own post | edit own | 1 | WS-583 |
| Edit any post | edit all | 300 | WS-583 |
| Change the tag list | retag | 75 | WS-586 |
| Close or reopen an own question | close own | 100 | WS-584 |
| Close or reopen any question | close all | 500 | WS-584 |
| Delete or reactivate an own post | delete own | 500 | WS-592 |
| Delete or reactivate any post | delete all | 1000 | WS-592 |
| Create a tag | tag creation | 30 | WS-585 |
| Up-vote | up-vote | 5 | WS-594 |
| Down-vote | down-vote | 50 | WS-594 |
| Accept an answer on an own question | accept own | 20 | WS-593 |
| Accept an answer on any question | accept all | 500 | WS-593 |
| Comment an own post | comment own | 1 | WS-587 |
| Comment any post | comment all | 1 | WS-587 |
| Convert an own comment | convert own | 50 | WS-588 |
| Convert any comment | convert all | 500 | WS-588 |
| Delete an own comment | delete own comment | 50 | WS-589 |
| Delete any comment | delete all comments | 500 | WS-589 |
| Flag a post | flag | 500 | WS-590 |
| Have links followed by crawlers | no-follow | 500 | silent, WS-601 |
| Post a picture or a link | editor | 30 | WS-602 |
| Show a detailed biography | biography | 750 | silent, WS-603 |
| Ask without validation | validation | 100 | silent, WS-604 |
| Moderate | moderate | 1000 | WS-591 |

The derived permission flags of a post — may ask, may answer, may accept, may edit, may close, may delete,
may comment, may convert, may post, may flag, may moderate, may use the full editor, may up-vote, may
down-vote, may view, may show the biography — are computed per post and per current user from these
thresholds; the definitions are [entities.md](entities.md) §4.2.

The reputation table of a forum is published at its own address, so that a participant can see exactly what
each score unlocks.

---

## 5. Moderation

### 5.1 The queues

| Queue | Contains |
|---|---|
| Pending | Questions created below the validation threshold. |
| Flagged | Posts a participant reported. |
| Closed | Questions closed with a reason. |
| Offensive | Posts marked offensive, which are also archived. |

Each queue is a filtered listing of the forum, visible only above the moderation threshold.

### 5.2 The operations

The eight moderation operations — validate, refuse, flag, mark as offensive, close, reopen, delete and
reactivate — with their guards, their records and their reputation consequences are §41 of
[workflows.md](workflows.md) and the transition table of [state-machines.md](state-machines.md) §7.

Two answers are shaped for the caller rather than for a person: flagging answers with a marker that
distinguishes a moderator from an ordinary participant, a post already flagged answers with an
"already flagged" marker, and a post in any other state answers with a "not flaggable" marker.

### 5.3 Closing reasons

Closing offers the basic reasons and marking offensive offers the offensive reasons; both lists are shipped
and are in [configuration.md](configuration.md) §5.3. Two basic reasons carry a reputation consequence: the
offensive-remarks reason and the spam reason deduct the flagging award from the author, and reopening
restores it, with the tenfold rule for a first question closed as spam (WS-609).

### 5.4 Archiving

Writing the active flag on a post writes the same value on every answer of the post; writing it on a forum
writes it on every post of the forum, archived posts included (WS-608). A post in the offensive state is
archived, which is why it leaves every default query.

---

## 6. Tags

A Forum Tag belongs to one forum and its name is unique within it (WS-606). It carries a colour index and a
count of the active questions that use it, which is what orders the "most used" list. Tags have their own
public address, their own discussion thread — so a participant may follow a tag — and participate in the
site search.

Creating a tag below the tag-creation threshold is refused (WS-585); during tag parsing the new name is
silently dropped instead, so that a participant below the threshold can still ask a question with existing
tags.

---

## 7. Blogs

### 7.1 What a blog is

A blog is a publication channel with a name, a subtitle, a cover configuration, free content for its landing
page, an optional site restriction and a set of posts. Archiving a blog archives every post of that blog,
already archived ones included, and unarchiving it unarchives them (WS-522).

### 7.2 The listing

The listing endpoint accepts an optional blog, an optional comma-separated tag list, a page number and a
search term; the procedure, the redirections and the page size of twelve are §37 of
[workflows.md](workflows.md) and WS-538 to WS-543.

Three parts of the page are derived rather than stored:

* the **tag cloud** is the set of tags used by the posts of the shown blogs, split into categorised tags,
  sorted by category name in upper case, and uncategorised tags, sorted by tag name in upper case;
* the **archive navigation** groups the posts by publishing month and formats the month and year names in
  the visitor's language and time zone;
* the **counts** shown to a designer are the published and unpublished counts, which a designer may also use
  as a filter; a visitor who is not a designer sees only posts whose publishing date is past (WS-535).

### 7.3 Reading a post

The procedure, the wrap-around of the next post and the once-per-session view counting are §36 of
[workflows.md](workflows.md), WS-536 and WS-537. The counter is increased with a lock-free increment that
skips locked rows, so that two concurrent readers never block each other (WS-572).

### 7.4 Publishing

The publication machine is [state-machines.md](state-machines.md) §10. The publishing date is maintained by
WS-524: it is set to the current moment on publication and cleared on unpublication, but only when the post
has no publishing date or its publishing date is already past, so that a future publishing date — a
scheduled post — is never overwritten.

Publishing a non-archived post posts a message **on the parent blog**, rendered from the shipped new-post
template, with the post title as subject and the publication subtype, so that the blog's followers are
notified (WS-526).

### 7.5 The teaser

The teaser is the manual teaser when there is one, and otherwise the first 200 characters of the
whitespace-collapsed plain text of the content followed by an ellipsis ([calculations.md](calculations.md)
§6.1). Writing the teaser writes the manual teaser, with the source-language clearing of WS-532, which is
what prevents a translation from overwriting the source text.

### 7.6 Comments

Comments are messages of the comment kind on the post's thread. Read access on a post is enough to post a
message on it (WS-530). The public comment list contains the messages that are not internal and whose
subtype is not internal (WS-529). A reply to the publication notification is downgraded to an internal note,
so that blog followers are not notified of every answer (WS-527), and comments are never pushed to the
internal inbox (WS-528). A blog never discloses its recipients to each other (WS-544).

### 7.7 The subscription feed

The most recent posts of a blog, ordered by publishing date descending, are rendered into a syndication
document. The number of entries is the requested limit capped at 50 and defaulting to 15. The base address
of the blog is used to build absolute links, and the post content is converted to plain text.

### 7.8 Tags and tag categories

A Blog Tag carries a globally unique name (WS-520), an optional category and a colour index. A Blog Tag
Category carries a globally unique name (WS-521) and groups tags for the filter side bar. Tag slugs in an
address are normalised, and a request carrying several tags is reduced to the first (WS-539, WS-540).

---

## 8. Public user profiles

### 8.1 Viewing

Viewing another account's profile requires the account to be published and the viewer's reputation score to
reach the site's minimum, 150 by default. The two refusals render the denial page with
`This profile is private!` and `Not have enough karma to view other users' profile.`; a non-existing account
answers "not found"; an account always sees its own profile (WS-557).

The profile shows the account's name, its picture, its city and country, its personal site, its public
description, its reputation score, its rank, its published badges, its reputation history and its forum
activity: the questions asked, the answers given, the votes received and the favourites.

### 8.2 The avatar

The avatar endpoint serves only the four permitted picture sizes and elevates its rights only when the
account is published and its reputation score is strictly positive; any other requested field answers
"forbidden" (WS-558).

### 8.3 Editing

The account may write its name, its personal site, its electronic mail address, its city, its country and
its public description; the publication flag is writable only by the owner; an administrator may edit
another account (WS-559).

### 8.4 Address validation

A token is derived from the current day at midnight, a stored secret, the account identifier and the
address, so that it is valid for one day. The validation message is sent with a link carrying the token;
consuming a valid token for an account whose score is zero raises the score to 3, and any other case leaves
the score unchanged (WS-560). This is the smallest possible bootstrap: it lets a brand-new account reach the
asking threshold of 3 without any other participant's help.

### 8.5 The ranking

The public ranking lists the published profiles ordered by reputation score descending, with their rank
names and their badge counts, and offers a search by name.

---

## 9. The public contact directory

### 9.1 One contact

A contact is published by setting its publication flag; the change is tracked and posts a message with the
subtype for publication or for unpublication. Its public page carries the contact's name, its public
description, its short public description, its public tags, its address and a map. A visitor reaching a
stale slug is redirected to the current slug; a contact that is not published is reachable only by a
Restricted Editor, and otherwise the request answers "not found" (WS-550, WS-551).

### 9.2 The directory

The directory lists the contacts that are published and have an assigned partner, faceted by industry and by
country, filterable by a published Partner Website Tag and by a free-text term matched against the name, the
public description and the industry name. Twenty contacts are shown per page. When the addressed country has
no matching contact but other countries do, the country filter is dropped and the listing reports that it
fell back to every country (WS-552 to WS-554).

### 9.3 The map

The map frame renders at most the requested number of published contacts, 80 by default, with their name,
address, latitude and longitude, and links each marker to the contact's reference page. When an explicit
contact list is supplied, only the company contacts of that list are rendered; when neither an explicit list
nor a named condition is supplied, no contact is rendered at all (WS-555, WS-556).

The two map helpers added to Contact are a static map picture address built from the street, the city, the
postal code and the country with the configured mapping key, and an external map link built from the same
address parts.

---

## 10. How these capabilities use the rest of the folder

| Contract | Used by |
|---|---|
| The publication mixin and its per-site reading | Blog, Blog Post, Forum Post through its view rule, Contact, Partner Website Tag |
| The multi-site restriction | Blog, Forum |
| The search engine metadata mixin | Blog, Blog Post, Blog Tag, Forum, Forum Post, Forum Tag |
| The searchable contract | Blog, Blog Post, Forum, Forum Post, Forum Tag, Contact |
| The cover properties mixin | Blog, Blog Post |
| The page visibility options mixin | Blog Post |
| The site index enumeration | every public address above |
| The visitor tracking hook | every public page above |
| The slug grammar | every public address above |
