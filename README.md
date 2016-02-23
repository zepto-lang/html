# html

A library for parsing and building HTML in zepto.
It is minimal and quite possibly buggy in more complex cases.
More about that in the [caveats](#caveats) section.

## Usage

The module exports four endpoints, `html:doctype`,
`html:build`, `html:with-skeleton` and `html:parse`.
Those endpoints can be used like so:

```clojure
; html:parse will parse HTML into a hash-map.
; Its capabilities are somewhat limited (see caveats)
; More about the format in format
(html:parse "<html><body><p class='section'>Hi</p></body></html>")
; => #{:tags (#{:name "html"
;               :tags (#{:name body
;                        :tags (#{:name "p"
;                                 :attributes #{"class" "section"}
;                                 :text "Hi"})})})}

; html:build will build HTML from such hash-maps.
; as such, it  is inverse to parse
(html:build #{:tags (#{:name "html"
                       :attribues #{"class" "not-full-screen"}
                       :tags (#{:name "body" :text "empty"})})})
; => <html class="not-full-screen"><body>empty</body></html>

; html:with-skeleton abstracts a bit over the admittedly
; tedious build procedure
(html:build (html:with-skeleton
  #{:head #{:tags (#{:name "script" :attribute #{"type" "text/javascript"} :text "alert(\"yay\");"})}
    :body #{:tags (#{:name "p" :text "notifications are annoying"})}}))
; => <html><head><script type="text/javascript">alert("yay");</script></head><body><p>notifications are annoying</p></body></html>

; html:doctype is the single helper function that is exposed
; it builds a DOCTYPE node
(html:doctype) ; => <!DOCTYPE html>
(html:doctype "foo-std") ; => <!DOCTYPE foo-std>
```

## Format

The hash-map format that is emitted by the parser (and consumed by the builder, respectively)
looks as follows:
````clojure
top-level = #{:tags [tag] :text string}
tag = #{:name string :attributes hash-map :text string :tags [tag]}
```

When building, the attributes and child tags can be omitted if they are empty and single.
The parser will always emit complete data, because it is both easier for the
library and its users to handle homogenous data.

## Caveats

The library is far from finished. As such, most even slightly fancy features of HTML
are not present in the builder.

### Parsing

Right now, the parsing only supports a subset of HTML that will suffice to
parse most HTML sites. No special nodes are understood, i.e. only regular or self-closing,
implicitly closed and comment tags are allowed. Special node (such as DOCTYPE) have only
rudimentary support and are treated as simple key-value nodes.

###Building

In the build process, there is no such thing as self-closing or implicitly closing tags.
All tags will behave like regular XML tags, which could lead to quite a bit of frustration
I imagine. Comment nodes don't exist here either.
