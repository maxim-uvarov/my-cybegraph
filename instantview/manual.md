# Instant View Manual
This document describes Telegram's Instant View format in **stupefying detail**. If this is your first time hearing about Instant View, please check out our [**Introduction**](https://instantview.telegram.org/) and [**Sample Templates**](/samples/) before you dive in.

If you're comfortable with the idea of Instant View templates, let's look at what makes them tick. To begin with, this is our in-house artist's idea of how Instant View pages are generated:

[![](https://instantview.telegram.org/file/811140938/2/FF1p2pxErHM.79819/a582b1940327a15468 "How Instant View Pages are made, click for hi-res version")](/file/811140069/3/HNOgxR07ql4.872234/f83bfd30da8845e186)

It turns out, he is not entirely wrong. This is how Instant View pages are **really** generated:

1.  Whenever Telegram needs to display a link preview for a URL, it also checks whether an **Instant View template** exists for that domain.
2.  If a template exists, our Instant View Bot obtains the page using the URL (it only processes pages that have the MIME-type `text/html`).
3.  The bot then applies **rules** from the template that determine the positioning of the key elements on the source page (using [XPath 1.0](https://www.w3.org/TR/xpath/) expressions) and modifies the page content to fit the **Instant View format**.
4.  The edited page is used to create a new **Instant View page** that will be displayed to the user.

This document will explore the [Instant View Format](#instant-view-format), the [Types of Rules](#types-of-rules) your template can utilize, as well as some useful [XPath constructs](#extended-xpath), [conditions](#supported-conditions), and [functions](#supported-functions) that may help you build better templates.

This manual was intended to be used as a reference, so you don't have to read the entire thing to get started. Our [sample templates](https://instantview.telegram.org/samples) make for a much better entry point. You can check back here whenever something is not clear.

### [](#recent-changes)Recent changes

#### [](#september-10-2020)September 10, 2020

*   Added the new [~allowed\_origin](#allowed-origin-new) option
*   Added the new [@load](#load-new) function
*   The [@inline](#inline) function now supports the `silent` parameter which ignores errors while fetching a page

#### [](#march-20-2019)March 20, 2019

**Version 2.1**

*   Supported the `srcset` attribute in **Image** and **Icon** types. The IV engine will automatically take the highest image resolution available (but not higher than 2560px).
*   The [@match](#match) function now returns only nodes which content was matched by regular expression
*   The [@replace](#replace) function now returns only nodes which content was replaced by regular expression
*   The [@inline](#inline) function no longer follows canonical redirect links while fetching a page

**Also in this update:**

*   Added the new [@split\_parent](#split-parent) function

#### [](#december-10-2018)December 10, 2018

**Instant View 2.0** expands the platform with support for right-to-left languages, new page blocks, useful functions and much more. We recommend using the latest version in your templates. To do this, add the following rule **at the beginning** of the template:

```
~version: "2.0"
```


Below is a list of changes in version 2.0:

New [properties](#properties):

*   kicker
*   site\_name

New [types](#supported-types):

*   Map
*   Table
*   Details
*   RelatedArticles
*   Marked
*   Subscript
*   Superscript
*   Icon
*   PhoneLink
*   Reference

New [types of rules](#types-of-rules):

*   [Options](#options)
*   [Block functions](#block-functions)

New [functions](#supported-functions):

*   [wrap\_inner](#wrap-inner)
*   [style\_to\_attrs](#style-to-attrs)

Other features and improvements:

*   Xpath query results can be appended into variables using `+` ([see more](#variables))
*   New XPath function [ends-with](#ends-with)
*   Lists can contain block elements such as paragraph, nested lists, tables and so on
*   Support for attribution in media captions (`<cite>` tag)
*   Support for image links (`href` attribute for the Image type)
*   Unsupported elements (such as an image inside the blockquote) now generate an error instead of moving up
*   Improved `@simplify` function for better processing RichText (e.g. saves line breaks formed by block elements)

### [](#instant-view-format)Instant View Format

An Instant View page is an object with the following [**properties**](#properties):



* Parameter: title Required
  * Type: RichText
  * Description: Page title
* Parameter: subtitle
  * Type: RichText
  * Description: Page subtitle
* Parameter: kicker 
  * Type: RichText
  * Description: Kicker
* Parameter: author
  * Type: String
  * Description: Author name
* Parameter: author_url
  * Type: Url
  * Description: Author link
* Parameter: published_date
  * Type: Unixtime
  * Description: Date published
* Parameter: description
  * Type: String
  * Description: A short description (used in link preview)
* Parameter: image_url
  * Type: Url
  * Description: Link preview photo (used in link preview)
* Parameter: document_url
  * Type: Url
  * Description: Link preview document (used in link preview)
* Parameter: site_name 
  * Type: String
  * Description: Name of website (used in link preview)
* Parameter: channel
  * Type: String
  * Description: The username of the article author's (or the originating website's) channel on Telegram in the format @username
* Parameter: cover
  * Type: Media (Image/Video/Embed/Map)
  * Description: Page cover
* Parameter: body Required
  * Type: Article
  * Description: Page content


#### [](#rtl-support)RTL support

The platform supports right-to-left languages. If `<html>` tag or tag referenced by body property has the attribute `dir="rtl"`, the page will be marked as RTL. In case this attribute is not set, you can set it manually to display the page in Instant View as RTL using the following rule:

```
  @set_attr(dir, "rtl"): $body   # if body is already defined
  @set_attr(dir, "rtl"): /html   # alternative way
```


#### [](#supported-types)Supported types

The IV page object can hold the following **types**:



* Type: Article
  * Allowed children: Header                     Subheader                     Paragraph                     Preformatted                     Divider                     Anchor                     List                     Blockquote                     Pullquote                     Media                     Image                     Video                     Audio                     Embed                     Slideshow                     Table                     Details                     Footer                     RelatedArticles
  * Description: Page content
  * HTML counterpart: <article>
* Type: Header
  * Allowed children: RichText
  * Description: Major heading
  * HTML counterpart: We use the top-level of the <h1> – <h4> headings found on the source page
* Type: Subheader
  * Allowed children: RichText
  * Description: Minor heading
  * HTML counterpart: We use the remaining headings <h1> – <h4> as well as <h5> – <h6> headings
* Type: Paragraph
  * Allowed children: RichText
  * Description: A paragraph
  * HTML counterpart: <p>
* Type: Preformatted
  * Allowed children: RichText
  * Description: Preformatted text
  * HTML counterpart: <pre> with the optional attribute data-language*
* Type: Anchor
  * Allowed children: –
  * Description: Anchor
  * HTML counterpart: <anchor> with the attribute name that contains the anchor name
* Type: Divider
  * Allowed children: –
  * Description: A separator
  * HTML counterpart: <hr>
* Type: List
  * Allowed children: ListItem
  * Description: A list
  * HTML counterpart: <ul> for a bullet list, <ol> for a numbered list
* Type: ListItem
  * Allowed children: version 1.0:                     RichText                     version 2.0:                     Header                     Subheader                     Paragraph                     Preformatted                     Divider                     Anchor                     List                     Blockquote                     Pullquote                     Media                     Image                     Video                     Audio                     Embed                     Slideshow                     Table                     Details
  * Description: A list item
  * HTML counterpart: <li>
* Type: Blockquote
  * Allowed children: RichText                     QuoteCaption
  * Description: A block quote
  * HTML counterpart: <blockquote>
* Type: Pullquote
  * Allowed children: RichText                     QuoteCaption
  * Description: A pull quote
  * HTML counterpart: <aside>
* Type: QuoteCaption
  * Allowed children: RichText
  * Description: Caption of a quote
  * HTML counterpart: <cite>
* Type: Media
  * Allowed children: Image                     Video                     Audio                     Embed                     Map                     MediaCaption
  * Description: Media content
  * HTML counterpart: <figure>
* Type: Image
  * Allowed children: –
  * Description: An image
  * HTML counterpart: <img> with the attribute src and the optional attribute href to make the image clickable. Allowed formats: GIF, JPG, PNG (GIF would be converted into Video type by IV)As of version 2.1, supports the attribute srcset (srcset has higher priority than src, the same as in a browser; IV gets the highest available resolution under 2560px).
* Type: Video
  * Allowed children: –
  * Description: A video
  * HTML counterpart: <video> with the attribute src, or containing the tag <source type="video/mp4"> with the attribute src
* Type: Audio
  * Allowed children: –
  * Description: An audio
  * HTML counterpart: <audio> with the attribute src, or containing the tag <source> with the attribute src and the attribute type (possible types: audio/ogg, audio/mpeg, audio/mp4)
* Type: Embed
  * Allowed children: –
  * Description: An embedded element
  * HTML counterpart: <iframe> with the attribute src. List of supported embeds
* Type: Map 
  * Allowed children: –
  * Description: A map
  * HTML counterpart: <img> or <iframe> with the attribute src referring to Google or Yandex maps
* Type: Slideshow
  * Allowed children: Media (Image/Video)                     Image                     Video                     MediaCaption
  * Description: A slideshow
  * HTML counterpart: <slideshow>
* Type: MediaCaption
  * Allowed children: RichText
  * Description: Media caption
  * HTML counterpart: <figcaption>                                                               As of IV 2.0, can contain an additional tag <cite> with attribution (author, creator or source of the media)
* Type: Table 
  * Allowed children: TableCaption                        TableRow
  * Description: A table
  * HTML counterpart: <table>
* Type: TableCaption 
  * Allowed children: RichText
  * Description: A table caption
  * HTML counterpart: <caption>
* Type: TableRow 
  * Allowed children: TableCell
  * Description: A table row
  * HTML counterpart: <tr> with the optional attributes align, valign, can be wrapped with <thead> or <tbody> or <tfoot> with the optional attributes align, valign
* Type: TableCell 
  * Allowed children: RichText
  * Description: A table cell
  * HTML counterpart: <td> for a standard cell, <th> for a header cell, with the optional attributes align, valign, colspan, rowspan
* Type: Details   (version 2.0+)
  * Allowed children: DetailsHeader                     Header                     Subheader                     Paragraph                     Preformatted                     Divider                     Anchor                     List                     Blockquote                     Pullquote                     Media                     Image                     Video                     Audio                     Embed                     Slideshow                     Table                     Details
  * Description: A collapsible block
  * HTML counterpart: <details> with the optional attribute open to be opened by default
* Type: DetailsHeader   (version 2.0+)
  * Allowed children: RichText
  * Description: A visible heading for the collapsible block
  * HTML counterpart: <summary>
* Type: RelatedArticles 
  * Allowed children: Header                        Link
  * Description: Related articles
  * HTML counterpart: <related> containing one of <h1> – <h6> tag as header of the block and <a> tags with the attribute href containing a URL of related article. Note: Only articles that have an IV will appear in this block on the IV page.
* Type: Footer
  * Allowed children: RichText
  * Description: The footer
  * HTML counterpart: <footer>
* Type: RichText
  * Allowed children: Bold                     Italic                     Underline                     Strike                     Fixed                     Marked                     Subscript                     Superscript                     Icon                     Link                     EmailLink                     PhoneLink                     LineBreak                     Anchor                     Reference                     String
  * Description: Formatted text
  * HTML counterpart: Text elements and supported tags
* Type: Bold
  * Allowed children: RichText
  * Description: Bold text
  * HTML counterpart: <b> or <strong>
* Type: Italic
  * Allowed children: RichText
  * Description: Italic text
  * HTML counterpart: <i> or <em>
* Type: Underline
  * Allowed children: RichText
  * Description: Underlined text
  * HTML counterpart: <u> or <ins>
* Type: Strike
  * Allowed children: RichText
  * Description: Strikethrough text
  * HTML counterpart: <s> or <del> or <strike>
* Type: Fixed
  * Allowed children: RichText
  * Description: Monospace text
  * HTML counterpart: <code> or <kbd> or <samp> or <tt>
* Type: Marked 
  * Allowed children: RichText
  * Description: Marked text
  * HTML counterpart: <mark>
* Type: Subscript 
  * Allowed children: RichText
  * Description: Subscript text
  * HTML counterpart: <sub>
* Type: Superscript 
  * Allowed children: RichText
  * Description: Superscript text
  * HTML counterpart: <sup>
* Type: Icon 
  * Allowed children: –
  * Description: A small image inside the text
  * HTML counterpart: <pic> with the attribute src, width, height. Can contain the attribute optional if the image is not important and may be ignored. Allowed formats: JPG, PNG.As of version 2.1, supports the attribute srcset (srcset has higher priority than src, the same as in a browser).
* Type: Link
  * Allowed children: RichText
  * Description: A link
  * HTML counterpart: <a> with the attribute href containing a URL
* Type: EmailLink
  * Allowed children: RichText
  * Description: A link to an email address
  * HTML counterpart: <a> with the attribute href containing a mailto: link
* Type: PhoneLink 
  * Allowed children: RichText
  * Description: A link to a phone number
  * HTML counterpart: <a> with the attribute href containing a tel: link
* Type: Reference 
  * Allowed children: RichText
  * Description: A reference
  * HTML counterpart: <reference> with the attribute name that contains the reference name
* Type: LineBreak
  * Allowed children: –
  * Description: Line break
  * HTML counterpart: <br>


##### [](#note-on-code-languages)Note on code languages

Telegram apps currently do not support code highlighting, but they will in the future. For this reason, it is advisable to include the code language attribute (`data-language`) for large `<pre>` blocks if it is supplied in the source.

* * *

### [](#types-of-rules)Types of Rules

Instant View rules are instructions that tell our IV bot where to find the meta-elements on the source page and how it should modify the content of the page when an Instant View page is created.

Each new line in a template describes a new rule. When the bot renders a page into the Instant View format, it applies rules from the relevant template one after another and ignores any empty lines. You can leave comments by starting a line with `#`, all following text on that line will be ignored unless enclosed in quote marks. You can use the `\` symbol to carry a rule over to the next string, like this:

```
# Comment 
# You can break up \
  a long rule into \
  multiple strings
```


Most rules are based on [XPath 1.0](https://www.w3.org/TR/xpath/) expressions used to locate the required nodes on the source page.

A block of rules may have a name. In this case, it can be reused in other rules.

We support the following types of rules:

#### [](#conditions)Conditions

Conditions offer unlimited flexibility for your templates. Rules of this type begin with the symbols `?` or `!` and use the following format:

```
?condition:  xpath_query   # condition example
!condition:  regexp        # parameter on the right depends on the type of the condition
?condition                 # some conditions don't have parameters
```


Groups of conditions that immediately follow one another are interpreted as a block. `?`\-rules follow the OR logic when joined, while `!`\-rules follow the AND logic. This means that for the bot to apply each block, all `!`\-conditions in the block and at least one of the `?`\-conditions within it must be met. This also means that each block must have at least one `?`\-condition.

Blocks of conditions split your rule set into groups. Groups of rules that do not have any conditions are always applied. All other groups are only applied if their conditions are met.

**Examples**

```
# If a rule is placed here, it will be applied
# Same with the one here

?false               # This condition is always false
# The rule placed here will never be applied, because the condition is not met
# Very useful indeed

?exists: //article   # This condition is true if the page has an article tag
# On these lines, a new group of rules is located, and it will be applied if
# there's an article tag on the page, despite the ?false condition above

?exists: //article
?exists: //div[@id="article"]
!exists: //meta[@property="og:title"]
# The rules below this block will be applied if an <article> tag or a
# <div id="article"> can be found, this tag must also be present: <meta property="og:title">
```


> [Check out the Supported Conditions to see what works out of the box »](#supported-conditions)

#### [](#properties)Properties

Properties are the building blocks for your IV page. Check the [Instant View Format](#instant-view-format) for a list of properties that can be used when creating IV pages (you can also define custom properties, but they will not be used anywhere on the resulting IV page). Use this format to fill properties:

```
property: xpath_query
property: "Some string"
property: null
```


Properties store the first node that matches the XPath expression `xpath_query`. By default, if a property already has a value, it will **not** be overwritten. You can change this behavior by adding `!` to the property name – in this case a new non-empty value can be assigned. If you add `!!` to the property name, even an empty new value can be assigned to the property.

You can also assign a `string` to the property instead of the `xpath_query`. In this case, the property will contain a text element with the specified text. It is also possible to assign `null` to discard the property's value (will only work with `!!`).

**Examples**

```
title:   //article//h1      # Looking for the 'title' in the article heading
title:   //h1               # If not found, checking for any headings at all
title:   //head/title       # If no luck either, trying the <title> tag
?path: /blog/.*             # On pages in the /blog/ section 
title!:  //div[@id="title"] # we prefer to use <div id="title">, if present
?path: /news/.*             # On pages in the /news/ section
title!!: //h3               # title is always in an h3 tag

author:   //article/@author  # Get author name from the author attribute
?exists:  //article[has-class("anonymous")]    
author!!: null               # Don't display author for anonymous posts
```


> **Reminder:** The _title_ and _body_ properties are required for an IV page to be created.

#### [](#variables)Variables

Variables are useful for storing and manipulating nodes before assigning them to [properties](#properties) for the final IV page. Variable names begin with the `$` symbol. Use this format to initialize them:

```
$variable: xpath_query
$variable: "Some text"
$variable: null
```


Variables store a list of nodes that match the Xpath expression `xpath_query`. If a variable is assigned for the second time, its previous value is overwritten. You can change this behavior by adding `?` to the variable name. You can also append a list of nodes to the previous one by adding `+` to the variable name.

You can also assign a `string` to the variable instead of the `xpath_query`. In this case, the variable will contain a list with one text element that has the specified text. It is also possible to assign `null` to discard the variable's value.

**Examples**

```
$images:  //img
$images:  //img[@src]     # the previous value will be overwritten
$images?: //article//img  # a new value will only be assigned if the variable is empty
$medias:  //img
$medias+: //video         # now $medias contains all img and video tags
```


#### [](#options)Options

Options affect how the page is processed by the bot. Options begin with the `~` symbol. Use this format to set them:

```
~option: "value"
~option: true
```


Options can be set with values in JSON format.

**Examples**

```
~version: "2.1"
```


> Check out [Supported Options](#supported-options) to see what else works out of the box.

#### [](#functions)Functions

Functions are extremely flexible, but you'll probably mostly use them to strip unnecessary nodes from the page and to replace certain elements with others. Function names begin with the `@` symbol, you can use the following format:

```
@function:                 xpath_query   # a function without parameters
@function(param):          xpath_query   # additional parameters are placed in paretheses
@function(p1 p2):          xpath_query   # a function with two parameters
@function(p1, "param #2"): xpath_query   # parameters can be separated by commas
                                         # and enclosed in quote marks when needed
@function:                 "Some text"   # use string instead of xpath_query if needed
```


The main argument of a function is a list of nodes that match the Xpath expression `xpath_query`. You can also use a `string` as the main argument instead of an `xpath_query`. In this case, the main argument passed to the function will be a list with one text element that has the specified text.

**Examples**

```
@remove:            //header               # removes all <header> tags found on the page
@replace_tag(<h1>): //div[@class="header"] # replaces all <div class="header"> tags with <h1>
<h1>:               //div[@class="header"] # an alias for @replace_tag(<h1>)
```


**Note:** You may find the [@debug function](#debug) particularly useful when creating XPath expressions.

> Check out [Supported Functions](#supported-functions) to see what else works out of the box.

#### [](#block-functions)Block Functions

Block functions manipulate blocks of rules. Block function names also begin with the `@` symbol, you can use the following format:

```
@function(xpath_query) {   # opening bracket should be at the end of the line
  $variable: xpath_query   # some other rules here
  @function: $@            #   inside the block function
}                          # closing bracket should be on a separate line
```


A block function can contain another block function so they can be nested. Block function can not contain [conditions](#conditions).

> Please note that block functions that execute a block of rules several times **(map**, **repeat**, **while**, **while\_not)** have a limit on the total number of iterations for the entire template.

**Examples**

```
@if( //article ) {    # if <article> exists
  @append(<p>): $    #   append paragraph into it
}
```


> Check out [Supported Block Functions](#supported-block-functions) to see what else works out of the box.

#### [](#include)Include

> **Note:** This is a service rule, it **will not work** in your templates. It was included in this reference to give you a better understanding of how the system works. You can see an example of this rule at work in the [Processing Pages](#processing-pages) section.

Rules of this type begin with the `+` symbol and use the following format:

```
+ rules
```


This rule inserts a block of rules with the specified name. In most cases, the name corresponds to the domain to which the rules apply.

**Examples**

```
+ core.telegram.org # inserting the block of rules that is used for core.telegram.org
?not_exists: $body
+ telegram.org      # inserting the block of rules that is used for telegram.org
```


#### [](#special-variables-and)Special variables `$$` and `$@`

The special variables work within a group of rules.

*   The `$$` variable always contains the result of the most recent XPath query.
*   The `$@` variable holds the return of the most recently run function.

These variables come in handy when you're creating chains of rules. If a rule is missing xpath\_query, the statement is considered to be equal to `$$`.

**Examples**

```
# Put a picture into a <figure> tag, then set it as the cover
@wrap(<figure>): //img[@id="cover"]
cover:           $@

# Insert a divider before each div.divider that's no longer required
@before(<hr>):   //div[has-class("divider")]
@remove          # this is the same as @remove: $
```


### [](#extended-xpath)Extended XPath

For your convenience, you can use the following constructs in your XPath expressions.

#### [](#context)Context

Regular XPath queries search for nodes in the context of the entire document. To specify the context for a particular expression, you can use variables in the format `$context`. If a matching variable exists, the query will be made relative to each variable node. If it doesn't and a matching property exists, the query will be made relative to the property node. If no matching variables or properties exist, the query return an empty list.

**Examples**

```
$headers:      //h1              # all <h1> tags on the page
article:       //article         # the first <article> tag on the page
$art_headers:  $article//h1      # all <h1> tags inside article
$header_links: $art_headers//a   # all <a> tags inside each $art_headers node
```


#### [](#zeroing-in-on-nodes)Zeroing in on nodes

The result of an XPath query is always a list of matching nodes. If you'd like to narrow the list down to a single node, you can use the following syntax: `(xpath_query)
[n]`, where `n` is the number of the desired node. Numbering starts at 1, you can get the last node by using the `last()` function. This syntax can only be applied to the entire expression.

**Examples**

```
$headers:    //h1                    # all <h1> tags on the page
$header2:    (//h1)
[2]               # the second <h1> tag on the page
$header2:    ($headers)
[2]           # same
$last_link:  ($header2//a)
[last()]   # the last link inside $header2
```


#### [](#has-class)has-class

You will often need to locate nodes that have a certain class. To do this, you can use the `has-class("class")` function that serves as an alias for the expression `contains(concat(" ", normalize-space(@class), " "), " class ")`.

**Examples**

```
# Transform all div.header elements into h1
<h1>: //div[contains(concat(" ", normalize-space(@class), " "), " header ")]
# same idea, but much shorter
<h1>: //div[has-class("header")]
```


#### [](#ends-with)ends-with

XPath has a `starts-with` function that checks whether the first string starts with the second string but does not have `ends-with`. In IV you can use the `ends-with("haystack","needle")` function that serves as an alias for the expression `(substring("haystack", string-length("haystack") - string-length("needle") + 1) = "needle")`.

**Examples**

```
# Select all JPG images
@debug: //img[(substring(@src, string-length(@src) - string-length(".jpg") + 1) = ".jpg")]
# the same, but much shorter
@debug: //img[ends-with(@src, ".jpg")]
```


#### [](#prev-sibling)prev-sibling

An axis that selects the previous sibling of the current node. Serves as an alias for the expression `preceding-sibling::*[1]/self`.

**Examples**

```
# Transform all div nodes, that immediately follow img nodes into figcaption
<figcaption>: //div[./preceding-sibling::*[1]/self::img]
# the same
<figcaption>: //div[./prev-sibling::img]
```


#### [](#next-sibling)next-sibling

An axis that selects the next sibling of the current node. Serves as an alias for the expression `following-sibling::*[1]/self`

**Examples**

```
# Join paragraphs following each other into one, separating them with a line break
@combine(<br>): //p/following-sibling::*[1]/self::p
# the same, but shorter
@combine(<br>): //p/next-sibling::p
```


### [](#supported-conditions)Supported conditions

Below are conditions supported in Instant View rules.

#### [](#domain)domain

```
domain: regexp
```


Checks whether the domain of the current page matches a regular expression. The check is case-insensitive, so the actual expression used will look like this: `/^regexp$/i`.

**Examples**

```
# Rule for *.subdomain.example.com URLs
?domain: .+\.subdomain\.example\.com
```


#### [](#domain-not)domain\_not

```
domain_not: regexp
```


Checks whether the domain of the current page does not match a regular expression. The check is case-insensitive, so the actual expression used will look like this: `/^regexp$/i`.

**Examples**

```
# Rule for *.subdomain.example.com URLs with an exception for dev.subdomain.example.com
?domain:     .+\.subdomain\.example\.com
!domain_not: dev\.subdomain\.example\.com
```


#### [](#path)path

```
path: regexp
```


Checks whether the path to the current page matches a regular expression. The check is case-insensitive, so the actual expression used will look like this: `/^regexp$/i`.

**Examples**

```
# Rule for example.com/news/* links
?path: /news/.+   # no need to escape the slash
```


#### [](#path-not)path\_not

```
path_not: regexp
```


Checks whether the path to the current page does not match a regular expression. The check is case-insensitive, so the actual expression used will look like this: `/^regexp$/i`.

**Examples**

```
# Rule for example.com/news/* links with an exception for example.com/news/recent
?path:     /news/.+
!path_not: /news/recent
```


#### [](#exists)exists

```
exists: xpath_query
```


Checks whether target nodes exist on the current page.

**Examples**

```
# Apply rule only to pages with an <article> tag
?exists: //article
```


#### [](#not-exists)not\_exists

```
not_exists: xpath_query
```


Checks whether target nodes do not exist on the current page.

**Examples**

```
# Apply rule only to pages with an <article> tag, that do not include any quotes.
?exists:     //article
!not_exists: //article//blockquote
```


#### [](#true)true

```
true
```


A condition that is always true.

**Examples**

```
?path:    /blog/.+
# Rules for the blog section go here
?true
# Rules that go here will be applied to all pages
```


#### [](#false)false

```
false
```


A condition that is always false.

**Examples**

```
?path:    /news/.+
# Rules for the news section go here
?false
# Rules that go here will never be applied
```


### [](#supported-options)Supported options

Below are options supported in Instant View rules.

#### [](#version)version

```
~version: "2.1"
```


Sets the version of IV used in the template. The behavior of some IV functions may change according to the version provided (see the manual of corresponding function).  
The value must be one of `"1.0"`, `"2.0"` or `"2.1"`. We recommend using the latest version, **2.1**.

**Examples**

```
~version: "2.1"
# Now you can use new features from IV 2.1
```


> Please note that `version` should be set at the beginning of the template before any other rules.

#### [](#allowed-origin-new)allowed\_origin **NEW**

```
~allowed_origin: "https://example.com"
~allowed_origin: ["https://example.com", "https://subdomain.example.com"]
```


Sets the origin (or the list of origins) from which content can be loaded using the [@load](#load-new) and [@inline](#inline) functions.  
The value should be _String_ or _Array of String_.

**Examples**

```
~allowed_origin: "https://api.example.com"
# Now you can load content from https://api.example.com regardless of the origin of current page
@load: "https://api.example.com/get/article"
```


### [](#supported-functions)Supported functions

The general format for a function is the following:

```
@function: xpath_query
```


Each function accepts the list of nodes returned by the XPath statement as the main parameter. You may omit `xpath_query`, in which case `$$`, the result of the previous XPath query, will be passed into the function. The function then processes each element in the list and returns a list of transformed nodes which are stored in the `$@` variable.

Below is a list of functions that are supported in Instant View rules.

#### [](#debug)debug

```
@debug: xpath_query
```


Logs the elements passed into the function, these elements will be displayed in the status line at the bottom of the screen in the [Instant View Editor](https://instantview.telegram.org/#instant-view-editor). Returns the elements passed. Consider combining with [`$$` and `$@`](#special-variables-and).

**Example**

```
@debug: //article//a     # displays all links from the page in the log
@debug                   # displays all links again
```


Look for debug output at the bottom of your Editor:

[![](https://instantview.telegram.org/file/811140378/19ec/lgDhyQ_HKcc.120673/c7602c526f316dc5a4 "Debug console at the bottom of the screen")](/file/811140378/19ec/lgDhyQ_HKcc.120673/c7602c526f316dc5a4)

#### [](#remove)remove

```
@remove: xpath_query
```


Removes target nodes from the page, returns an empty list.

**Examples**

```
@remove: //div[has-class("related")] # remove article div.related
@debug                               # empty list
```


#### [](#append)append

```
@append("Some text"): xpath_query
@append(@attr):       xpath_query
@append(<tag>):       xpath_query
@append(<tag>, attr, value[, attr, value[, ...]]): xpath_query
```


Inserts content, specified by the parameter, to the end of each target node.

*   If the first parameter is in angle brackets, a new `<tag>` node will be created. The following two parameters then specify the name and value of an attribute for that node. If `value` has the format `@attr`, the value of the attribute `attr` in the target node will be used as the value. `value` can be an XPath expression that begins with `.`, in this case the query will be executed relative to the relevant node, and the text value of the returned elements will be used as the attribute's value. Otherwise, `value` will be used as the attribute's value.
*   If the first parameter has the format `@attr`, a new text element with value of the attribute `attr` of the targeted node will be created.
*   Otherwise, a text element with the text specified in the first parameter will be created.

Returns a list of the newly created nodes.

**Examples**

```
# <div class="a"><em>1</em></div>
@append(<p>): //div
# <div class="a"><em>1</em><p></p></div>
```


#### [](#prepend)prepend

```
@prepend("Some text"): xpath_query
@prepend(@attr):       xpath_query
@prepend(<tag>):       xpath_query
@prepend(<tag>, attr, value[, attr, value[, ...]]): xpath_query
```


Inserts content, specified by the parameter, to the beginning of each target node.

*   If the first parameter is in angle brackets, a new `<tag>` node will be created. The following two parameters then specify the name and value of an attribute for that node. If `value` has the format `@attr`, the value of the attribute `attr` in the target node will be used as the value. `value` can be an XPath expression that begins with `.`, in this case the query will be executed relative to the relevant node, and the text value of the returned elements will be used as the attribute's value. Otherwise, `value` will be used as the attribute's value.
*   If the first parameter has the format `@attr`, a new text element with value of the attribute `attr` of the targeted node will be created.
*   Otherwise, a text element with the text specified in the first parameter will be created.

Returns a list of the newly created nodes.

**Examples**

```
# <div class="a"><em>1</em></div>
@prepend(<p>, data-class, @class): //div
# <div class="a"><p data-class="a"></p><em>1</em></div>
```


#### [](#after)after

```
@after("Some text"): xpath_query
@after(@attr):       xpath_query
@after(<tag>):       xpath_query
@after(<tag>, attr, value[, attr, value[, ...]]): xpath_query
```


Inserts content, specified by the parameter, after each target node.

*   If the first parameter is in angle brackets, a new `<tag>` node will be created. The following two parameters then specify the name and value of an attribute for that node. If `value` has the format `@attr`, the value of the attribute `attr` in the target node will be used as the value. `value` can be an XPath expression that begins with `.`, in this case the query will be executed relative to the relevant node, and the text value of the returned elements will be used as the attribute's value. Otherwise, `value` will be used as the attribute's value.
*   If the first parameter has the format `@attr`, a new text element with value of the attribute `attr` of the targeted node will be created.
*   Otherwise, a text element with the text specified in the first parameter will be created.

Returns a list of the newly created nodes.

**Examples**

```
# <div class="a"><em>1</em><em>2</em></div>
@after("!"): //div/em
# <div class="a"><em>1</em>!<em>2</em>!</div>
```


#### [](#before)before

```
@before("Some text"): xpath_query
@before(@attr):       xpath_query
@before(<tag>):       xpath_query
@before(<tag>, attr, value[, attr, value[, ...]]): xpath_query
```


Inserts content, specified by the parameter, before each target node.

*   If the first parameter is in angle brackets, a new `<tag>` node will be created. The following two parameters then specify the name and value of an attribute for that node. If `value` has the format `@attr`, the value of the attribute `attr` in the target node will be used as the value. `value` can be an XPath expression that begins with `.`, in this case the query will be executed relative to the relevant node, and the text value of the returned elements will be used as the attribute's value. Otherwise, `value` will be used as the attribute's value.
*   If the first parameter has the format `@attr`, a new text element with value of the attribute `attr` of the targeted node will be created.
*   Otherwise, a text element with the text specified in the first parameter will be created.

Returns a list of the newly created nodes.

**Examples**

```
# <div class="a"><em>1</em></div>
@before("@"): //div/em
# <div class="a">@<em>1</em></div>
```


#### [](#append-to)append\_to

```
@append_to(xpath_query): xpath_query
@append_to($var):        xpath_query
```


Inserts each target node to the end of the base node.

The first parameter specifies the base node. You can pass an XPath expression that begins with `.`, in this case the query will be executed relative to the relevant node. The _first relevant node_ will be used as the base node. You can also pass a variable name. If such a variable exists, the _first relevant node_ will be used as the base node. If the variable doesn't exist or is empty, the property with a matching name will be used as the base node.

Returns a list of target nodes.

**Examples**

```
$div:             //div
# <div class="a"><em></em></div><p>Text</p>
@append_to($div): //p
# <div class="a"><em></em><p>Text</p></div>
```


#### [](#prepend-to)prepend\_to

```
@prepend_to(xpath_query): xpath_query
@prepend_to($var):        xpath_query
```


Inserts each target node to the beginning of the base node.

The first parameter specifies the base node. You can pass an XPath expression that begins with `.`, in this case the query will be executed relative to the relevant node. The _first relevant node_ will be used as the base node. You can also pass a variable name. If such a variable exists, the _first relevant node_ will be used as the base node. If the variable doesn't exist or is empty, the property with a matching name will be used as the base node.

Returns a list of target nodes.

**Examples**

```
$div:              //div
# <div class="a"><em></em></div><p>Text</p>
@prepend_to($div): //p
# <div class="a"><p>Text</p><em></em></div>
```


#### [](#after-el)after\_el

```
@after_el(xpath_query): xpath_query
@after_el($var):        xpath_query
```


Inserts each target node after the base node.

The first parameter specifies the base node. You can pass an XPath expression that begins with `.`, in this case the query will be executed relative to the relevant node. The _first relevant node_ will be used as the base node. You can also pass a variable name. If such a variable exists, the _first relevant node_ will be used as the base node. If the variable doesn't exist or is empty, the property with a matching name will be used as the base node.

Returns a list of target nodes.

**Examples**

```
$div:                        //div
# <div class="a"><p>Text</p><em></em></div>
@after_el("./../self::div"): //p
# <div class="a"><em></em></div><p>Text</p>
```


#### [](#before-el)before\_el

```
@before_el(xpath_query): xpath_query
@before_el($var):        xpath_query
```


Inserts each target node before the base node.

The first parameter specifies the base node. You can pass an XPath expression that begins with `.`, in this case the query will be executed relative to the relevant node. The _first relevant node_ will be used as the base node. You can also pass a variable name. If such a variable exists, the _first relevant node_ will be used as the base node. If the variable doesn't exist or is empty, the property with a matching name will be used as the base node.

Returns a list of target nodes.

**Examples**

```
$div:                        //div
# <div class="a"><p>Text</p><em></em></div>
@before_el("./../self::div"): //p
# <p>Text</p><div class="a"><em></em></div>
```


#### [](#replace-tag)replace\_tag

```
@replace_tag(<tag>):     xpath_query
<tag>:                   xpath_query
```


Changes the name of the tag. The new name for the tag should be passed as the first parameter. Returns target nodes.

**Examples**

```
# <div class="list unordered"><div class="item"></div><div class="item"></div></div>
@replace_tag(<li>): //div[has-class("item")]
<ul>:               //div[has-class("list")]
# <ul class="list unordered"><li class="item"></li><li class="item"></li></ul>
```


#### [](#wrap)wrap

```
@wrap(<tag>): xpath_query
```


Wrap each target node in the `<tag>` tag. Returns a list of the new `<tag>` elements.

**Examples**

```
# <em>1</em><em>2</em>
@wrap(<b>): //em
# <b><em>1</em></b><b><em>2</em></b>
@wrap(<u>)
# <b><u><em>1</em></u></b><b><u><em>2</em></u></b>
@wrap(<p>): $@
# <b><p><u><em>1</em></u></p></b><b><p><u><em>2</em></u></p></b>
```


#### [](#wrap-inner)wrap\_inner

```
@wrap_inner(<tag>): xpath_query
```


Wrap an HTML structure around the content of each target node in the `<tag>` tag. Returns a list of new `<tag>` elements.

**Examples**

```
# <p>H<sub>2</sub>0</p>
@wrap_inner(<b>): //p
# <p><b>H<sub>2</sub>0</b></p>
```


#### [](#clone)clone

```
@clone: xpath_query
```


Creates a copy of each target node. Returns a list of the newly created nodes.

**Examples**

```
# <p class="text">Paragraph</p>
@clone: //p
# <p class="text">Paragraph</p><p class="text">Paragraph</p>
```


#### [](#detach)detach

```
@detach: xpath_query
```


Separates the target node from the rest in the parent tag. Creates a copy of the parent tag and wraps the target node into this new instance. Returns a list of parent tags that contain target nodes.

**Examples**

```
# <a href="#1">
#   <b>1</b>
#   <p>Link #1</p>
# </a>
# <a href="#2">
#   <b>2</b>
#   <p>Link #2</p>
# </a>

@detach: //a/b

# <a href="#1">
#   <b>1</b>
# </a>
# <a href="#1">
#   <p>Link #1</p>
# </a>
# <a href="#2">
#   <b>2</b>
# </a>
# <a href="#2">
#   <p>Link #2</p>
# </a>

@after(<br>): $@

# <a href="#1">
#   <b>1</b>
# </a><br>
# <a href="#1">
#   <p>Link #1</p>
# </a>
# <a href="#2">
#   <b>2</b>
# </a><br>
# <a href="#2">
#   <p>Link #2</p>
# </a>
```


#### [](#split-parent)split\_parent

```
@split_parent: xpath_query
```


Splits the parent tag by the target node. Places preceding siblings wrapped into parent element before the target node. Places following siblings wrapped into parent element placed after target node.

Returns a list of target nodes.

**Examples**

```
# <p class="text">
#   <b>First</b> line
#   <figure><img src="photo.jpg"></figure>
#   <i>Second</i> line
# </p>
@split_parent: //p/figure
# <p class="text">
#   <b>First</b> line
# </p>
# <figure><img src="photo.jpg"></figure>
# <p class="text">
#   <i>Second</i> line
# </p>
```


#### [](#pre)pre

```
@pre: xpath_query
```


Specifies that the text inside the target node is already formatted. Returns a list of matching nodes.

**Examples**

```
# <p>     Some          text  , </p>
# <p>  Some  another      text  </p>
@pre: (//p)
[1]
# Result in the Instant View page:
#      Some          text  , 
# Some another text
```


#### [](#set-attr)set\_attr

```
@set_attr(attr, value):       xpath_query
@set_attr(attr, @attr):       xpath_query
@set_attr(attr, "Some text"): xpath_query
@set_attr(attr, "Some text", @from-attr, ...): xpath_query
```


Sets an attribute in each matching node. The name of the attribute is passed in the first parameter. The value of the attribute is the rest of the parameters concatenated.

If `value` has the format `@attr`, the value of the attribute `attr` of the target node will be used as the value. `value` can be an XPath expression that begins with `.`, in this case the query will be executed relative to the relevant node, and the text value of the returned elements will be used as the attribute's value. Otherwise, `value` will be used as the attribute's value.

Returns a list of attribute nodes.

**Examples**

```
# <p class="a"></p>
@set_attr(data-class, @class)
# <p class="a" data-class="a"></p>
@set_attr(id, @class, "_", ./@data-class)
# <p class="a" data-class="a" id="a_a"></p>
```


#### [](#set-attrs)set\_attrs

```
@set_attrs(attr, value): xpath_query
@set_attrs(attr, value[, attr, value[, ...]]): xpath_query
```


Sets multiple attributes for each of the target nodes. Each two parameters specify the name and value for the new attributes.

If `value` has the format `@attr`, the value of the attribute `attr` of the target node will be used as the value. `value` can be an XPath expression that begins with `.`, in this case the query will be executed relative to the relevant node, and the text value of the returned elements will be used as the attribute's value. Otherwise, `value` will be used as the attribute's value.

Returns a list of the matching nodes.

**Examples**

```
# <p class="a"></p>
@set_attrs(data-class, @class, id, "a_a"): //p
# <p class="a" data-class="a" id="a_a"></p>
```


#### [](#match)match

```
@match(regexp):                         xpath_query
@match(regexp, match_index):            xpath_query
@match(regexp, match_index, modifiers): xpath_query
```


Performs a search based on a regular expression. Replaces the content of the target node with the search result. The second parameter specifies the number of the captured parenthesized subpattern. If this is not specified, the text that matched the full pattern will be used. You can specify modifiers for the regular expression using the optional parameter ([ims modifiers](http://pcre.org/pcre.txt) are supported).

Returns a list of target nodes. As of IV **2.1**, returns a list of target nodes the content of which matched the regular expression.

**Examples**

```
# <p class="plainText">Hello, world!</p>
@match("[a-z]+!"): //p
# <p class="plainText">world!</p>
@match("([a-z]+)!", 1): //p/text()
# <p class="plainText">world</p>
@match("..t", 0, "i"): //p/@class
# <p class="inT">Bye, world!</p>

# <ul><li>a1</li><li>a</li><li>21b</li><li>.</li></ul>
@match("[0-9]+"): //ul/li
# <ul><li>1</li><li></li><li>21</li><li></li></ul>
@debug: $@
# Debug 2 nodes:
#   [0]: <li>1</li>
#   [1]: <li>21</li>
```


#### [](#replace)replace

```
@replace(regexp, replacement):            xpath_query
@replace(regexp, replacement, modifiers): xpath_query
```


Performs a search and replace operation using the regular expression. You can specify modifiers for the regular expression using the optional parameter ([ims modifiers](http://pcre.org/pcre.txt) are supported).

Returns a list of target nodes. As of IV **2.1**, returns a list of target nodes the content of which was replaced by the regular expression.

**Examples**

```
# <p class="text">Hello, world!</p>
@replace("Hello", "Goodbye"): //p/text()
# <p class="text">Goodbye, world!</p>
@replace(".t$", "mp"): //p/@class
# <p class="temp">Goodbye, world!</p>
@replace("goodb", "B", "i"): //p/text()
# <p class="temp">Bye, world!</p>

# <ul><li>a1</li><li>a</li><li>21b</li><li>.</li></ul>
@replace("[0-9]+", "($0)"): //ul/li
# <ul><li>a(1)</li><li>a</li><li>(21)b</li><li>.</li></ul>
@debug: $@
# Debug 2 nodes:
#   [0]: <li>a(1)</li>
#   [1]: <li>(21)b</li>
```


#### [](#urlencode)urlencode

```
@urlencode: xpath_query
```


Encodes the content of the target node as per RFC 3986. Returns target nodes.

**Examples**

```
# <a href="https://telegram.org">link</a>
$a: //a
@urlencode: $a/@href
# <a href="https%3A%2F%2Ftelegram.org">link</a>
@set_attr(href, "/?url=", @href): $a
# <a href="/?url=https%3A%2F%2Ftelegram.org">link</a>
```


#### [](#urldecode)urldecode

```
@urldecode: xpath_query
```


Decodes the content of the target node as per RFC 3986. Returns target nodes.

**Examples**

```
# <a href="/?url=https%3A%2F%2Ftelegram.org">link</a>
$a: //a
@match("^/\?url=(.+)$", 1): $a/@href
# <a href="https%3A%2F%2Ftelegram.org">link</a>
@urldecode
# <a href="https://telegram.org">link</a>
```


#### [](#htmlencode)htmlencode

```
@htmlencode: xpath_query
```


Convert special characters in the target node to HTML entities. Returns the target nodes.

**Examples**

```
# <p>&lt;b&gt;Some text&lt;\b&gt;</p>
@htmlencode: //p
# <p>&amp;lt;b&amp;gt;Some text&amp;lt;\b&amp;gt;</p>
```


#### [](#htmldecode)htmldecode

```
@htmldecode: xpath_query
```


Convert special HTML entities in the target node back to characters. Returns the target nodes.

**Examples**

```
# <p>&amp;lt;b&amp;gt;Some text&amp;lt;\b&amp;gt;</p>
@htmldecode: //p
# <p>&lt;b&gt;Some text&lt;\b&gt;</p>
```


#### [](#style-to-attrs)style\_to\_attrs

```
@style_to_attrs(css_prop, attr): xpath_query
@style_to_attrs(css_prop, attr[, css_prop, attr[, ...]]): xpath_query
```


Gets a value of a CSS property from the style attribute and sets it into an attribute for each of the target nodes. Each two parameters specify the name for the new attribute and the name of CSS property. Returns a list of matching nodes.

**Examples**

```
# <img style="width: 20px; height: 20px">
@style_to_attrs(width, data-width, height, data-height): //img
# <img style="width: 20px; height: 20px" data-width="20" data-height="20">
```


#### [](#background-to-image)background\_to\_image

```
@background_to_image: xpath_query
```


Transforms the target node with a background picture into an `<img>` tag with an `src` attribute. The target node must contain a `style` attribute with the background. Returns a list of transformed nodes.

**Examples**

```
# <div class="bg_image" style="background-image:url(image.jpg)"></div>
@background_to_image: //div[has-class("bg_image")]
# <img src="image.jpg">
```


#### [](#json-to-xml)json\_to\_xml

```
@json_to_xml: xpath_query
```


Transforms the content of the target node to the XML format. The contents must be in valid JSON format. The resulting XML tree will be inserted into the document. Returns the root element of the XML tree.

**Examples**

```
# <div data-var='{"a":1,"b":[1,2],"c":{"p":"Hello!"}}'></div>
@json_to_xml: //div/@data-var
@debug: $@
# <xml><a>1</a><b><item>1</item><item>2</item></b><c><p>Hello!</p></c></xml>
```


#### [](#html-to-dom)html\_to\_dom

```
@html_to_dom: xpath_query
```


Parse the content of the target node in HTML format and insert it into the document. Returns the root element of the inserted HTML tree.

**Examples**

```
# <div data-content="&lt;b&gt;Some text&lt;\b&gt;"></div>
@html_to_dom: //div/@data-content
@debug: $@
# <dom><b>Some text</b></dom>
```


#### [](#combine)combine

```
@combine:                        xpath_query
@combine(" - "):                 xpath_query
@combine(<tag>):                 xpath_query
@combine(<tag>[, " - "[, ...]]): xpath_query
```


Combines each target node with its preceding node if such a node exists. You can use parameters to specify nodes to be inserted above the target node. If the parameter is in angular brackets, a new `<tag>` node will be created. Otherwise a text element with the text specified by the parameter will be used.

Returns a list of nodes, preceding the target nodes.

**Examples**

```
# <pre>1 2 3</pre>
# <pre> 4 5 </pre>
# <pre>6 7 8</pre>
@combine:             //pre/following-sibling::*[1]/self::pre
# <pre>1 2 3 4 5 6 7 8</pre>

# <p>first</p>
# <p>second</p>
@combine(<br>, <br>): //p/following-sibling::*[1]/self::p
# <p>first<br><br>second</p>

# <h1>Title</h1>
# <strong>Subtitle</strong>
@combine(<br>, "\n"): //h1/following-sibling::*[1]/self::strong
# <h1>Title<br>
# Subtitle</h1>
```


#### [](#datetime)datetime

```
@datetime(-2): xpath_query
@datetime(-2, "en-US", "yyyy-MM-dd"): xpath_query
```


Transforms the date and time from the text of the target node into a unixtime timestamp. The parameter specifies the timezone of the original date and time.

You can also pass a locale and a pattern to parse the date and time in more complicated cases. If a non-gregorian calendar is used, this needs to be specified in the locale. For example, `"fa-IR@calendar=persian"`. Available patterns are documented [here](http://userguide.icu-project.org/formatparse/datetime#TOC-Date-Time-Format-Syntax).

On success, returns a text element with the unixtime timestamp. Otherwise, returns the target element.

**Examples**

```
# <div id="date">1 January 2017, 15:04</div>
@datetime(-2):  //div[@id="date"]
published_date: $@   # the property published_date will hold a unixtime timestamp

# <time>دوشنبه, ۲۳ اسفند ۱۳۹۵</time>
@datetime(0, "fa-IR@calendar=persian", "EEEE, dd MMMM yyyy"): //time
published_date: $@   # the property published_date will hold a unixtime timestamp
```


#### [](#simplify)simplify

> **Note:** This is a service function. There is no reason for you to use it in your templates. It was included in this reference to give you a better understanding of how the system works (see the `..after` block).

```
@simplify: xpath_query
```


Processes the target nodes according to the Instant View format. Returns the target nodes.

**Examples**

```
# <p><span><b>S</b>ome</span> <em>important</em> text!</p>
@simplify: //p
# <p><b>S</b>ome <em>important</em> text!</p>
```


#### [](#inline)inline

```
@inline:         xpath_query
@inline(silent): xpath_query
```


If the target element is an `<iframe>` with the attribute `src`, the content of the iframe will be loaded. The target element will be replaced with the content of the iframe.

If the target node is an HTML-comment, a `<comment>` element with the content of the comment will be created. The comment will be replaced by the newly created node.

You can use `silent` parameter to ignore errors while processing this function. You will still see an error in the debug console, but the rules that come after will continue to execute.

Returns a list of replaced nodes.

> Note: The @inline function adheres to the [same-origin policy](https://en.wikipedia.org/wiki/Same-origin_policy). This means that the target and the inlined pages should have the same origin. You can specify additional allowed origins using the option [~allowed\_origin](#allowed-origin-new)

**Examples**

```
# <p><iframe src="/subpage.html"></iframe></p>
# /subpage.html:
#   <html><body><p>Hello!</p></body></html>
@inline: //p/iframe
# <p><html><body><p>Hello!</p></body></html></p>
@debug
# <html><body><p>Hello!</p></body></html>

# <p><!--<b>Hello!</b>, world!--></p>
@inline: //p/comment()
# <p><comment><b>Hello!</b>, world!</comment></p>
@debug
# <comment><b>Hello!</b>, world!</comment>
```


#### [](#load-new)load **NEW**

```
@load:         "https://example.com"
@load(silent): xpath_query
```


Loads the content of the URL. The URL can be specified as a string or be the content of the target node.

You can use `silent` parameter to ignore errors while processing this function. You will still see an error in the debug console, but the rules that come after will continue to execute.

Returns the newly created node with loaded content.

> Note: The @load function adheres to the [same-origin policy](https://en.wikipedia.org/wiki/Same-origin_policy). This means that the target and the loaded pages should have the same origin. You can specify additional allowed origins using the option [~allowed\_origin](#allowed-origin-new)

**Examples**

```
@append(<iframe> src "/subpage.html"): /html/body
$iframe: $@
@remove
@inline: $iframe
# the same, but much shorter
@load: "/subpage.html"

@load(silent): //meta[@property="api:url"]/@content
# trying to load content from api:url
@if ( $@ ) {
  # Do something with content
}
```


#### [](#unsupported)unsupported

```
@unsupported: xpath_query
```


Specifies that the target node contains essential content that can't be represented in the Instant View format. Returns the target nodes.

**Examples**

```
# <p>See the graph below:</p>
# <canvas class="article_graph" width="600" height="400"></canvas>
@unsupported: //canvas[has-class("article_graph")]
```


> **Tip:** It is important to identify that a page contains essential unsupported content — no IV page will be generated to ensure that the user never gets incomplete info from an article. The system will take care of the most popular cases in the `..after` block, but your template must cover everything that the system doesn't catch.

### [](#supported-block-functions)Supported block functions

The general format for a block function is the following:

```
@function(xpath_query) {
  # block of rules
}
```


Block functions usually accept the list of nodes returned by the XPath statement as a parameter. Depending on the function and its parameters, rules inside the block can be run several times or never. Block functions can contain any type of rules except conditions.

Below is a list of block functions that are supported in Instant View rules.

#### [](#if)if

```
@if(xpath_query) {
  # block of rules
}
```


Executes the block of rules if target nodes exist on the current page. If target nodes do not exist, the block of rules will be skipped. The `$$` and `$@` variables contain target nodes found by the XPath query.

**Example**

```
@if( //div[has-class("blockquote")] ) { # if div.blockquote exists
  <blockquote>: $@                      #   change tag name to blockquote
  <cite>: $@//div[has-class("author")]  #   and mark div.author as a caption
}
```


#### [](#if-not)if\_not

```
@if_not(xpath_query) {
  # block of rules
}
```


Executes a block of rules if target nodes do not exist on the current page. If such nodes exist, the block of rules will be skipped. The `$$` and `$@` variables contain target nodes found by the XPath query.

**Example**

```
@if_not( $body//footer ) {     # if footer does not exist
  @append(<footer>): $body     #   append it into body
}
```


#### [](#map)map

```
@map(xpath_query) {
  # block of rules
}
```


Executes a block of rules once per each target node found by the XPath query. At the beginning of each iteration, the following variables will be set:

*   `$$` will contain the target nodes found by XPath query;
*   `$@` will contain the current target node;
*   `$index` will contain the index of the current target node (starts with 1);
*   `$first` will contain `<true>` if the current target node is the first in the list and an empty list otherwise;
*   `$middle` will contain `<true>` if the current target node is between the first and the last in the list, empty list otherwise;
*   `$last` would contain `<true>` if the current target node is the last one in the list, empty list otherwise.

**Example**

```
@map( //div[@id="list"]/p ) {   # for each paragraph in list
  $p: $@                        #   store the current paragraph in $p variable
  @prepend_to($p): ". "         #   and
  @prepend_to($p): $index       #   number them starting with 1
}
```


> Please note that the `@map` function should be used **only** if it is impossible to use **regular functions** with xpath queries (for example you need to use $index). Otherwise, regular functions are faster because they process several nodes at once.

**Example**

```
# Bad way:
@map( //div[has-class("image")] ) {
  $image: $@
  <cite>:       $image//p/span
  <figcaption>: $image//p
  <figure>:     $image
}
# The same result, but faster:
$image:       //div[has-class("image")]
<cite>:       $image//p/span
<figcaption>: $image//p
<figure>:     $image
```


#### [](#repeat)repeat

```
@repeat(n) {
  # block of rules
}
```


Executes a block of rules **n** times. At the beginning of each iteration, the following variables will be set:

*   `$$` and `$@` will contain the previous value of `$$`;
*   `$index` will contain the index of the current iteration (starts with 1);
*   `$first` will contain `<true>` if this is the first iteration, an empty list otherwise;
*   `$middle` will contain `<true>` if the current iteration is between the first and last, empty list otherwise;
*   `$last` will contain `<true>` if this is the last iteration, empty list otherwise.

**Example**

```
# appends "1, 2, 3, 4, 5" to $body
@repeat( 5 ) {
  @append_to($body): $index
  @if_not( $last ) {
    @append_to($body): ", "
  }
}
```


#### [](#while)while

```
@while(xpath_query) {
  # block of rules
}
```


Executes a block of rules while target nodes exist on the current page. At the beginning of each iteration the following variables will be set:

*   `$$` and `$@` will contain target nodes found by XPath query;
*   `$index` will contain the index of the current iteration (starts with 1);
*   `$first` will contain `<true>` if it is the first iteration, an empty list otherwise.

**Example**

```
@while( //iframe ) {
  @inline: $@          # inline all nested iframes
}
```


#### [](#while-not)while\_not

```
@while_not(xpath_query) {
  # block of rules
}
```


Executes a block of rules while target nodes do not exist on the current page. At the beginning of each iteration the following variables will be set:

*   `$$` and `$@` will contain target nodes found by XPath query;
*   `$index` will contain the index of the current iteration (starts with 1);
*   `$first` will contain `<true>` if this is the first iteration, an empty list otherwise.

**Example**

```
@while_not( //video ) {   # if no video exists on the current page
  @inline: //iframe       #   try to inline iframes, maybe video is inside the frame
}
```


### [](#embedded-elements)Embedded elements

Instant View supports widgets from popular services. We display them as embedded elements or specially formatted quotes. The Instant View bot analyzes all iframes in the document body and generates a widget if the service is supported.

> Some popular widgets do not use iframes. Note that the `..after` block of rules contains rules to transform such widgets to an Instant View compatible format.

We currently support the following services:

*   Youtube
*   Vimeo
*   Tweets & Twitter Videos
*   Facebook Posts & Videos
*   Instagram
*   Giphy
*   SoundCloud
*   GithubGist
*   Aparat
*   VK.com Videos

### [](#processing-pages)Processing pages

All pages are processed according to the following rules:

```
# Url: http://example.com/some_page.html
+ example.com
?true
+ ..after
```


If a page is on a third-level domain and above, the following rules are applied:

```
# Url: http://some.subdomain.example.com/some_page.html
+ some.subdomain.example.com
?not_exists: $body
+ subdomain.example.com
?not_exists: $body
+ example.com
?true
+ ..after
```


In other words, at first the bot attempts to use a block of rules that features the full domain name. If such a block does not exist, the bot checks for the next level up to the second-level domain.

`..after` is a special block of rules that is applied to all pages irrespective of the domain name.

#### [](#working-with-subdomains)Working with subdomains

If the page is on a third-level domain or higher, you need to manually select the domain level to which your template will apply. Use this menu in the top left corner of the page:

[![](https://instantview.telegram.org/file/811140424/1/iHfwojr4Vgs.13156/e22aeb9d421eb01c85 "Select domain level")](/file/811140424/1/iHfwojr4Vgs.13156/e22aeb9d421eb01c85)  

Choose a level that hosts pages with the same structure, so that your rules are relevant to all the matching pages on this domain.

> For example, use `wikipedia.org` to create an IV page for `https://en.wikipedia.org/wiki/Domain_name` because the page structure doesn't depend on language in Wikipedia.