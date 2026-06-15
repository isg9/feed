---
title: 'research!rsc: Data Structures Go Programs'
url: https://research.swtch.com/godata2
published: "2009-12-08T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/godata2
---

# research!rsc: Data Structures Go Programs

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# Data Structures Go Programs Russ Cox December 8, 2009 *research.swtch.com/godata2* Posted on Tuesday, December 8, 2009.

I've been writing about Go internals, which I'll keep doing, but I also want to spend some time on Go programs. Go's [reflect package](http://golang.org/pkg/reflect/) implements data reflection. Any value can be passed to `reflect.NewValue` (its argument type is `interface{}`, the empty interface) and then manipulated and examined by looking at the result, a `reflect.Value`. Typically, you use a [type switch](http://golang.org/doc/go_spec.html#Type_switches) to determine what kind of value it is—a `*IntValue`, a `*StructValue`, a `*PtrValue`, and so on—and then walk it further. This is how [`fmt.Print`](http://golang.org/src/pkg/fmt/print.go?h=doprint#L849) prints arbitrary data passed to it.

Go allows programs to tag the fields in struct definitions with annotation strings. These annotations have no effect on the compilation but are recorded in the run-time type information and can be examined via reflection. The annotations can be hints for reflection-driven code traversing the data structures. This blog post shows two examples of such code: an XML parser that writes directly into data structures, and a template-based output generator that reads from them.

### An XML Linked List

