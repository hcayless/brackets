# Schemas and Validation 
## Part 2 (see (Part 1)[validation.md])

In Part 1, I discussed some of the basics of what a schema is, and why you'd want one. To recap, a schema gives you a way to check whether your document is following the conventions set by the TEI and by your own project. I talked a bit about why it might be tricky to obtain a schema for an arbitrary TEI document given that there are multiple kinds of schema and given that even grabbing the schema over the internet from a browser-based editor might require configuration where the schema is hosted that our TEI editor software would have no control over. In this section I'm going to talk a bit about why browser-based validation is hard, and what solutions might be available. The good news is that there probably are solutions available that didn't exist a few years ago.

Recall that there are basically four schema types: DTDs, XML Schema, Relax NG, and Schematron. The first three work by matching the content of each element in an XML document against a "content model"—a pattern. If we look at the content model of `<app>`, my favorite element, at https://www.tei-c.org/release/doc/tei-p5-doc/en/html/ref-app.html, we see:

```
element app
{
   att.global.attributes,
   att.global.rendition.attributes,
   att.global.linking.attributes,
   att.global.analytic.attributes,
   att.global.facs.attributes,
   att.global.change.attributes,
   att.global.responsibility.attributes,
   att.global.source.attributes,
   attribute type { teidata.enumerated }?,
   attribute from { teidata.pointer }?,
   attribute to { teidata.pointer }?,
   attribute loc { list { teidata.word+ } }?,
   ( lem?, ( model.rdgLike | model.noteLike | witDetail | wit | rdgGrp )* )
}
```
This is Relax NG, which has two syntaxes, compact (as above) and XML. Relax NG treats attributes as part of the content model; DTDs and XML Schema (and TEI ODD) treat them separately. If we ignore the attributes for now, we have
```
( lem?, ( model.rdgLike | model.noteLike | witDetail | wit | rdgGrp )* )
```
which might remind you of something else. Let's unpack it a little: model.rdgLike and model.noteLike are model classes, which generally contain multiple elements. In this case they each only contain one, rdg and note respectively. So we could rewrite that part of the model thus:

