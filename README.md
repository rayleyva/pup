# pup

pup is a command line tool for processing HTML. It reads from stdin,
prints to stdout, and allows the user to filter parts of the page using
[CSS selectors](https://developer.mozilla.org/en-US/docs/Web/Guide/CSS/Getting_started/Selectors).

Inspired by [jq](http://stedolan.github.io/jq/), pup aims to be a
fast and flexible way of exploring HTML from the terminal.

## Upcoming talk

I'll be giving a talk on pup on October 15th at 11:00am (PDT) for Codementor!
RSVP on the [event page](https://www.codementor.io/officehours/3636154029/stdin-stdout-pup-go-and-life-at-the-command-line)
and join the hangout to discuss, pup, Go, and command-line tools.

## Install

Direct downloads are available through the [releases page](https://github.com/EricChiang/pup/releases/latest).

If you have Go installed on your computer just run `go get`.

    go get github.com/ericchiang/pup

If you're on OS X, use [Brew](http://brew.sh/) to install (no Go required).

    brew install https://raw.githubusercontent.com/EricChiang/pup/master/pup.rb

## Quick start

```bash
$ curl -s https://news.ycombinator.com/
```

Ew, HTML. Let's run that through some pup selectors:

```bash
$ curl -s https://news.ycombinator.com/ | pup 'td.title a[href^=http] attr{href}'
```

Even better, let's grab the titles too:

```bash
$ curl -s https://news.ycombinator.com/ | pup 'td.title a[href^=http] json{}'
```

## Basic Usage

```bash
$ cat index.html | pup [flags] [selectors] [optional display function]
```

or

```bash
$ pup < index.html [flags] [selectors] [optional display function]
```

## Examples

Download a webpage with wget.

```bash
$ wget http://en.wikipedia.org/wiki/Robots_exclusion_standard -O robots.html
```

####Clean and indent

By default pup will fill in missing tags and properly indent the page.

```bash
$ cat robots.html
# nasty looking HTML
$ cat robots.html | pup --color
# cleaned, indented, and colorful HTML
```

####Filter by tag
```bash
$ pup < robots.html title
<title>
 Robots exclusion standard - Wikipedia, the free encyclopedia
</title>
```

####Filter by id
```bash
$ pup < robots.html span#See_also
<span class="mw-headline" id="See_also">
 See also
</span>
```

####Chain selectors together

The following two commands are (somewhat) equivalent.

```bash
$ pup < robots.html table.navbox ul a | tail
```

```bash
$ pup < robots.html table.navbox | pup ul | pup a | tail
```

Both produce the ouput:

```bash
</a>
<a href="/wiki/Stop_words" title="Stop words">
 Stop words
</a>
<a href="/wiki/Poison_words" title="Poison words">
 Poison words
</a>
<a href="/wiki/Content_farm" title="Content farm">
 Content farm
</a>
```

Because pup reconstructs the HTML parse tree, funny things can
happen when piping two commands together. I'd recommend chaining
commands rather than pipes.

####Limit print level

```bash
$ pup < robots.html table -l 2
<table class="metadata plainlinks ambox ambox-content" role="presentation">
 <tbody>
  ...
 </tbody>
</table>
<table style="background:#f9f9f9;font-size:85%;line-height:110%;max-width:175px;">
 <tbody>
  ...
 </tbody>
</table>
<table cellspacing="0" class="navbox" style="border-spacing:0;">
 <tbody>
  ...
 </tbody>
</table>
```

####Slices

Slices allow you to do simple `{start:end:by}` operations to limit the number of nodes
selected for the next round of selection.

Provide one number for a simple index.

```bash
$ pup < robots.html a slice{0}
<a id="top">
</a>
```

You can provide an end to limit the number of nodes selected.

```bash
$ # {:3} is the same as {0:3}
$ pup < robots.html a slice{:3}
<a id="top">
</a>
<a href="#mw-navigation">
 navigation
</a>
<a href="#p-search">
 search
</a>
```

## Implemented Selectors

For further examples of these selectors head over to [MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference).

```bash
cat index.html | pup .class
# '#' indicates comments at the command line so you have to escape it
cat index.html | pup \#id
cat index.html | pup element
cat index.html | pup [attribute]
cat index.html | pup [attribute=value]
# Probably best to quote enclose wildcards
cat index.html | pup '[attribute*=value]'
cat index.html | pup [attribute~=value]
cat index.html | pup [attribute^=value]
cat index.html | pup [attribute$=value]
```

You can mix and match selectors as you wish.

```bash
cat index.html | pup element#id[attribute=value]
```

## Display Functions

Non-HTML selectors which effect the output type are implemented as functions
which can be provided as a final argument.

#### `text{}`

Print all text from selected nodes and children in depth first order.

```bash
$ cat robots.html | pup .mw-headline text{}
History
About the standard
Disadvantages
Alternatives
Examples
Nonstandard extensions
Crawl-delay directive
Allow directive
Sitemap
Host
Universal "*" match
Meta tags and headers
See also
References
External links
```

#### `attr{attrkey}`

Print the values of all attributes with a given key from all selected nodes.

```bash
$ pup < robots.html a attr{href} | head
#mw-navigation
#p-search
/wiki/MediaWiki:Robots.txt
//en.wikipedia.org/robots.txt
/wiki/Wikipedia:What_Wikipedia_is_not#NOTHOWTO
//en.wikipedia.org/w/index.php?title=Robots_exclusion_standard&action=edit
//meta.wikimedia.org/wiki/Help:Transwiki
//en.wikiversity.org/wiki/
//en.wikibooks.org/wiki/
//en.wikivoyage.org/wiki/
```

#### `json{}`

Print HTML as JSON.

```bash
$ cat robots.html  | pup div#p-namespaces a
<a href="/wiki/Robots_exclusion_standard" title="View the content page [c]" accesskey="c">
 Article
</a>
<a href="/wiki/Talk:Robots_exclusion_standard" title="Discussion about the content page [t]" accesskey="t">
 Talk
</a>
```

```bash
$ cat robots.html  | pup div#p-namespaces a json{}
[
 {
  "attrs": {
   "accesskey": "c",
   "href": "/wiki/Robots_exclusion_standard",
   "title": "View the content page [c]"
  },
  "tag": "a",
  "text": "Article"
 },
 {
  "attrs": {
   "accesskey": "t",
   "href": "/wiki/Talk:Robots_exclusion_standard",
   "title": "Discussion about the content page [t]"
  },
  "tag": "a",
  "text": "Talk"
 }
]
```

Use the `-i` / `--indent` flag to control the intent level.

```bash
$ cat robots.html  | pup --indent 4 div#p-namespaces a json{}
[
    {
        "attrs": {
            "accesskey": "c",
            "href": "/wiki/Robots_exclusion_standard",
            "title": "View the content page [c]"
        },
        "tag": "a",
        "text": "Article"
    },
    {
        "attrs": {
            "accesskey": "t",
            "href": "/wiki/Talk:Robots_exclusion_standard",
            "title": "Discussion about the content page [t]"
        },
        "tag": "a",
        "text": "Talk"
    }
]
```

If the selectors only return one element the results will be printed as a JSON
object, not a list.

```bash
$ cat robots.html  | pup --indent 4 title json{}
{
    "tag": "title",
    "text": "Robots exclusion standard - Wikipedia, the free encyclopedia"
}
```

Because there is no universal standard for converting HTML/XML to JSON, a
method has been chosen which hopefully fits. The goal is simply to get the
output of pup into a more consumable format.

## Flags

```bash
-c --color         print result with color
-f --file          file to read from
-h --help          display this help
-i --indent        number of spaces to use for indent or character
-n --number        print number of elements selected
-l --limit         restrict number of levels printed
--version          display version
```

## TODO

Add more tests!
