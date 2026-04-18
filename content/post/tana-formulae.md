---
title: "Hidden Tana Feature: Formula Fields"
date: 2026-04-18T15:44:21+02:00
draft: false
tags: ['tana']
---

_If you're just interested in the how-to -- you can [jump past my context](#core-content)._  
_If you're just looking for the formula function reference, you can [jump to that, too](#functions)._

[Tana Outliner](https://outliner.tana.inc/) is a pretty cool piece of note-taking/"personal knowledge management " software -- despite their heavy advertisement of AI interactivity -- which manages mostly to deliver on the promise of letting you do very little work regarding _data capture_, while still making _data retrieval_ possible. 

It's nowhere near as "infinitely flexible" as tools like [Obsidian](https://obsidian.md/); but in trading off that flexibility, it also frees you from thinking of things like "Where in my filesystem did I put that note?". Instead, it's an ad-hoc "knowledge database" that successfully shapes relations around small bits of trivial data entry.

This database model makes you _feel_ like Tana should be the kind of database you can easily query for things like _"How much money did I spend on projects that I started but didn't finish, this year?"_, or -- if you're like my wife -- _"How many Smash matches did I win while playing as Corrin?"_. 

Tana features [live search nodes](https://outliner.tana.inc/docs/search-nodes) that can help you do simple queries like "show all of the notes _tagged #project_ with a _status field_ set to _unfinished_ and a _start date_ within _this year_, and sum the _cost_ column"; and that usually works pretty well -- but it doesn't given you much control over how the data is displayed, or allow you to have fields that are **based on the values of other fields**. This seems like a pretty glaring omission, especially when you look at the competition from omnipresent products like Notion.

Luckily, most of Tana's internal features are implemented _in Tana_, even when they're not quite surfaced that way. That means that we can take advantage of the same internals that drive things like Tana's Live Search -- and its _sum columns_ to squeeze a bit more out of Tana.

## Formula 'Fields' {#core-content}

To accomplish fields whose values are _based on other fields_, we'll start off by creating a Tana _field_, e.g. by using the shortcut '>' on a new node. Then, we'll do something that's surprisingly well-surfaced for how undocumented it is: we'll open the command palette (e.g. with Ctrl+K or Cmd+K) and then select "Set Formula = Yes".

{{<figure src="/post-media/tana-formulae/formula-node-new.png" 
    alt="close up of a tana figure now bearing a new icon that appears as an 'fx', titled 'Show forumla' on a screen reader">}}

Note that this field no longer has a value to be edited: while we can give it a title, the area to the right of the 'fx' _show formula_ button has changed into something that will -- in a moment -- show the output of the formula we're creating.

The real magic happens when we click that _show formula_ button -- and the field suddenly becomes editable again. Now, instead of entering normal data, Tana will process each node as a string _function name_; with function arguments provided as _children_ of the node _containing the function name_. Spoiler alert: the available functions aren't documented in Tana, but I've [documented them below](#functions).

Let's say we had two existing fields -- we'll call them _rectangle length_ and _rectangle width_ -- and we wanted to compute the _rectangle area_, which we can get by multiplying these two fields together. We'll use the multiply function, which we can enter by typing in the `multiply` function ( or its shorthand of a single `*`) as our function node:

{{<figure src="/post-media/tana-formulae/formula-node-asterisk.png" 
    alt="close up of two fields -- rectangle length with a value of 10, and rectangle width with a value of 20, and our formula field, in edit mode, with just an asterisk function name as its value">}}

This _function node_ will multiply any values present in its _children_ -- which can be any of three things:

- **constant values**, which we enter in directly, such as `3.1415`;
- **references to values**, which we enter in as typical Tana [node references](https://outliner.tana.inc/docs/nodes-and-references); or
- **additional functions**, which then can have their own arguments as their own children.

In our example, we'll add in two _references_ to the field values captured above by holding _alt_ (_option_ on macOS) and dragging the field values into our formula:

{{<figure src="/post-media/tana-formulae/formula-node-references.png" 
    alt="our same formula node, but now with two Tana references to the _10_ and _20_ we had entered above as children of the asterisk node">}}

If we then exit formula-entry mode by clicking the _show formula_ button again, we're now greeted with a cheerful multiplication of our two values:

{{<figure src="/post-media/tana-formulae/formula-node-simple-multiply.png" 
    alt="our formula node now shows the value '200', a multiplication of our references">}}

Of course, this isn't nearly as useful as it could be -- since those references always refer to those two particular field _values_. What if we want to use this in a template, e.g. in the content we specify for a [supertag](https://outliner.tana.inc/docs/supertags)?

## Referencing other Fields

Fortunately, Tana has functions that allow us to look up the value of a _particular field_ on a relevant node, which is aptly named `lookup`. Instead of the direct references to our _field values_, we can return to _show formula_ mode, and insert a node with the text `lookup`:

{{<figure src="/post-media/tana-formulae/formula-node-lookupField-empty.png" 
    alt="the same fornula node, but with both references replaced by the simple string 'lookup'">}}

_*Note:* I've used `multiply` instead of an asterisk, here -- but the two are aliases, and thus equivalent. You can use either one, and in this document I'll alternate between them to drive that point home._

Since `lookup` is a function that takes an argument itself, we'll need to child nodes to each `lookup` indicating which field we'd like to get the value of. These require _field references_ -- which we can get in one of two ways:

- We can start the child node with the `@` symbol, opening an autocomplete dialog that will help us search for the field reference; or
- We can click on the relevant field, open the command palette, and then select _Copy node link_. We can then paste the copied reference into our target field.

We'll add _field references_ to our length and width fields, each specified as children ('arguments') of our `lookup` function node:

{{<figure src="/post-media/tana-formulae/formula-node-lookupField-filled.png" 
    alt="the same formula node, but with both 'lookup' functions now having a field reference as a child node">}}

If we exit _show formula_ mode, we now have successfully computed the area of our rectangle -- and using only the local field values!

{{<figure src="/post-media/tana-formulae/formula-node-lookupField-complete.png" 
    alt="our completed formula node, now showing that it's computed 10 * 20 to 200">}}

_*Note*: technically, `lookup` takes two arguments as child nodes; but the second is optional. Helpfully, if not provided, it defaults to referencing the _parent node_ that our formula field belongs to; equivalent to providing the `SELF` function defined below as our second argument._

#### Formulas work live

A cool feature of _formula fields_ is that the work live -- which means that if we click on after our _rectangle width_ of 20 and backspace away the last zero, Tana instantly updates our derived field:

{{<figure src="/post-media/tana-formulae/formula-node-lookupField-tweaked.png" 
    alt="our formula node updates live, so it now succesfully shows that 10 * 2 is 20">}}


#### Formulas can reference formulas

Another cool feature of _formula fields_ is that they're still fields -- which means we can use them directly in other computations. So, if we wanted to, we could easily figure out the volume of a _prism_ with a base of this rectangle:

{{<figure src="/post-media/tana-formulae/formula-prism-source.png"
    alt="the same formula components, but we've added a 'prism height' field with a value of 2, and then a new formula node that computes the product of a LOOKUP of our original formula node, and the new field we added">}}

If we exit out of _show formula_ mode, we can see that everything's worked exactly as we'd hoped:

{{<figure src="/post-media/tana-formulae/formula-prism-result.png"
    alt="exiting show formula mode shows the result we'd hope for: 20 * 10 * 2 = 400">}}

# Formula Functions {#functions}

Unfortunately, as mentioned above, formula node functionality is currently entirely undocumented. To make up for that, I've copy-pasted my own notes on how each of the various _formula functions_ work, exported directly from my Tana workspace:

The order here is preserved from the way they're defined in Tana's javascript; which seems to group related functions.

---

The following functions seem to exist on formula nodes, with _child nodes_ being their arguments. Functions are a node with that contains only their name as a string, any child nodes on the function node are taken as arguments.

- `top` - returns the first N elements from an array, such as returned by `childrenOf`
    - **first parameter:** N, the number of children to return
    - **second parameter**: the array to return the elements from
- `first` - alias for `top` ; returns the first N elements from an array
    - **first parameter:** N, the number of children to return
    - **second parameter:** the array to return the elements from
- `pickRandom`  - picks a random element from an array; re-computed on every evaluation
    - **first parameter**: the array to select from
- `bottom` - returns the last N elements from an array
    - **first parameter:** N, the number of children to return
    - **second parameter:** the array to return the elements from
- `last` - alias for `bottom` , returns the last N elements from an array
    - **first parameter:** N, the number of children to return
    - **second parameter:** the array to return the elements from
- `sum`  - summates all child values, _flattening all values first_
- `subtract`  - subtracts all values after the first from the first value, _flattening all values first_
- `count`  - counts the number of children, _flattening all values first_
- `max` - returns the highest child value, _flattening all values first_
- `min`  - returns the lowest child value, _flattening all values first_
- `average`  - returns the average of all children, _flattening all values first_
- `mean`  - alias for `average` 
- `round`  - rounds the only child argument to the nearest whole number
    - **first argument**: ****the number to round
- `+`  - alias for `sum` 
- `-`  - alias for `subtract` 
- `multiply` - multiplies all children; _accepts only a flat set of child nodes_
- `*`  - alias for multiply
- `divide`  - divides the first child by each of the remaining values, in order
- `/`  - alias for `divide` 
- `findInstance`  - finds all nodes of a given _type_ (e.g. supertag or base type) which exist either _above_ (parent-wards) or _below_ (child-wards) the given node
    - **first child:** a reference to the type we'd like to search for; e.g. a reference to a supertag
    - **second child:** sets the search direction; either the string ABOVE  or BELOW 
    - **third child:** the node to search with respect to; defaults to SELF 
- `findFieldValues`  - finds the field value for all fields either _above_ (parent-wards) or _below_ (child-wards) the given node
    - **first child:** a reference to the field we want to get the data for
    - **second child**: sets the search direction; either the string `ABOVE`  or `BELOW` 
    - **third child**: the node to search with respect to; defaults to `SELF` 
- `findInstances`  - finds all instances of a given item; i.e. all nodes that have a tag, or all instances of a given field
    - **first child:** a reference to the supertag (or base type) that we want to search for, _or_ a reference to the field we want to find instances of
- `lookup` - looks up the value of a given field on a provided node, _as a value string_
    - **first child:** a reference to the field we want to get the data for
    - **second child:** a reference to the node we want to look for fields on, or an array of such references; defaults to SELF 
- `lookupField` - looks up a _field_'s contents __given a reference to it; while `lookup` will return the field's value, `lookupField` will return a reference to those contents
    - **first child:** a reference to the field we want to get the data for
    - this can be gotten with CTRL+K, then _copy link to node_
    - this can also be an attr id, as given in the schema view, or a function that yields a reference
    - **second child:** a reference to the node we want to look for fields on, or an array of such references; defaults to _`SELF`_ 
- `filter`  - filters a list of nodes using a search query
    - **first argument**: an array of nodes to run the filter on
    - **second argument:** a search expression to apply to the given set of nodes, exactly as you would enter it into the search query builder, except as a single node
    - "except as a single node" here means that there's no automatic top-level _AND_, so you'll need to add that in yourself if you want to AND together multiple conditions
    - it's likely easiest to build the query as a single (e.g. AND) node there, and then move it over
- `childrenOf` - resolves references to all children of a given node
    - **first child:** a reference to the node we want to get the children of
    - this can be gotten with CTRL+K, then copy link to node
    - this can also be an attr id, as given in the schema view for API, or a function that yields a reference
- `nodesBelow`  - resolves references to all nodes _below_ a given node; i.e. finding the children of a given node, and their children, and so on recursively. maintains structure
    - first child: a reference to the node we want to get the children of
- `SELF` - yields a reference the _parent node_ for the field whose formula is being set
- `CREATED`  - returns the creation time for SELF
    - **first child:** format string, as in Tana's other date format strings
- `CURRENT_USER` - returns the login for the current user
- `CURRENT_USER_REF`  - returns a reference to the current user
- `CURRENT_DATE_REF`  - returns a reference to the current date; re-computed on every evaluation
- `CURRENT_DATETIME_REF` - returns a reference to the current time; re-computed on every evaluation
- `concat` - concatenates together the values from all provided arrays
- `strlen` - returns the number of characters in the child node
    - **first child:** the node to get the string length of; child nodes of this node are ignored
- `blockFilter` - inverted variant of `filter` ; returns all nodes for which a given search expression does _not_ match
    - **first argument:** an array of nodes to run the filter on
    - **second argument:** a search expression to apply to the given set of nodes, exactly as you would enter it into the search query builder, except as a single node
- `uniq`  - filters an array, returning unique array elements; note that this is done by reference, rather than by value
    - **first child:** the array to be filtered
- `backlink`  - find all nodes with a given field ("F") that references the given node ("N")
    - **first child, "F"**: a reference to the field which will be searched for references to the given node
    - **second child, "N":  **the node to be searched _for_; defaults to `SELF` 
- `ownerOf` - returns the _owner_ for the given node (every Tana node has a single _owner_, where it conceptually resides, even if it's referenced in multiple _parents)_
- `formatDate`  - formats a given date
    - **first child:** the date to format, which should be a parseable date string, rather than as a reference
    - **second child:** the format string, as in Tana's other date format strings
- `emojiCount`  - counts the number of emoji in a single node; _child nodes are ignored_
    - **first argument**: the node to count emoji from
