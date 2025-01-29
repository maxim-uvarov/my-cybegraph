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
# My cybergraph

The information about me written as a showcase of possibilities of using cybergraphs.

## Information

### My projects

#### Cybergraph applications

For humans in general, it is easier to write in markdown. :)

For Nushell users, a properly formatted `.nu` script can often be a convenient and powerful way to communicate, and when there is a huge need, we can always use something like `numd`.

For Logseq users, the GUI of Logseq itself will do quite well.

For machines, it will be much easier to write in data-specific formats.

It is possible to convert all data between each other.

So here is an example to show why this conversion might be useful.

#### Github

```nu
# this command will display your github organisations
> gh repo list nushell-prophet --json name,url | from json
```

#### Video courses

##### Excel for Internet Marketing, 2017

##### Power BI for Internet Marketing, 2017-2019

The first course on the market, where I imagined how Power BI could be used for the tasks that I had in my career previously.

Even though professionals using Power BI might find my solutions demonstrated in videos not to be the best practices, I believe they served as a good kickstart for using the tool.

And it is worth noting that examples of working with Search Engine Statistics from a real e-commerce advertiser with well-structured campaigns and performance were indispensable.

### Ideas about myself that have appeared to be relevant to the current stage of this work

#### Information exchange

Back in 2017, I learned from Dr. Dmitriy Shamenkov that the communication process has physiological roots. This means that it is appropriate to apply the rules of physiological processes to communication between people.

It's better to be precise in communication. I disclose even unfavorable details about myself to help create a more accurate picture of me.

#### Any educational process includes trial and error

We just learn about the world by living, constantly making assumptions about the current situation and predictions based on it. In each moment of time, we receive feedback from the environment.

#### Favorite quote about my creative process

> "... anyone can do any amount of work, provided it isn't the work he is supposed to be doing at that moment."
> -- Robert Benchley, in Chips off the Old Benchley, 1949

I found this quote in one of the most insightful essays of all time for me: [Structured Procrastination](https://structuredprocrastination.com) by [John Perry](http://john.jperry.net).

### My favorite books

#### Man, the manipulator: the inner journey from manipulation to actualization. Shostrom, E. L. (1967).

I read the Russian translation of this book at the age of 19, and it significantly influenced my worldview.

#### What The Buddha Taught. Rahula, Walpola (1959).

I read this book at the age of 35 and just loved it for its condensed knowledge.

#### I Am That: Talks with Sri Nisargadatta Maharaj, by translator Maurice Frydman (1973).

I read this book at the age of 36, and it has been with me since then.

#### The Manual: What Women Want and How to Give It to Them. W. Anton.

I never did well in approaching women even since I read this book. However, I value its content very highly because of the style, depth, and systematic approach.

