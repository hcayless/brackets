# Experiment: is the JavaScript port of libxml2 viable?

As we go along, it will obviously be necessary to test things, and I want to do this as openly as possible. Partly so you can experience along with me the true horrors of how broken everything is, but also share in whatever successes we have. In my last [post](validation-2.md), I mentioned the possibility that we might be able to use libxml2 itself, and its incredibly useful command-line program xmllint to do XML validation for us. I am sad to report that the answer is a qualified fail. In fact, it's a fail in the worst way possible: it does not work right out of the box, but we might, possibly, with some (or maybe considerable) effort, be able to fix it. 

I've checked in the code I used for the experiment in the [experiments/1_XML.js](docs/experiments/1_XML.js) folder, and it's available at https://hcayless.github.io/brackets/experiments/1_XML.js/ Here's what I did:

I set up a basic editing environment. It uses [CodeMirror](https://codemirror.net/) as the editor. CodeMirror is not necessarily the final choice for our TEI editor, it's just convenient and easy to use. We'll explore alternatives later. I created an index.html page with the code to load up the editor and xmllint.js, which I grabbed from https://github.com/kripken/xml.js. The page automatically loads a fairly complex sample document and its Relax NG schema. It displays a "validate" button, which should turn green when clicked in the case of success or red for failure, in which case it will also display a list of errors. The document is valid against its schema (at least Jing, the validator embedded in oXygen says so). For all of this to run, we'll need it to come from a web server. There are various ways to do this, but if you have Python installed, you can do what I did and run `python -m SimpleHTTPServer 8000` from inside the experiments/1_XML.js directory.

I've said it doesn't work. Here's how it doesn't work: attempting to validate the default document against its schema fails with some 200-odd messages in the form `RNG internal error trying to compile notAllowed`. It can't read our schema. Trying this with the command line version produces the same set of error messages, but at the end says:

> editio.xml validates

So what's happening amounts to a large set of errors parsing the schema (which is correct), but which don't prevent the validation from happening. This doesn't help us, because the way XML.js wants us to check success is by seeing whether there is an `errors` object returned. As always in these situations, I resort to Googling the error message, which yields few results: a post in 2008 from one [Syd Bauman](https://github.com/sydb) on the libxml2 support list, unanswered; some complaints from the TEI Simple development repository about the same thing; a vague hint in some release notes that someone at Google may have patched this for Android, but if so, it never made it back to the source. 

The `notAllowed` construct in Relax NG is used when merging schemas—it essentially disallows elements or attributes which may be referenced as part of groups. It's useful because it means if you're switching off features of a schema, you don't have to also make the effort to simplify or rewrite every content model that references that feature. Libxml2 does not like it. Okay, what if we use a schema that doesn't have `notAllowed`? What if we just use TEI All, which by definition has everything? Another problem:

> Access to XMLHttpRequest at 'https://tei-c.org/Vault/P5/3.6.0/xml/tei/custom/schema/relaxng/tei_all.rng' from origin 'http://localhost:8000' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.

You may remember I identified this as a potential problem back in the first post on [validation](validation.md). The TEI server is supposed to allow CORS, and with luck. by the time you read this, it will be again, but that bit of configuration seems to have been lost in our recent move to a new hosting provider, [Huma-Num](https://www.huma-num.fr/). So we can't load our schema from the server. No matter, I can just download it, put it next to my index.html, and load it from there...

> Cannot enlarge memory arrays. Either (1) compile with  -s TOTAL_MEMORY=X  with X higher than the current value 16777216, (2) compile with  -s ALLOW_MEMORY_GROWTH=1  which adjusts the size at runtime but prevents some optimizations, (3) set Module.TOTAL_MEMORY to a higher value before the program runs, or if you want malloc to return NULL (0) instead of this abort, compile with  -s ABORTING_MALLOC=0 

In other words, TEI All is too big. We could probably make it work by re-compiling xmllint.js and changing the TOTAL_MEMORY settings. This would not make it work with our preferred schema. Incidentally, if I generate an XML Schema and attempt to validate using *it*, I get: 

> warning: failed to load external entity "dcr.xsd"
>
> file_0.xsd:3: element import: Schemas parser warning : Element '{http://www.w3.org/2001/XMLSchema}import': Failed to locate a > schema at location 'dcr.xsd'. Skipping the import.
>
> warning: failed to load external entity "xml.xsd"
>
> file_0.xsd:4: element import: Schemas parser warning : Element '{http://www.w3.org/2001/XMLSchema}import': Failed to locate a > schema at location 'xml.xsd'. Skipping the import.
>
> file_0.xsd:509: element attribute: Schemas parser error : attribute use (unknown), attribute 'ref': The QName value '{http://> www.isocat.org/ns/dcr}datcat' does not resolve to a(n) attribute declaration.
>
> file_0.xsd:512: element attribute: Schemas parser error : attribute use (unknown), attribute 'ref': The QName value '{http://> www.isocat.org/ns/dcr}valueDatcat' does not resolve to a(n) attribute declaration.
>
> file_0.xsd:917: element attribute: Schemas parser error : attribute use (unknown), attribute 'ref': The QName value '{http://> www.w3.org/XML/1998/namespace}id' does not resolve to a(n) attribute declaration.
>
> file_0.xsd:927: element attribute: Schemas parser error : attribute use (unknown), attribute 'ref': The QName value '{http://> www.w3.org/XML/1998/namespace}lang' does not resolve to a(n) attribute declaration.
>
> file_0.xsd:930: element attribute: Schemas parser error : attribute use (unknown), attribute 'ref': The QName value '{http://> www.w3.org/XML/1998/namespace}base' does not resolve to a(n) attribute declaration.
>
> file_0.xsd:933: element attribute: Schemas parser error : attribute use (unknown), attribute 'ref': The QName value '{http://> www.w3.org/XML/1998/namespace}space' does not resolve to a(n) attribute declaration.

The schema relies on a couple of small, external files, which the JavaScript version of xmllint cannot load. Everything is broken.

I think there's a workaround I can use for the XML Schema validation, but it's going to be complicated for the general case, where I just want to load a schema from a URL, and furthermore, I didn't really want to support XML Schema that much anyway, especially not in preference to Relax NG. 

This all nicely highlights some of the ways building software is difficult. It looks like there is something that will do most of what I want, that I can have for free, but using it is complicated by bugs in the underlying software and by the altered environment (the web instead of the filesystem) in which it's being called. Maybe I can still have it, but not for free. Maybe I can fork XML.js and rewrite some bits of it so it will work for me, but that will mean instead of being able to rely on code maintained by someone else, I will be responsible for maintaining it. This is one way project scopes explode. It's also why developers may get that haunted look when you say "Why can't we just use X?"

Result: Inconclusive; lean towards endless screaming.
