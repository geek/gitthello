Gitthello
====

Synchronize Github issues with Trello.

This still work in progress (i.e. there are no tests) but it does seem to
work if the planets are in the correct alignment.

Why?
====

There wasn't a good alternative for synchronizing existing issues across
multiple repos. [Zapier](http://zapier.com) can do this partially but only
for new issues and also it's very impractical if you have several repos
that you want to sync (one zap for one event per repo).

Github has Trello integration but that only creates cards for commits and
pull requests but we wanted issues.

Features
====

Gitthello runs on [Heroku](http://heroku.com) using the free [Heroku Scheduler](https://devcenter.heroku.com/articles/scheduler). It doesn't not require
a dyno, so it doesn't cost anything to have this sync run every 10 minutes
from Heroku.

What gets synchronized?
----

This only synchronizes issues as cards, i.e. each issue becomes an card
in Trello and every new card in trello becomes an issue at github. Comments
are not synchronized.

It's bidirectional, meaning that a new issue created in any repo will create
a new todo card in trello. A new card in trello will create a new issue at
github. Closing an issue at github will move the corresponding card to the
done list in trello and moving a card to the done list in trello will
close the corresponding github issue.

Issues that change (i.e. title or description) don't get update in trello,
i.e. the cards, but a new card isn't recreated either.

Backlog issues (i.e. those issues marked with a backlog label) are created
in the backlog list.

How does it work?
----

To card at Trello an URL attachment is made with the name 'github'. If an
card has an attachment, it represents that issue and reflects the state of
that issue.

Installation
====

Once the [environment](https://github.com/wooga/gitthello/blob/master/.env.sample) has been defined, it's a matter of pushing to
heroku and done.

Configuration
----

First setup the environment. Use [.env.sample](https://github.com/wooga/gitthello/blob/master/.env.sample) as a guide and create a .env file with the following:

    GITHUB_ACCESS_TOKEN='abcde....'

You'll need an [Github access token](https://github.com/settings/applications).
A personal access token with repo access does the job.

    TRELLO_DEV_KEY=somekey'

The trello developer key can be [obtained here](https://trello.com/1/appKey/generate#), it's the key not the secret.

    TRELLO_MEMBER_TOKEN='somelongerkey'

Trello member key can be generated by checking out [this post](http://stackoverflow.com/questions/17178907/how-to-get-a-permanent-user-token-for-writes-using-the-trello-api/17301115#17301115). You'll only need to replace the key parameter
with your developer key, application name is irrelevant.

For each trello board to be synchronized, a seperate configuration is
required. The complete configuration for a board is:

    BOARDS[ONE][NAME]='Name of the Board'
    BOARDS[ONE][REPOS_TO_CONSIDER]='owner/repo_one,owner/repo_two'
    BOARDS[ONE][REPO_FOR_NEW_CARDS]='owner/card_repo'

Bascially BOARDS is a hash of hashes where the top-level key (ONE) is
arbitrary and the attribute names (NAME, REPOS_TO_CONSIDER and
REPO_FOR_NEW_CARDS) are fixed.

    BOARDS[ONE][REPOS_TO_CONSIDER]='owner/repo_name,owner2/repo_name3,...'

This is a comma-separated list of all repos from issues are taken. Each repo
is defined by the owner and the repo_name, e.g.

    https://github.com/wooga/gitthello

Here the owner is 'wooga' and the repo_name is 'gitthello'. You can have as
many repos as you like, but you need to have access to those repos.

    BOARDS[ONE][REPO_FOR_NEW_CARDS]='owner/repo_name'

When a new card is created, a new issue will be created in this repo and linked
to the card. Here you can only define a single repo. The issue created won't
be re-added to trello and the card will be updated with the github issue
URL.

    BOARDS[ONE][NAME]='My board name'

Name of the board where all this action should take place. This board needs
some predefined lists for all this to work.

A second board gets a new top-level key, e.g.

    BOARDS[TWO][NAME]='Other name of the another Board'
    BOARDS[TWO][REPOS_TO_CONSIDER]='owner/repo_three,owner/repo_four'
    BOARDS[TWO][REPO_FOR_NEW_CARDS]='owner/card_repo_two'

If the repos to be considered, i.e. REPOS_TO_CONSIDER, overlap between boards,
then cards for issues will be sync'ed into both boards. Not ideal and should
be avoided.

If the repo for new cards overlap, then all issues for new cards will be
created there, might be desirable.

Trello Board Requirements
----

Gitthello assumes that the trello board has at least three lists

    To Do - where all new issues go when sync'ed
    Done - where cards are moved to once they have been completed
    Backlog - where issues are created to if they are labeled as backlog

The names of these lists can be set to something other than the default above by setting the environment variables:

    BOARDS[ONE][TODO_LIST_NAME]='To do list name'
    BOARDS[ONE][BACKLOG_LIST_NAME]='Backlog list name'
    BOARDS[ONE][DONE_LIST_NAME]='Done list name'

Besides those three lists, a board can have as many lists as it likes. Any
card created in any list (except done) will create a new issue @ github.

Testing
====

To run the synchronization task locally:

    rake sync

This will do the synchronization between github and trello. This is also
the task that you need to set for the heroku scheduler.

Limitations
====

There are lots.

Comments are not synchronized. Labels are not synchronized. And lots of other
things aren't synchronized!

License
====

Released under the GPLv2.

See https://www.gnu.org/licenses/gpl-2.0 for details.

Contributing to Gitthello
====

* fork the project
* start a feature branch
* make sure to add tests
* please try not to mess with the Rakefile, version, or history
