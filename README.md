# trakt2feed

**Trakt2Feed** is a program that will take your Trakt viewing history
and turn it into an Atom web feed.

## Why?

I like to track my TV watching, so I've been a VIP member of
[Trakt](https://trakt.tv/) since early in 2019. I'm very happy to pay the
$30/year subscription for a couple of reasons - firstly, because it's nice to
support the project and, secondly (and more importantly), because VIP members
get web feeds of their watching history and I use that feed to power the
[Watching](https://dave.org.uk/watching/) page on my website. If you're
interested (and I realise that no-one is) then that page will show you the
most recent ten TV shows and films that I have watched.

Recently, Trakt have announced they are going to increase the annual
subscription from $30/year to $60/year. This has (rightly, in my opinion)
annoyed a lot of people.

* Doubling the subscription is really a bit rude
* In doing this, they have broken a promise that grandfathered pricing would be honoured for ever (some people were paying $15/year)

I know that prices need to rise sometimes. But I think that Trakt have handled
this in a ver bad manner. I immediately cancelled my subscription.

This left me with a problem. Not an immediate problem (because I had recently
paid my annual subscription), but at some point my web feed would vanish. So
I needed to replace it. I was prepared to move to a different service, but
I did some research and nothing seemed to really meet my needs.

Then I discovered [`traktexport`](https://github.com/purarue/traktexport)
on GitHub. This is a program that uses the Trakt API to produce a JSON
file containing your Trakt viewing history. And I realised that I could
easily write a program that takes this JSON file and turns it into a web
feed.

So that's what I did.

And you don't need a Trakt VIP subscription to access the Trakt API. So
I've recreated the main benefit (to me, at least) of the VIP subscription
for no cost.

## How?

You might also want a web feed of your Trakt watching history. Let me
explain how you get one.

First you need `traktexport`. If, like me you're going to run this on
GitHub Actions and GitHub Pages (I'll explain those later), you don't
need it installed permenantly. But you'll need it installed at first so
you can set up your authentication to the Trakt API.

The `traktexport` repo has instructions for installing the code and
authenticating to the API. It's a Python program, so you'll need Python
installed first - and that's beyond the scope of this document.

You install `traktexport` by running:

    $ pip install traktexport

The API authentication is explained in the
[`traktexport`](https://github.com/purarue/traktexport). It can be a little
tricky, but you only need to do it once. The basic steps are:

1. Register a new Trakt API application at https://trakt.tv/oauth/applications
2. Run `traktexport auth yourTraktUsername`
3. Follow the instructions - which involve opening a web address and typing a PIN

That creates a file called `~/.local/share/traktexport.json` (on Linux, at least).
This contains your authentication details and you'll need this information later.

At this point, you can test that everything is working by running:

    $ traktexport export yourTraktUsername > data.json

And you should end up with a file called `data.json` containing information
about your watch history.

Now you'll need my half of the process.

Go to my repo (https://github.com/davorg/trakt2feed) and press the "Fork"
button (you'll need a GitHub account). This will give you your own copy
of my repo in your GitHub account.

Clone your GitHub repo locally:

    $ https://github.com/YOUR-GITHUB-ACCT-NAME/trakt2feed.git

Oh, by the way, you'll need `git` installed :-)

That will create a directory called `trakt2feed` which contains a file called
`config.json`. You'll want to edit that to change the configuration data.

* **feed_title:** A title to use for the feed
* **feed_link:** The URL where you will be publishing the feed
* **feed_author:** Something that identifies you as the author of the feed

Commit that change:

    $ git commit config.json -m'Updated config'

And push it back to GitHub:

    $ git push

In order to run my code on your local machine, you'll need some Perl modules
installed:

    $ sudo apt-get update
    $ sudo apt-install cpanminus
    $ cpan --sudo --installdeps --notest .

That will churn away for a while and when it's finished, you should be able
to run my code:

    $ bin/mkfeed data.json

That will write the feed to the screen. Or, more usefully, you can write it
to a file:

    $ bin/mkfeed data.json > feed.xml

So if you wanted to stick those two commands in a crontab:

    traktexport export yourTraktUsername > data.json
    bin/mkfeed data.json > feed.xml

And then copy `feed.xml` to a web server somewhere, then you would have a
feed of your Trakt watch history.

But we can do better than that. We can set this up to run using GitHub
Actions and GitHub Pages.