I've done some research on this topic.

### How it works now

The process of documentation generation is roughly like this:

1. Command line arguments are parsed.  It tells which textual files should be added to YARD documentation as so called extra file objects.
2. Meta data (title, author) is extracted from extra file objects by using a few simplistic regexps.
3. Software source files (*.rb, *.c) are parsed, and some kind of intermediary source code documentation is generated.
4. For each **item** to be documented (be it module, method, or extra file object)
    * Generate final documentation for that **item**, which involves running relevant markup processor like Redcarpet, RDoc or Asciidoctor.

In this algorithm, extra file object titles must be known before step 4. begins.  This is because the final documentation may contain some navigation controls, which may include links to any other extra file object.  This is particularly true in case of HTML output.  Consequently, document metadata cannot obtained from AsciiDoc processor (via its `Asciidoctor::Document#attr` API method).

### What we can do about it

#### Option 1: Enhance regexp

Regular expressions used in step 2 can be improved to parse AsciiDoc's attribute syntax.  However, due to the complexity of AsciiDoc syntax, it is not easy to write proper regular expression.  Consider following example:

```adoc
= Some Document

:arbitrary-attr: This is value for the "arbitrary-attr" attribute
:yard-title: This is value for the "yard-title" attribute

Lead paragraph

[source]
--------------------------------------------------------------------------------
:yard-title: This is just a code sample, not an attribute definition for
the current document.
--------------------------------------------------------------------------------
```

In the above example, `/^:yard-title:/` regular expression matches twice, which is not what we want, because the last occurrence isn't actually an attribute definition.

Therefore, I propose adding an additional requirement: that YARD-specific attributes must be defined before the first line which is not a section title nor attribute definition.  I don't think that any fancy techniques are really needed in these attributes, therefore in my opinion such constraint is acceptable.

#### Option 2: Run Asciidoctor twice

Asciidoctor processor is run for the first time in step 2, and then again in step 4.  The first run does not produce any output document, it is only to parse the source document and to extract YARD-specific attributes.  This is a full-featured solution, which honours AsciiDoc's conditional processing, include directives, etc.  No special constraints are required.  Obviously some overhead is involved, but IMHO it is acceptable.

#### Option 3: Render contents of extra file objects earlier

Markup processors for extra file objects are run at step 2 or 3 instead of step 4.  Generated output is stored in a variable and merged into template at step 4.

----

@mojavelinux, @lsegal, thoguhts?