```
( lem?, ( rdg | note | witDetail | wit | rdgGrp )* )
```
It looks rather like a regular expression. `<app>` may contain an optional, single, `<lem>`, followed by any number of `<rdg>`, `<note>`, `<witDetail>`, `<wit>`, or `<rdgGrp>`. Just like in a regular expression, `?` means "zero or one" and `*` means "zero or more". Parentheses create groups, `|` means "or". The DTD content model looks like:
```
<!ELEMENT app (((lem)?,(%model.rdgLike;|%model.noteLike;|witDetail|wit|rdgGrp)*))>
```
(basically the same thing). The XSD like:
```xml
<xs:sequence>
  <xs:element minOccurs="0" ref="tei:lem"/>
  <xs:choice minOccurs="0" maxOccurs="unbounded">
    <xs:group ref="tei:model.rdgLike"/>
    <xs:group ref="tei:model.noteLike"/>
    <xs:element ref="tei:witDetail"/>
    <xs:element ref="tei:wit"/>
    <xs:element ref="tei:rdgGrp"/>
  </xs:choice>
</xs:sequence>
```
Again, the same, just with a different syntax. The ODD content model looks like:
```xml
<content>
 <sequence>
  <elementRef key="lem" minOccurs="0"/>
  <alternate maxOccurs="unbounded"
   minOccurs="0">
   <classRef key="model.rdgLike"/>
   <classRef key="model.noteLike"/>
   <elementRef key="witDetail"/>
   <elementRef key="wit"/>
   <elementRef key="rdgGrp"/>
  </alternate>
 </sequence>
</content>
```
Remember, the ODD lets us generate the other three. Validating in any of these cases means reading the content model and generating from it a program into which you can feed the actual content (the elements, attributes, and text) of an element, and which will tell you if the content you've given it matches the model. Typically, we'd do this kind of checking with something called a Finite State Machine (or Finite State Automaton). You can think of it as a list of "states", some of which will be "final" (i.e. if we end up in that state, we're ok). Each state has a list of possible "transitions"—basically if/then conditions—if it's presented with a particular input, then the state machine will jump to another state (it might jump back to itself)

```
start: (if 'lem' go to 'one'), (if 'rdg' or 'note' or 'witDetail' or 'wit' or 'rdgGrp' then go to 'two) [OK to stop here]
one: (if 'rdg' or 'note' or 'witDetail' or 'wit' or 'rdgGrp' then go to 'two) [OK to stop here]
two: (if 'rdg' or 'note' or 'witDetail' or 'wit' or 'rdgGrp' then go to 'two) [OK to stop here]
```
So if we want to check a randomly-chosen `<app>` element in a real document for validity, like this one

```xml
<app>
  <lem xml:id="lem1.1nondum">Nondum</lem>
  <rdg wit="#G #P" xml:id="rdg1.1nundum" ana="#orthographical">nundum</rdg>
  <witDetail wit="#G" target="#rdg1.1nundum" type="correction-original">corr. <ref
      target="#G1">G<hi rend="super">1</hi></ref></witDetail>
</app>
```
our inputs would be 'lem', 'rdg', 'witDetail', so we'd begin at the 'start' state, take our first input, which is 'lem', and jump to state 'one', then our second input, 'rdg' would have us jump to state 'two', our third input is 'witDetail', and we jump to state 'two' again. We're done with inputs, and 'two' is an "accepting" state, so it's ok to stop there. We are done, and our element is valid! Finite automata can be deterministic (DFA) or nondeterministic (NFA). You can think of the difference this way: if you think of our state machine as a graph, where the nodes are states, and the arcs are transitions, then for a given input, there is only one path through the graph in a DFA; at each step, there's only one possible transition (that's why it's deterministic). For an NFA, there might be multiple choices for a given input, so traversing the graph is more complicated—we'll have to follow all the possible paths until we either find a state where the last token in our input can be accepted, or we run out of road. Because this is software, when our two roads diverge, we can simply clone ourselves and take both. The difference from a DFA being that we incur a (small) cost in energy and memory usage for each additional path taken. To help us think about nondeterminism, we might try rewriting our content model a little:

```
( lem?, rdg?, ( rdg | note | witDetail | wit | rdgGrp )* )
```
which would be compiled to something like:
```
start: (if 'lem' go to 'one'), (if 'rdg' or 'note' or 'witDetail' or 'wit' or 'rdgGrp' then go to 'two) [OK to stop here]
one: (if rdg then go to 'two'), (if 'rdg' or 'note' or 'witDetail' or 'wit' or 'rdgGrp' then go to 'three) [OK to stop here]
two: (if 'rdg' or 'note' or 'witDetail' or 'wit' or 'rdgGrp' then go to 'three) [OK to stop here]
three: (if 'rdg' or 'note' or 'witDetail' or 'wit' or 'rdgGrp' then go to 'three) [OK to stop here]
```

Now, when we're in state 'one', and our next token is 'rdg', we have to make a choice: do we go to 'two', or to 'three'? In this case, in the end, it makes no difference. Either way will work, and whichever of our clones happens to get the answer first will win. It does make a big difference to our choice of schema though. If we try this with a DTD and attempt to validate it using the program `xmllint`, we will get the error:
```
validity error : Content model of app is not determinist: (lem* , rdg? , (rdg | note | witDetail | wit | rdgGrp)*)
```
If we try it with XML Schema, we will get a much more confusing, but equivalent error (something about violating "Unique Particle Attribution"—could you maybe be a little *more* obscure? Thanks).

Relax NG is perfectly happy with this content model, by the way. So two schema types require deterministic content models, and one doesn't care, and we can see that the process of checking against a content model involves well-understood algorithms. You will note that our modified content model doesn't make sense—adding an optional `<rdg>` doesn't change what sorts of input would be valid or not, so maybe it will seem intuitive that the nondeterministic content model can be simplified to a deterministic one. In fact, any NFA can be transformed into a DFA. The only downside being that doing so adds states, in the worst case exponentially, to the DFA. 

All of this tells us that the act of validation isn't super hard (even though there's more to it than I've outlined here), so, given that there may not be good XML validators that will run in a web browser, why not just write our own? It's complicated by the various schema types, and Relax NG has two syntaxes, one "compact" which we've seen, and one XML based, that looks a bit more like ODD. So that's four different sytaxes to understand, and three different grammars. It's a lot. TEI people tend to use Relax NG almost exclusively though, so we could say we'll just support Relax NG. Moreover although ODD, like Relax NG, supports nondeterministic content models, the baseline TEI developed by the Consortium only uses deterministic ones—because we need to be able to generate DTD and XML Schema versions of the schemas we publish. TEI ODD also does not at this time exploit all of the capabilities of Relax NG, so we wouldn't need a complete implementation of the specification. And it's worth noting that an online editor's validation does not need to be as fast and scaleable as the enterprise-ready Open Source versions, [libxml2](http://www.xmlsoft.org/) (written in C) and [Jing-Trang](https://github.com/relaxng/jing-trang) (written in Java).

Before we go diving into writing our own validator, it's worth taking a look around to see what new toys the internet might have thrown our way. There have been huge advances in porting programs written in other languages to browser-executable versions using [WebAssembly](https://webassembly.org/) as a build target, and in fact, there is a port of libxml2 available at https://github.com/kripken/xml.js, which we should evaluate. This would, if it works, give us DTD, XML Schema, and Relax NG validation in the browser, which would be super. As ever, though, things are not as simple as we might like: this does nothing for Schematron, and there are signs, if we look at the repository's issues, that it may not work properly with non-Latin characters, which would be a deal-breaker for this project.

To conclude this episode: there may be a way forward towards validating TEI documents in our still theoretical editor, and if it doesn't work, I might have some useful thoughts on building a sufficient validator. Note that the ported libxml2 solution doesn't help us with providing schema-aware help to our users, and it won't help us with Schematron at all. It might fail to handle documents with alphabets outside the [ASCII](https://en.wikipedia.org/wiki/ASCII) range, or outside the Unicode [Basic Multilingual Plane](https://en.wikipedia.org/wiki/Plane_(Unicode)#Basic_Multilingual_Plane). It might take a Gigabyte of memory to run. It might make your laptop hot enough to cook an egg on. We shall see. 

To be continued...
