---
title: 'research!rsc: Electoral Programming'
url: https://research.swtch.com/electoral
published: "2008-06-13T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/electoral
---

# research!rsc: Electoral Programming

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# Electoral Programming Russ Cox June 13, 2008 *research.swtch.com/electoral* Posted on Friday, June 13, 2008.

A few days ago, the web site [fivethirtyeight.com](http://www.fivethirtyeight.com) posed the question of how many ways there were to win the electoral college with a minimal set of states, that is, sets of states that add up to 270 or more votes, but for which removing any state from the set would drop under 270.

Isabel Lugo, a mathematician, reached into his toolbox and pulled out the most trusted tool in combinatorics: generating functions. His [recent blog post](http://godplaysdice.blogspot.com/2008/06/fuller-solution-to-tuesdays-electoral.html) gives a nice explanation.

I'm a programmer, so I reached into my toolbox and pulled out my favorite tool for combinatorics: dynamic programming. Dynamic programming is an odd name for what is essentially cached recursion, minus the recursion.

Lets consider the simpler problem of counting the number of ways to add up to exactly 270 electoral votes. We can write a simple recursive equation to compute the number of ways:

```
ways(n, v) = ways(n-1, v-votes[n]) + ways(n-1, v)

```

`Ways(n, v)` is the number of ways, using just the first `n` states, to get `v` votes. Either you include state `n`'s votes or you don't. If you do, then you need `v-votes[n]` votes from the first `n-1` states, and if you don't, then you need `v` votes from the first `n-1` states. `Ways(n, v)` is just the sum of the number of ways to achieve either of the two simpler scenarios. (There is of course a base case: using the first 0 states, there's one way to get 0 votes, and no way to get any other number of votes. There's also no way, using any number of states, to get a negative number of votes.)

This makes for a simple recursive program:

```
    int64
    ways(int n, int64 v)
    {
        if(n == 0 && v == 0)
            return 1;
        if(v < 0 || n <= 0)
            return 0;
        return ways(n-1, v-votes[n-1]) + ways(n-1, v);
    }

    printf("ways to get exactly 270 = %lld\n", ways(51, 270));

```

(The District of Columbia doesn't get any congressmen, but it does get electoral votes, hence 51. The discussion above talked about `votes[n]` as though `votes` was 1-indexed, but in C it is 0-indexed, hence `votes[n-1]`.)

Each call to `ways(n, *)` results in two calls to `ways(n-1, *)`, making for an unfortunate O(2N) run-time; since N=51, it's worth our time to write a faster program.

There are 251 function calls but only 52\*271 different possible function arguments; thus the recursive program repeats the same calls billions and billions of times. If we add a cache, we can avoid the repetition:

```
    int64 cache[52][271];   // initialized to -1

    int64
    ways(int n, int64 v)
    {
        if(n == 0 && v == 0)
            return 1;
        if(v < 0 || n <= 0)
            return 0;
        if(cache[n][v] == -1)
            cache[n][v] = ways(n-1, v-votes[n-1]) + ways(n-1, v);
        return cache[n][v];
    }

    printf("ways to get exactly 270 = %lld\n", ways(51, 270));

```

Now, after the initial call to `ways`, there are at most two calls to fill each cache entry, or at most 2\*52\*271 calls. That will run *much* faster.

We could stop here, but we can simplify the code even further by removing the recursion and filling in the cache via iteration.

```
    int64 ways[52][271];    // initialized to 0

    ways[0][0] = 1;
    for(n=1; n<=51; n++){
        for(v=0; v<=270; v++){
            ways[n][v] = ways[n-1][v];
            if(v-votes[n-1] >= 0)
                ways[n][v] += ways[n-1][v-votes[n-1]];
        }
    }

    printf("ways to get exactly 270 = %lld\n", ways[51][270]);

```

It's important that at the time the iteration is computing `ways[n][v]`, it has already computed the entries in ways it will need. Since `ways(n, *)` depends on `ways(n-1, *)` it suffices to fill in the whole row `ways[n-1]` before starting on `ways[n]`. It's not necessary to iterate over `v` from 0 to 270. We could go from 270 to 0:

```
    int64 ways[52][271];    // initialized to 0

    ways[0][0] = 1;
    for(n=1; n<=51; n++){
        for(v=270; v>=0; v--){
            ways[n][v] = ways[n-1][total];
            if(v-votes[n-1] >= 0)
                ways[n][v] += ways[n-1][v-votes[n-1]];
        }
    }

    printf("ways to get exactly 270 = %lld\n", ways[51][270]);

```

In fact, `ways[n][v]` depends on ` ways[n-1][u]` only for `u <= v`, so if `v` iterates down from 270 to 0, we can reuse a single table row (we can also take the opportunity to replace `n-1` with `n`, now that `n` doesn't index into `ways` anymore):

```
    int64 ways[271];    // initialized to 0

    ways[0] = 1;
    for(n=0; n<51; n++)
        for(v=270; v>=votes[n]; v--)
            ways[v] += ways[v-votes[n]];

    printf("ways to get exactly 270 = %lld\n", ways[270]);

```

Many dynamic programming solutions have this property, that you only need array entries below and to the left, so that if you iterate from the right you can keep just a single row. For this problem, the space savings is not significant, but in some problems it is.

Now we've got a pretty simple, straightforward way to compute the number of ways to get 270 votes, but that wasn't the original question. The original question was how many ways there are to get at least 270 votes but with a minimal set of states.

We could compute the number of ways to get 270 votes, and 271, and 272, etc., but for the larger counts, we need to make sure only to include ways that use a minimal set. We can ensure minimality by using state `n`'s votes only if the total is not big enough already:

```
    int64 ways[400];    // initialized to 0

    ways[0] = 1;
    for(n=1; n<=51; n++)
        for(v=270+votes[n]-1; v>=votes[n]; v--)
            ways[v] += ways[v-votes[n]];

```

The `+=` will never add in `ways[u]` for any `u >= 270`. `Ways[v]` is the number of ways to get exactly `v` votes with a set of states that is minimal with respect to 270. To get the number of ways to get at least 270 votes, sum the end of the array:

```
    total = 0;
    for(v=270; v<400; v++)
        total += ways[v];

    printf("minimal ways to get at least 270 = %lld\n", total);

```

The upper bound of 400 is just a number that is big enough: no minimal winning set could have more than 400 votes.\*

There's one subtlety here: we only add in state `n`'s votes if those votes put the total count over 270, but maybe state `n` has 55 votes and the total is currently 269. Adding in state `n` puts the total over 270, but there must be smaller states already in the 269, so it's not a minimal set. To avoid such a situation, simply consider each state in size order, from most votes to least. Then when we're adding a state, the sets being considered can only contain bigger states, so the code above does compute the desired answer.

For concreteness, there is a complete C program below. It runs in about 80 microseconds on my Thinkpad X40. That's a lot faster than waiting out essentially any O(251) would have been.

I think it's neat that there are two such very different ways to think about the same computation: the abstract functional elegance of generating functions, and the imperative directness of dynamic programming. People who are much more comfortable with one approach or the other can pick the one that suits them. Personally, I'm one of the people Lugo supposed would say that the “approach via generating functions is just dynamic programming with a bunch of extra symbols floating around for no good reason.”\*\*

\* In fact, since the state with the most votes has 55, no minimal winning set could have more than 324 votes, but since you can't get 269 votes using bigger states, 324 is impossible for a minimal set. Thus the actual upper bound is even smaller. In fact, if you add up the states with the most votes, you need the first 11 to get to 271, and the state that seals the victory has 15. Thus it would suffice to use 284 as the upper bound instead of 400. This kind of digression is exactly the reason that 400 is good enough!

\*\* A similar correspondence is the one between [static single analysis (SSA) form
and continuation-passing style (CPS)](http://citeseer.ist.psu.edu/139456.html). It's easy to see the SSA advocates saying that CPS is just SSA with a bunch of extra lambdas floating around for no good reason!

```
#include

typedef long long int64;

int votes[51] = {
    55, 34, 31, 27, 21, 21, 20, 17, 15, 15,
    15, 13, 12, 11, 11, 11, 11, 10, 10, 10,
    10,  9,  9,  9,  8,  8,  7,  7,  7,  7,
     6,  6,  6,  5,  5,  5,  5,  5,  4,  4,
     4,  4,  4,  3,  3,  3,  3,  3,  3,  3,
     3,
};

int64 ways[400];

int
main(int argc, char **argv)
{
    int n, v, reps;
    int64 total;

    for(n=0; n<400; n++)
        ways[n] = 0;

    ways[0] = 1;
    for(n=0; n<51; n++)
        for(v=270+votes[n]-1; v>=votes[n]; v--)
            ways[v] += ways[v-votes[n]];

    total = 0;
    for(v=270; v<400; v++)
        total += ways[v];

    printf("%lld\n", total);
    return 0;
}

```

(Comments originally posted via Blogger.)

- [Isabel Lugo](http://www.blogger.com/profile/15671307315028242949)(June 13, 2008 6:32 AM) Thanks! I'm glad to see that somebody documented this.

And to be honest, your solution or something like it might be what's running under the hood of my solution; in the end I outsourced the computational work to Maple, and I don't know what it's doing.

- [Jack](http://www.blogger.com/profile/04805435564360543720)(June 15, 2008 2:12 PM) Isn't O(2^51) = O(1)?

- [Stephan Schroevers](http://www.blogger.com/profile/03746679755202178161)(June 15, 2008 4:35 PM) Jack, yes it is. But *2^51* relates only to the American situation, not to a general instance of the problem, in which there are *n* states, giving runtime *O(2^n)*, not *O(2^51) = O(1)*.

Note that even though a function *f(n)* may be dominated by *g(n)* from some *n\_0* onward, it may be profitable to select a *O(g(n))* algorithm over a *O(f(n))* algorithm if one knows that typical inputs will have size *n < n\_0*.

- [S](http://www.blogger.com/profile/07860792846652677912)(June 16, 2008 11:23 AM) The real issue is not how well Obama or McCain might do in the closely divided battleground states, but that we shouldn't have battleground states and spectator states in the first place. Every vote in every state should be politically relevant in a presidential election. And, every vote should be equal. We should have a national popular vote for President in which the White House goes to the candidate who gets the most popular votes in all 50 states.

The National Popular Vote bill would guarantee the Presidency to the candidate who receives the most popular votes in all 50 states (and DC). The bill would take effect only when enacted, in identical form, by states possessing a majority of the electoral vote -- that is, enough electoral votes to elect a President (270 of 538). When the bill comes into effect, all the electoral votes from those states would be awarded to the presidential candidate who receives the most popular votes in all 50 states (and DC).

The major shortcoming of the current system of electing the President is that presidential candidates have no reason to poll, visit, advertise, organize, campaign, or worry about the voter concerns in states where they are safely ahead or hopelessly behind. The reason for this is the winner-take-all rule which awards all of a state's electoral votes to the candidate who gets the most votes in each separate state. Because of this rule, candidates concentrate their attention on a handful of closely divided "battleground" states. Two-thirds of the visits and money are focused in just six states; 88% on 9 states, and 99% of the money goes to just 16 states. Two-thirds of the states and people are merely spectators to the presidential election.

Another shortcoming of the current system is that a candidate can win the Presidency without winning the most popular votes nationwide.

The National Popular Vote bill has been approved by 18 legislative chambers (one house in Colorado, Arkansas, Maine, North Carolina, Rhode Island, and Washington, and two houses in Maryland, Illinois, Hawaii, California, and Vermont). It has been enacted into law in Hawaii, Illinois, New Jersey, and Maryland. These states have 50 (19%) of the 270 electoral votes needed to bring this legislation into effect.

See http://www.NationalPopularVote.com

- [Russ Cox](http://swtch.com/~rsc/)(June 16, 2008 12:10 PM) @s: Feel free to head over to the political blogs. This blog only cares about the electoral college insofar as it leads to interesting programming problems.

I'm not saying your comment is right or wrong, just that it's out of place here.

- [Monstre](http://www.blogger.com/profile/03975843196252061184)(August 8, 2009 11:02 AM)This post has been removed by the author.

- [Monstre](http://www.blogger.com/profile/03975843196252061184)(August 8, 2009 11:04 AM) nice one !!never thought it was so easy ..

check out mine ..and comment if u are interested .. www.spyfree.info

- [Fiddler](http://www.blogger.com/profile/15331041685611149300)(March 26, 2011 4:07 PM) Minor typo:

ways\[n\]\[v\] = ways\[n-1\]\[total\];

should be

ways\[n\]\[v\] = ways\[n-1\]\[v\];
