# The forum

Everything a replacement needs in order to reproduce the behaviour of a question-and-answer forum:
its two modes, its three privacy levels, the five post states, the whole reputation-point table,
voting, accepting, moderation, tags, favourites, ordering, relevance and the public pages.

## 1. Modes

| Mode | Meaning |
|---|---|
| `questions` | One answer may be marked as the accepted one. The interface presents the forum as a knowledge base. |
| `discussions` | Several answers coexist without an accepted one. The interface presents the forum as a conversation space. |

The mode is presentation only: nothing in the permission system depends on it. The
answer-accepting mechanism itself is always available and always gated by the thresholds.

## 2. Privacy and who sees what

| Privacy | Anonymous visitor | Signed-in visitor | Administrator |
|---|---|---|---|
| `public` | Reads the forum, its posts and its tags. | Reads everything. | Reads everything. |
| `connected` | Sees nothing. | Reads everything. | Reads everything. |
| `private` | Sees nothing. | Reads everything only when they belong to the forum's authorised group. | Reads everything. |
| empty | The forum belongs to a course; the course visibility governs access. | The same. | Reads everything. |

Setting the privacy to `public` or `connected` clears the authorised group automatically.

Beside privacy, three further filters apply to a post:

1. **The author's balance.** The view marker is false for a post whose author holds zero or fewer
   reputation points, unless the reader is the author or the reader may close posts. This is the
   mechanism that hides the output of an abusive account without deleting anything;
   [`LSG-115d`](business-rules.md#lsg-115d).
2. **The state.** A post in state `pending` is hidden from everybody except moderators and its own
   author; opening it otherwise answers not-found.
3. **The archiving marker.** An archived post is hidden from the default queries. Archiving is the
   operation the interface calls deleting.

When a forum belongs to a course, three additional rules replace the privacy rules: anonymous
visitors reach only the forums, posts and tags of published courses whose visibility is `public`;
signed-in visitors reach those of published courses whose visibility is `public` or `connected`,
plus those of the courses they are enrolled in; and Learning Officers reach everything.

## 3. Posts: questions and answers

A post with no parent is a **question**; a post with a parent is an **answer** to that question. The
hierarchy is one level deep; [`LSG-094`](business-rules.md#lsg-094) refuses anything deeper or
circular.

A comment is not a post at all: it is a message on the discussion thread of a post. Converting
between an answer and a comment is a first-class operation, described in
[`workflows.md`](workflows.md#workflow-18).

An answer created through the public pages receives the title `Re: <question title>`.

The five states, their transitions, their guards and their side effects are machine 3 of
[`state-machines.md`](state-machines.md#machine-3). Closing and reopening apply only to questions:
the operation returns without doing anything when any post in the set has a parent. Archiving a
question archives its answers; un-archiving reverses it; archiving a forum archives every one of its
posts.

**Deleting a post.** Refused by [`LSG-099`](business-rules.md#lsg-099). When the deleted post was
the accepted answer, the accepted-answer amount is removed from its author with the reason `The
accepted answer is deleted`, and the same amount is removed from the acting user with the reason
`Delete the accepted answer`.

## 4. The complete reputation-point table

Every value is configured per forum. Two families exist: what an event **grants** to somebody, and
what an action **requires** of the acting user. The full tables with their identifiers and defaults
are sections 19.3 and 19.4 of [`entities.md`](entities.md); the table below restates the grants with
their beneficiary, because the beneficiary is easy to get wrong.

| Event | Who receives | Default |
|---|---|---|
| A question by the author becomes active — at creation for a trusted author, at validation otherwise | The author | 2 |
| The author's question is upvoted | The author | 5 |
| The author's question is downvoted | The author | −2 |
| The author's answer is upvoted | The author | 10 |
| The author's answer is downvoted | The author | −2 |
| The acting user accepts somebody else's answer | The **acting user** | 2 |
| The author's answer is accepted by somebody else | The answer's author | 15 |
| The author's post is closed as offensive, closed as spam, or marked offensive | The author | −100 |

Rules that qualify the grants:

1. Accepting one's own answer moves nothing: both accept movements are skipped when the acting user
   wrote the answer.
2. Un-accepting an answer applies both movements multiplied by minus one, with the reasons `Accepted
   answer removed` for the author and `Remove validated answer` for the acting user.
3. The flagged amount is applied as stored. Because it is negative by default, closing as offensive
   removes points and reopening gives them back; the tenfold multiplier for a first-time author is
   [`LSG-CALC-022`](calculations.md#lsg-calc-022).
4. Reputation points earned on a forum are never taken back when a post is simply archived, only
   when the accepted answer is deleted.

### 4.1 Requirements at a glance

| Action | Own post | Anybody's post |
|---|---|---|
| Ask a question | 3 | — |
| Answer a question | 3 | — |
| Publish a question without moderation | 100 | — |
| Edit a post | 1 | 300 |
| Change the tags of a post | 75 | 75 |
| Close or reopen a post | 100 | 500 |
| Archive, un-archive or delete a post | 500 | 1000 |
| Accept an answer | 20 on one's own question | 500 |
| Comment a post | 1 | 1 |
| Convert between comment and answer | 50 | 500 |
| Delete a comment | 50 | 500 |
| Upvote | 5 | 5 |
| Downvote | 50 | 50 |
| Create a tag | 30 | 30 |
| Flag a post as offensive | 500 | 500 |
| Moderate | 1000 | 1000 |
| Post pictures and links at all | 30 | 30 |
| Post links that search engines may follow | 500 | 500 |
| Have one's biography displayed beside one's posts | 750 | 750 |

"Own post" means the acting user created the post; for accepting an answer it means the acting user
created the **parent question**; for converting or deleting a comment it means the acting user wrote
the **comment**. An administrator bypasses every requirement.

Two requirements have an escape hatch: an upvote is always allowed when it cancels the reader's own
downvote, and a downvote is always allowed when it cancels the reader's own upvote. This lets a
low-reputation member undo a mistake.

Every refusal follows the pattern `%d karma required to <action>.`; the complete list is section 9
of [`business-rules.md`](business-rules.md).

### 4.2 Content restrictions derived from the thresholds

- Below the follow threshold, every anchor in the submitted content is rewritten so its address is
  preserved and a no-follow marker is added. This never refuses the write.
- Below the editor threshold, submitting content that contains a picture element, an anchor with an
  address, or an inline style loading a background picture from an address, is refused by
  [`LSG-109`](business-rules.md#lsg-109).

## 5. Voting

One vote record exists per user and post; [`LSG-112`](business-rules.md#lsg-112) enforces it. The
value is `1`, `-1` or `0`; zero means a cancelled vote, and the record is kept rather than deleted.

The operation, the intent table, the resulting value and the checks are
[`workflows.md`](workflows.md#workflow-17). The reputation movement and the worked sequence are
[`LSG-CALC-021`](calculations.md#lsg-calc-021).

A non-administrator may never set the owner of a vote or its recipient, neither through the written
values nor through defaults inherited from the calling context; those keys are stripped
([`LSG-114`](business-rules.md#lsg-114)).

The vote total of a post is the sum of the values of its votes, therefore upvotes minus downvotes,
with cancelled votes contributing zero.

## 6. Accepting an answer

Only one answer per question may be accepted. Pressing accept on an answer:

1. Refuses with the status `own_post` when the reader wrote that answer.
2. Clears the acceptance marker on every **other** answer of the same question.
3. Toggles the acceptance marker on the answer itself.

Writing the marker is guarded by the accept permission, which uses the own threshold when the reader
asked the **parent question** and the all threshold otherwise;
[`LSG-100`](business-rules.md#lsg-100). The points moved are in section 4. The answered marker on the
question becomes true as soon as any of its answers is accepted, and the default post ordering puts
the accepted answer first.

## 7. Favourites

A signed-in user may bookmark a question. Bookmarking also subscribes the user to the question's
discussion thread; un-bookmarking deliberately leaves the subscription in place, so the user keeps
receiving answers until they explicitly unfollow.

The favourite count of a question is the number of users holding the bookmark. The favourite figure
of a forum counts the questions that have at least one bookmark, not the bookmarks themselves.

## 8. Moderation

Four queues exist, each reachable only by a reader whose balance reaches the moderation threshold;
anybody else receives not-found ([`LSG-115a`](business-rules.md#lsg-115a)).

| Queue | Contents |
|---|---|
| Validation | Posts in state `pending`. |
| Flagged | Posts in state `flagged`, ordered by last change descending, optionally narrowed by a title search. |
| Offensive | Posts in state `offensive`. |
| Closed | Posts in state `close`. |

From the flagged queue a moderator may mark a post offensive, which asks for a reason among the
reasons of the `offensive` kind, or accept the post back. A batch operation marks as offensive every
flagged post grouped by author, by author country, or by explicit selection, always using the spam
reason. Closing a post asks for a reason among the reasons of the `basic` kind.

A member who already has a question waiting for validation on a forum may not ask another one
([`LSG-115b`](business-rules.md#lsg-115b)), and asking requires a valid electronic mail address on
the account ([`LSG-115c`](business-rules.md#lsg-115c)).

## 9. Comments and notifications

Posting a comment on a post:

1. Requires the comment threshold; [`LSG-108`](business-rules.md#lsg-108).
2. On an answer, the followers of the parent question who subscribed to the comment subtype are
   added to the recipients, so a comment on an answer reaches the people watching the question.
3. The record name shown in the notification is the parent question's title when one exists.
4. The message is posted with elevated rights, so a reader who may comment but may not read the
   follower list still notifies them.
5. Comments never become inbox notifications; only outgoing messages are sent. This keeps the inbox
   usable on a busy forum.
6. The question's last-activity instant is refreshed.

Editing a post posts a notification: `Answer Edited` under the answer-edited subtype on the parent
question for an answer, and `Question Edited` under the question-edited subtype on the question
itself. The forum-level subtypes mirror the post-level ones, so a follower of the forum receives new
questions and new answers.

## 10. Tags

A tag belongs to one forum; [`LSG-111`](business-rules.md#lsg-111) makes the name unique inside a
forum, and the same name may exist on two forums as two records.

Typing tags on a post form produces a list of entries. Each entry that begins with an underscore is
a new label whose name follows the underscore:

1. When a tag of that name already exists on the forum, it is reused.
2. Otherwise a new tag is created, but only when the author's balance reaches the tag-creation
   threshold and the name is non-empty. Below the threshold the entry is dropped silently; creating
   a tag directly raises [`LSG-110`](business-rules.md#lsg-110).

Entries that do not begin with an underscore are existing tag identifiers and are kept.

The usage count of a tag is the number of active, non-archived posts carrying it. A forum exposes
two derived lists: the five most used tags, ordered by post count descending then by name then by
identifier; and the unused tags, those whose count is zero or empty.

The tag page lists the tags of a forum, optionally narrowed by a starting letter, by a search, and
by one of four filters: all, followed — the tags the reader subscribed to — most used, unused. The
ordering is by post count descending when a starting letter is active and by name otherwise. The
letters offered as a pager are the distinct uppercase first letters of the tags, sorted, prefixed by
the entry `All`. The two refusals of an invalid pager letter or filter are
[`LSG-115e`](business-rules.md#lsg-115e).

## 11. Listing and ordering questions

The question list of a forum applies, in order: the site filter, the state filter `active`, the
visibility filter, and then the chosen narrowing.

| Narrowing | Condition added |
|---|---|
| No answers yet | The post has no answer. |
| Solved | The answered marker is true. |
| Unsolved | The answered marker is false. |
| Mine | The reader created the post. |
| Followed | The reader's contact follows the post. |
| Tagged | The reader's contact follows one of the post's tags. |
| Favourites | The reader bookmarked the post. |
| Upvoted | The reader cast a vote on the post. |
| One forum | The forum is the chosen one. |
| One or several tags | The tag list holds any of the chosen tags. |
| Answers included | The restriction that the parent must be empty is dropped. |

Without the answers-included option, only questions are listed.

The five orderings offered are:

| Ordering | Sort key |
|---|---|
| Newest | Creation instant descending. |
| Last Updated | Last-activity instant descending. |
| Most Voted | Vote total descending. |
| Relevance | Relevance descending. |
| Answered | Answer count descending. |

The forum's default ordering decides which one applies when the visitor chooses none. An ordering
typed into the address is validated before use and dropped when it is not a legal sort expression.
The list is paged ten questions at a time.

**The relevance formula** is [`LSG-CALC-020`](calculations.md#lsg-calc-020).

**Related questions.** Under a question, up to five related questions are proposed, ranked by the
tag overlap of [`LSG-CALC-023`](calculations.md#lsg-calc-023), then by last-activity instant
descending. A question with no tag proposes nothing.

## 12. Forum statistics

The six figures and the last-post rule are
[`LSG-CALC-024`](calculations.md#lsg-calc-024). A question's view counter is incremented by one each
time its page is opened, without taking a lock.

## 13. Public profile pages

The forum and the learning platform share one public profile mechanism, described in
[`workflows.md`](workflows.md#workflow-22) and in section 34 of
[`entities.md`](entities.md). Three points belong to the forum specifically:

- A biography is shown beside a post only when the author holds at least the biography threshold and
  the profile is published.
- After a reputation gain the platform proposes destinations to the user; the forum adds `See our
  Forum` pointing at `/forum`.
- The badge and rank pages are reachable from the forum menu, so a member can see what the community
  rewards.

## 14. Shipped closing reasons

Thirteen reasons are shipped; they are listed with their kinds in
[`configuration.md`](configuration.md#shipped-reasons). Two of the basic reasons are special-cased
by the closing and reopening logic: the offensive reason and the spam reason move reputation points,
and the spam reason multiplies the amount by ten for a first-time author.

## 15. Shipped badges

Twenty-nine badges are shipped with the forum, twenty-eight of them driven by a challenge, together
with twenty-seven goal definitions and twenty-eight challenge lines. Every challenge uses the
periodicity `once`, the personal display, no periodic report, a real-time reward, the category
`forum` and a population filter selecting the users whose balance is strictly greater than zero.
Every goal definition counts records matching a filter, displays the result as a yes-or-no
indicator, uses the condition "the higher the better" and is evaluated in batch. The full catalogue
is [`configuration.md`](configuration.md#shipped-forum-badges).

## 16. Forums attached to a course

When the course-forum package is installed:

- A course may point at one forum, and a forum may serve only one course;
  [`LSG-063`](business-rules.md#lsg-063).
- Creating a course with a forum, or attaching a forum to a course, clears that forum's privacy,
  which hands access control over to the course rules.
- Detaching a forum from a course makes that forum private and restricts it to the Learning Officer
  group, so an orphaned course forum does not become publicly readable.
- A forum attached to a course takes its illustration from the course picture when it has none of
  its own.
- The forum's visibility mirrors the course's visibility.
- A counter of the forum's active posts is exposed on the course, and the course form offers an
  action that opens the forum.

## 17. Site integration

- A forum carries search-engine metadata; a post exposes its plain content as its page description
  and the author's picture as its social illustration.
- A question page emits structured data describing the question, its accepted answer and up to five
  suggested answers; nothing is emitted when the post is an answer, or when it has neither an
  accepted answer nor any suggested answer.
- Every active, viewable question appears in the site map, with its last change date.
- A counter of forums is maintained per site; a forum with no site is counted on every site.
- The site builder offers a suggested Forum menu entry pointing at `/forum`.
- The forum welcome banner is dismissed by a browser cookie.