The Go [XML package](http://golang.org/pkg/xml/) implements a function `Unmarshal` which parses XML directly into data structures. As a contrived but illustrative example, here is an XML encoding of a linked list:

```
var encoded = `
   a
       b
           c



`

```

(That's a backquoted multi-line Go string literal.)

Given this Go data structure definition:

```
type List struct {
	Value string
	List  *List
}

func (l *List) String() string {
	if l == nil {
		return "nil"
	}
	return l.Value + " :: " + l.List.String()
}

```

we can invoke `xml.Unmarshal` to turn the XML above into an instance of this data structure:

```
func main() {
	var l *List
	if err := xml.Unmarshal(strings.NewReader(encoded), &l); err != nil {
		log.Exit("xml.Unmarshal: ", err)
	}
	fmt.Println(l)
}

```

This program prints `a :: b :: c :: nil`. The call to `xml.Unmarshal` walked the XML and the data structure at the same time, matching the XML tags `and` against the names of the structure fields and allocating pointers where necessary.

There are decisions to be made when matching XML against the data structure that aren't easily expressed with just the usual struct information. For example, we might want to make the value an attribute instead of a sub-element. Because attributes and sub-elements might use the same names, the XML package requires struct fields holding attributes to be tagged with `"attr"`. This data structure and XML encoding would also work in the program above:

```
type List struct {
	Value string "attr"
	List  *List
}

var encoded = `





`

```

If a field has type `string`, it gets the character data from the element with that name, but sometimes you need to accumulate both character data and sub-elements. To do this, you can tag a field with `"chardata"`. This data structure and XML encoding would also work:

```
type List struct {
	Value string "chardata"
	List  *List
}

var encoded = `abc`

```

The annotations are trivial but important: where the encoding and the data structures are not a perfect match (and they rarely are), the annotations provide the hints necessary for the XML package to make them work together anyway.

### Code Search Client

Let's look at real XML data. Google's [Code Search](http://www.google.com/codesearch) provides [regular expression search](http://swtch.com/~rsc/regexp/regexp1.html) over the world's public source code. The search \[ [file:/usr/sys/ken/slp.c expected](http://www.google.com/codesearch?q=file:/usr/sys/ken/slp.c+expected)\] returns [a single, famous result](http://www.google.com/codesearch/p?hl=en#miRTe8ZyR0o/Archive/PDP-11/Trees/V6/usr/sys/ken/slp.c&q=file:/usr/sys/ken/slp.c%20expected&sa=N&cd=1&ct=rc&l=264). Instead of using the web interface, though, we can issue the same query using the [Code Search GData API](http://code.google.com/apis/codesearch/docs/2.0/reference.html) and get [this XML](http://www.google.com/codesearch/feeds/search?q=file:/usr/sys/ken/slp.c+expected) back (simplified and highlighted for structure):

```
 version="1.0" encoding="UTF-8"?>
 xmlns="http://www.w3.org/2005/Atom"
     xmlns:gcs="http://schemas.google.com/codesearch/2006"
     xml:base="http://www.google.com">
 http://www.google.com/codesearch/feeds/search?q=file:/usr/sys/ken/slp.c+expected
 2009-12-04T23:49:22Z
  type="text">Google Code Search
   version="1.0" uri="http://www.google.com/codesearch">Google Code Search

    name="http://www.tuhs.org" uri="http://www.tuhs.org">
    name="Archive/PDP-11/Trees/V6/usr/sys/ken/slp.c">
    lineNumber="325" type="text/html"><pre>  * You are not <b>expected</b> to understand this.
</pre>


```

**From the API documentation or just by eyeballing, we can write** **data structures for the information we'd like to extract:**

**```** **type Feed struct {** **XMLName xml.Name "http://www.w3.org/2005/Atom feed"** **Entry []Entry** **}**

**type Entry struct {** **XMLName xml.Name "http://www.w3.org/2005/Atom entry"** **Package Package** **File File** **Match []Match** **}**

**type Package struct {** **XMLName xml.Name "http://schemas.google.com/codesearch/2006 package"** **Name string "attr"** **URI string "attr"** **}**

**type File struct {** **XMLName xml.Name "http://schemas.google.com/codesearch/2006 file"** **Name string "attr"** **}**

**type Match struct {** **XMLName xml.Name "http://schemas.google.com/codesearch/2006 match"** **LineNumber string "attr"** **Type string "attr"** **Snippet string "chardata"** **}**

**```**

**The `XMLName` fields are not required.** **The XML package records the XML tag name in them.** **If there is a string annotation (as there is here),** **then the XML package requires that the XML element being** **considered has the given name space and tag name.** **That is, the annotation is both documentation and** **an assertion about what kind** **of XML goes into this structure.**

**A full Atom GData feed has many other fields,** **but these are the only ones that we care about.** **The XML parser will throw away data that doesn't fit into** **our structures.**

**We can fetch the feed via HTTP and unmarshal** **the HTTP response data directly into the data structures:**

**```** **r, _, err := http.Get(`http://www.google.com/codesearch/feeds/search?q=` +** **http.URLEscape(flag.Arg(0)))** **if err != nil {** **log.Exit(err)** **}**

**var f Feed** **if err := xml.Unmarshal(r.Body, &f); err != nil {** **log.Exit("xml.Unmarshal: ", err)** **}**

**```**

**and then print the results:**

**```** **for _, e := range f.Entry {** **fmt.Printf("%s\n", e.Package.Name)** **for _, m := range e.Match {** **fmt.Printf("%s:%s:\n", e.File.Name, m.LineNumber)** **fmt.Printf("%s\n", cutHTML(m.Snippet))** **}** **fmt.Printf("\n")** **}**

**```**

**(The `cutHTML` function returns its argument** **with HTML tags stripped: the** **`m.Snippet` is HTML but we just want to print text.)**

**Compared to the `xml.Unmarshal`, that loop** **with all the `fmt.Printf` calls sure looks clunky.** **We can use a reflection-driven process to print the data too.** **The [template package](http://golang.org/pkg/template/) is a Go implementation** **of [json-template](http://code.google.com/p/json-template/).** **We can write a template like:**

**```** **var tmpl = `{.repeated section Entry}** **{Package.Name}** **{.repeated section Match}** **{File.Name}:{LineNumber}:** **{Snippet|nohtml}** **{.end}**

**{.end}`**

**```**

**and then print the data using that template:**

**```** **t := template.MustParse(tmpl, template.FormatterMap{"nohtml": nohtml})** **t.Execute(&f, os.Stdout)**

**```**

**The formatter map definitions arrange that the** **`{Snippet|nohtml}` in the template run the** **data through the `nohtml` function when printing.** **Both variants of our program produce the same output:**

**```** **http://www.tuhs.org** **Archive/PDP-11/Trees/V6/usr/sys/ken/slp.c:325:** *** You are not expected to understand this.**

**```**

**### Parsing HTML**

**I still have three more tricks up my sleeve.**

**First, in a pinch and if the HTML is** **mostly well formatted or you're just lucky, the XML parser** **can handle HTML too.** **Second, when the XML parser walks into a struct field, if the field** **is a pointer and is nil, it allocates a new value, as we** **saw in the linked list example. But if the field is a non-nil** **pointer already, the parser just follows the pointer.** **The same goes for growing slices.** **Third, if the XML parser cannot find a struct field for a particular** **subelement, before giving up it looks for a field named `Any`** **and uses that field instead.**

**Combined, these three rules mean that this structure:**

**```** **type Form struct {** **Input []Field** **Any *Form** **}**

**type Field struct {** **Name string "attr"** **Value string "attr"** **}**

**```**

**can be used to save a tree of form and other** **information from a web page. When you unmarshal into it,** **you get a tree showing the structure of the web page,** **with each element becoming a `*Form`** **because the parser used the `Any` field.**

**But who wants to walk that tree just to** **find the form fields?** **Instead, we can set the `Any` pointer to** **point back at the `Form` it is in.** **Then as the XML parser thinks it is walking the data structure,** **in fact it will be visiting the same `Form`** **struct over and over.** **Each time it finds an ``, the parser** **will append it to the `Input` slice.**

**```** **func main() {** **r, _, err := http.Get(`http://code.google.com/p/go/`)** **if err != nil {** **log.Exit(err)** **}**

**var f Form** **f.Any = &f**

**p := xml.NewParser(r.Body)** **p.Strict = false** **p.Entity = xml.HTMLEntity** **p.AutoClose = xml.HTMLAutoClose** **if err := p.Unmarshal(&f, nil); err != nil {** **log.Exit("xml.Unmarshal: ", err)** **}**

**for _, fld := range f.Input {** **fmt.Printf("%s=%q\n", fld.Name, fld.Value)** **}** **}**

**```**

**This program prints:**

**```** **q=""** **projectsearch="Search projects"**

**```**

**### Further Reading**

**There are a few more examples of this technique in** **the standard Go libraries.** **The net package's [DNS client](http://golang.org/src/pkg/net/dnsmsg.go)** **uses this technique to build generic packing and unpacking routines for DNS packets.** **The [JSON package](http://golang.org/pkg/json/) provides approximately the same** **functionality as the XML package, but with JSON as the wire format.** **In fact, JSON is arguably [more suited](http://golang.org/pkg/xml/#Bugs) to the task.** **(Remember: “the [essence of XML](http://homepages.inf.ed.ac.uk/wadler/papers/xml-essence/xml-essence.pdf) is this: the problem it solves is not hard, and it does not solve the problem well.”)** **The [ASN1 package](http://golang.org/pkg/asn1/) also works this way, though its generality** **is limited: it implements just enough to parse SSL/TLS certificates.** **There are also [other ways to use reflection](http://golang.org/search?q=NewValue), which will** **have to wait for future posts.**

**It's actually a little surprising how often this technique gets** **used in the Go libraries. Part of the reason is that it's a shiny new hammer,** **but it seems to be a very useful shiny new hammer,** **because so much of modern programming is talking to** **other programs over a variety of wire encodings.**

**Unmarshalling directly into data structures via reflection, with just a few necessary** **hints as annotations, can be implemented in other languages,** **but the [examples I've seen](http://code.google.com/p/gdata-python-client/source/browse/trunk/src/gdata/codesearch/__init__.py) are not** **nearly as elegant or as concise as the Go versions.**

***[Apologies to Jon Bentley](http://books.google.com/books?id=kse_7qbWbjsC&lpg=PA21&ots=DfwZDUFT3s&dq=%22data%20structures%20programs%22%20bentley&pg=PA21#v=onepage&q=&f=false).***

**(Comments originally posted via Blogger.)**

**- [bflm](http://www.blogger.com/profile/04562789893798531364)(December 8, 2009 12:18 PM) Hello Russ,**

**thanks for your inspiring blog posts as for your great involvement in Go.**

**I've been trying the xml.Unmarshal just a few days ago and had to give up this easy way due to a problem with attributes like some-attrib="something". The dash is obviously an illegal character for an identifier of a struct field in which such attrs I wanted to get stored by Unmarshal. Looking at the code of xml/read.go around line 272 it seems to me, that this situation is not addressed.**

**One possible remedy could or may be introducing an attr tag "syntax" of "attr attr-name", which is similar to encoding of the XMLName tag, with the "real" name at its end (after the namespace). By the way, does the later solve the analogous situation with dash-in-tagname xml tags also?**

**I'm aware that I've maybe just misunderstood something.**

**Thank you in advance for any hints/solutions and please forgive me my non-native-speaker English.**
