Any programmer worth his salt hates repeating code.  That's why we find situations that force us to repeat ourselves grating.  [Today, Web Development Sucks](http://harry.me/2011/01/27/today-web-development-sucks/) begrudges the pains of having to repeat validations both client- and server-side.

Sadly, this is not a web issue but a software issue.  For several years now I've been developing a client-server WinForms application that deals with the same greivous pain.  See, all programmers that care about a good user experience will also care about performance.  And so to reduce roundtrips, we repeat some logic on the client.

If we unreservedly trusted the client, couldn't we simply shift our backend validations to the frontend?  I don't think so.  The backend must assume that anything the client sends is untrustworthy.    Malicious threats aside, it's a matter of maintaining data integrity and probably the best place to do that is as close to the database as we can get--in fact, [right at the database](http://97things.oreilly.com/wiki/index.php/Database_as_a_Fortress).  The database is, after all, the last line of defense.  If our data gets compromised so does our business.  Thus, we've learned not to rely on validations we can't guarantee.

Where does this leave us?  We enforce our rules on our trusty server.  But since we care about user experience, that leaves us where, again?  With repeating logic on the client.

I've [pondered this](http://stackoverflow.com/questions/2556070/how-do-you-keep-your-business-rules-dry) repeatedly as project after project we've dealt with at least duplicating some validations.  Recently, I've been working on a [personal project](https://github.com/mlanza/thingy).  Same issue.  I thought I might try again to address it.  This lead me to one possibly good solution: *metarules*.

1. Represent your validations (rules) using metadata.
2. Write server- and client-side interpreters that enforce them.

In this way, the repetitious bit is in writing the interpreters.

Take the following model:

    class Person < ActiveRecord::Base
      validates_presence_of :first_name
    end

We code our validations right on the model.  But what if we didn't?  What if that validation was stored as metadata in the database?  Let's forget for the moment that databases already support this by allowing us to mark fields not nullable.  What we want is a more universal approach.  What if we persisted our validations as data (in the database or elsewhere) and any given tier was equipped to abide by that data?

    TYPE      ATTRIBUTE       RULE
    --------  --------------  -----------------------------
    person    first_name      required

The server-side model could then interpret and enforce those metarules.  The same thing could happen client side just by outputting our rules as data--not code--to the client:

    {
      "attribute_rules": [{
        "type": "person",
        "attribute": "first_name",
        "rule": "required"
      }]
    }

What we've done is traded hardcoded logic for interpretive data.  We enforce our rules on both the front and back ends in a DRY manner.  This allows the client to avoid needless roundtrips to the server.  It allows the server to choke out bad data if the client rules are circumvented.  Now to universally effect validations we're left to maintain our metadata and our interpreters.

Any sort of rule can be universally enforced so long as we provide:

1. Rules represented as data
2. Interpreters <sup>1</sup> that understand and enforce them

I dunno but it seems to me that a schemaless database (like MongoDB) would be a perfect repository for these kinds of rules.  Anyway, as I continue to work through this, I'm sure I'll learn some things and have more to say.

I haven't found any good articles addressing this problem.  If you have, please provide links.  I'd love to learn what others are doing.  I'd be especially gratified to see well-used design patterns make this less an issue, especially as I too weary of repeating validations.  I love DRY code and architectures.  I always fret over having to maintain the same logic twice.  My gut tells me I'm inviting trouble.

<ol class='footnotes'>
<li>I see no reason why you couldn't use a compiler instead of an interpreter.</li>
</ol>