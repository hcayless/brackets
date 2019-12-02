# Prolegomena to Creating an Online TEI Editor

I've thought for a few years now that it would be a very good thing if there was a decent, online, free TEI editor, but I've been reluctant to actually tackle the project for a few reasons. The reasons for wanting such a thing are pretty compelling. Current support for TEI editing is only really adequate in oXygen, which is a proprietary XML editor. Its pricing is quite reasonable (at least for a North American / Western European academic audience), but that leaves out a lot of people, actually. This sets up a pretty hefty barrier to learning TEI for students and for academics outside the circle of wealthy countries. Moreover, an online editor would be a very nice feature for many projects that might want to allow online editing, and it would be super to have a no-installation-required TEI editing setup available for teaching TEI. So there are good reasons to to it. Why might I be intimidated by it? After all, there are several good browser-based code editors available, even ones that do nice things like XML syntax highlighting. I'll start by enumerating some of the things I'd want to see in an online TEI editor:

1. It should do syntax checking and at least some level of validation.
2. It should provide some level of help to users. Ideally it should suggest what elements you can insert at a particular point.
3. It should at minimum have a basic mode where all tags are visible.
4. It should have decent multilingual support—ideally including RTL (right-to-left), even better mixed LTR and RTL.

I'd add a couple more infrastructural requirements:

5. Ideally, no server-side processing should be required. At least it should be kept to a minimum.
6. It should be reasonably easy to deploy (i.e. you don't have to be an expert coder to get it going).
7. It should be as low-maintenance as possible.

This seems likely to turn into a multi-part series of mini-essays, hopefully one that will gradually add bits of working code, and maybe even a functioning editor. A few years ago, I was involved in a similar effort led by people at MITH (which ultimately failed). It was called "[Angles](https://securegrants.neh.gov/publicquery/Download.aspx?data=EbwGdSyLkD7zoB3W75cvd%2bXST%2bWypC%2blxN287EYCLJwL%2fBHyeaDHU3RDLrfbJLx%2b1ItCBTsjaJWE63FMYJfoM3kRWI6WIx3tbxFY0YxJWCyrSmv4VZBvmZMmZD6Yx6Oo%2fKNMeiAmM31gZyGwdni6YQ%3d%3d)", so I'm tentatively calling this thing "Brackets".

I have the following topics in mind, and they will get filled in as I get to them. No promises about having a completed TEI editor here, or even a complete set of posts, but I'll do what I can.

* Schemas and validation
  * [Part 1](validation.md)
  * [Part 2](validation-2.md)
  * [Experiment 1](experiment_1.md)
* Why not a tagless editor?
* Multilingual support
* Web vs. desktop