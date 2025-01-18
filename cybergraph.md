```nu
let $cybergraph = {
    title: 'My cybergraph'
    author: maxim-uvarov
    updated: (date now)
    location: ('cybergraph.md' | path expand)
    status: wip
    hashes: {
        previous_version: {
            ipfs: ''
            sha512: ''
        }
    }
}

print ($cybergraph | to yaml)
```

Output:

```
title: My cybergraph
author: maxim-uvarov
updated: 2025-01-16 11:57:03.550784 -03:00
location: /Users/user/git/cybergraph/cybergraph.md
status: wip
hashes:
  previous_version:
    ipfs: ''
    sha512: ''
```

## Foreword

For humans in general it is easier to write in markdown:)
For nushell users often properly formatted `.nu` script can be a convenient and powerful way for communication, and on the huge need we can always use something like `numd`.
For Logseq users the GUI of the Logseq itself will do quite well.
For machines it will be much easier to write in data specific formats.
It is possible to convert all data between each other.
So here is the description.

## Information

### My projects

```nu
# this command will display your github organisations
> gh repo list nushell-prophet --json name,url | from json
```

#### Github

#### Video courses

##### Excel for Internet Marketing, 2017

##### Power BI for Internet Marketing, 2017-2019

The first course on the market, where I imagined how Power BI could be used to the tasks, that I had in my career previously. 

Even though professionals using Power BI might find my solutions demonstrated in videos not of the best practices, yet I believe they served as a good kickstart for using the tool.

And it worth nothing to say that examples of working with Search Enigne Statistics of a real ecommerce advertiser with good structured campaigns and performance were indispencible.

### My principles

#### Information exchange

I learned from dr.Dmitriy Shamenkov, that the Communication Process has phisiological roots. And it's appropriate to approximate the rules of phisiological proceses on the communication between pepole.

It's better to be precise in communication. I disclose even not favorable details about me to help create a more precise picture of me.

#### Any educational process includes trial an error

We just learn about the world by living, constantly making assumptions of the current situation and predictions on it. And in each moment of time we recieve feedback from the environment.

#### Favorite quote about my creative process

> "... anyone can do any amount of work, provided it isn't the work he is supposed to be doing at that moment."
> -- Robert Benchley, in Chips off the Old Benchley, 1949

I found this quote in one of the most insightful essays of all time for me: [structured procrastination](https://structuredprocrastination.com) by [John Perry](http://john.jperry.net).

### My favorite books

#### Man, the manipulator: the inner journey from manipulation to actualization. Shostrom, E. L. (1967).

I read the translation to Russian of this book in the age of 19 and it influenced much my world view.

#### What The Buddha Taught. Rahula, Walpola (1959).

I read this book in the age of 35 years old and just loved it for condensed knowledge.

#### I Am That: Talks with Sri Nisargadatta Maharaj, by translator Maurice Frydman (1973).

I read this book in the age of 36 and it is with me since then.

#### The Manual: What Women want and how to give it to Them, by W. Anton.

I never did a good join in approaching women even since I read this book. Though, I value its content very high: because of the style, deepness and systematic approach.


