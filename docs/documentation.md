# Documentation

*Following documentation taken from [mkanki](https://github.com/nornagon/mkanki)*

First, some information about how Anki works.

In Anki, things to be remembered are called _notes_. Each note can have several
fields—most commonly, a "Front" and a "Back", but the fields can be arbitrary.
Each note can potentially correspond to many individual _cards_, each of which
should help you remember one facet of the note.

The thing that describes how those fields are turned into flashcards is called
a _model_. Every note is described by exactly one model. The model defines
which fields are allowed, and additionally defines one or more _templates_,
which are written as HTML with [mustache](https://mustache.github.io/)-like
placeholders.

Finally, each card belongs to a _deck_. Decks collect cards into logical groups
that you might want to study separately from each other.

_Models_, _notes_, _cards_ and _decks_ are the fundamental concepts of Anki. In
mkanki, cards are implicitly defined by notes and models, and you will only
deal with models, notes, and decks.

##
### `Model`

Anki supports two types of models: _standard_ and _cloze_. Standard models
generate one card per template, while cloze models generate one card per cloze
deletion. See the [Anki cloze documentation][anki-cloze-docs] for more on cloze
deletion.

#### `new anki.Model(props)`

Create a new standard model with the given properties. `props` is an object
with the following fields.

- `id` string - a stable, unique identifier for this model. Generate this once
  with `+new Date` and then hard-code it into your code. Keeping this stable
  means that if the package is updated and re-imported into Anki, the app will
  be able to tell which cards are new and which cards should be merged into
  already-existing cards, preserving study history.
- `name` string - the name of the model. Shows up in the "Add" UI in Anki.
- `flds` Array&lt;{name: string}&gt; - the fields in the model.
- `tmpls` Array&lt;{name?: string, qfmt: string, afmt: string}&gt; - a list of
  card templates to be generated from each note. `qfmt` is the HTML template
  for the question, and `afmt` is the HTML template for the answer. `name` is
  displayed in the configuration screen in Anki and nowhere else, and will
  default to "Card N". See the [Anki template
  documentation][anki-template-docs] for more on template formatting.
- `req` Array&lt;[number, "all" | "any", Array&lt;number&gt;]&gt; - this
  describes which fields must be non-empty in order for a card to be generated.
  Each entry in this list is a tuple of the template index, "all" or "any", and
  a list of field indices. In order for a card to be generated for a given note
  and template, one or all of the fields specified in the field list must be
  non-empty. If the requirement isn't met for a given (template, note) pair, no
  card will be generated.

#### `new anki.ClozeModel(props)`

Create a new cloze model with the given properties. `props` is an object with
the following fields.

- `id` string - a stable, unique identifier for this model. Generate this once
  with `+new Date` and then hard-code it into your code. Keeping this stable
  means that if the package is updated and re-imported into Anki, the app will
  be able to tell which cards are new and which cards should be merged into
  already-existing cards, preserving study history.
- `name` string - the name of the model. Shows up in the "Add" UI in Anki.
- `flds` Array&lt;{name: string}&gt; - the fields in the model.
- `tmpl` {name?: string, qfmt: string, afmt: string} - the cloze template to
  be generated from each note. `qfmt` is the HTML template for the question,
  and `afmt` is the HTML template for the answer. `name` is displayed in the
  configuration screen in Anki and nowhere else, and will default to "Cloze".
  See the [Anki template documentation][anki-cloze-template-docs] for more on
  cloze template formatting. Cloze models can only have one template.

#### `model.note(fieldValues, [tags], [guid])`

Create a note using this model.

- `fieldValues` Array&lt;string&gt; | {[fieldName: string]: string} - If
  `fieldValues` is an array, the order of fields will be matched with the order
  of the `flds` in the model. If `fieldValues` is an object, the keys must be
  the names of fields in the model.
- `tags` is an array, the array will be joined with space separated values in `Package`
- `guid` string _(optional)_ - a stable, unique identifier for this note. When
  re-importing an updated version of this note, Anki will replace notes with
  matching identifiers. Defaults to a hash of the field values.

### `Deck`

In mkanki, decks are collections of notes (not cards, as in Anki proper).

#### `new anki.Deck(id, name)`

Create a new deck.

- `id` string - a stable, unique identifier for this deck. Generate this once
  with `+new Date` and then hard-code it into your code. Keeping this stable
  means that if the package is updated and re-imported into Anki, the app will
  be able to tell which cards are new and which cards should be merged into
  already-existing cards, preserving study history.
- `name` string - the name of the deck. When importing, Anki will create new
  decks with the specified names for each deck in the package.

#### `deck.addNote(note)`

Add a note to this deck. Technically, it is possible for a single note in Anki
to generate cards belonging to multiple decks, but mkanki does not support
that.

- `note` Note - create notes using [`model.note()`](#modelnotefieldvalues).

### `Package`

A package collects together decks, notes, and any media objects (images, audio,
video, etc.) to be exported into a `.apkg` file.

#### `new anki.Package()`

Create a new empty package.

#### `package.addDeck(deck)`

Add a deck to this package.

- `deck` [Deck](#deck) - the deck to add.

#### `package.addMedia(data, name)`

Add a media file to this package.

- `data` string | Buffer - the contents of the media file.
- `name` string - the name of the file in the package.

#### `package.addMediaFile(filename, [name])`

Add a media file from the filesystem to this package.

- `filename` string - path to the file.
- `name` string _(optional)_ - the name of the file in the package. Defaults to
  `filename`.

#### `package.writeToFile(filename)`

Serializes the package to a file.

- `filename` string - path to the exported package. Conventionally ends in
  `".apkg"`.


[anki-template-docs]: https://apps.ankiweb.net/docs/manual.html#cards-and-templates
[anki-cloze-docs]: https://apps.ankiweb.net/docs/manual.html#cloze-deletion
[anki-cloze-template-docs]: https://apps.ankiweb.net/docs/manual.html#cloze-templates
