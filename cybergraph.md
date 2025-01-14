```nu
$env.cybergraph = {
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

print $env.cybergraph
```

Output:

```
╭──────────┬──────────────────────────────────────────╮
│ title    │ My cybergraph                            │
│ author   │ maxim-uvarov                             │
│ updated  │ now                                      │
│ location │ /Users/user/git/cybergraph/cybergraph.md │
│ status   │ wip                                      │
│ hashes   │ {record 1 field}                         │
╰──────────┴──────────────────────────────────────────╯
```

## Foreword

For humans in general it is easier to write in markdown:)
For nushell users often properly formatted `.nu` script can be a convenient and powerful way for communication, and on the huge need we can always use something like `numd`.
For machines it will be much easier to write in data specific formats.
It is possible to convert all data between each other.
So here is the description.

## Information

### My projects

#### Github

#### Video courses

##### Excel for Internet Marketing, 2017

##### Power BI for Internet Marketing, 2017-2019

```nu
# this command will display your github organisations
> gh project list
```

### My principles

#### Information exchange

I learned from Dmitriy Shamenkov, that communication has phisiological roots. And it's appropriate to approximate the rules of phisiological proceses on the communication between pepole.

It's better to be precise in communication. I disclose even not favorable details about me to help create a more precise picture of me.

#### Any educational process includes trial an error

We just learn about the world by living, constantly making assumptions of the current situation and predictions. And in each moment of time we recieve feedback from the environment.

#### Favorite quote about my creative process

> "... anyone can do any amount of work, provided it isn't the work he is supposed to be doing at that moment."
> -- Robert Benchley, in Chips off the Old Benchley, 1949

I found this quote in one of the most insightful essays of all time for me: [structured procrastination](https://structuredprocrastination.com) by [John Perry](http://john.jperry.net).
