# panki

A project management tool for Anki deck builders.

## Contents

- [Installation]
- [Getting Started]
- [Converting an Anki Package]
- [Scaffolding]
  - [Project Scaffolding]
  - [Component Scaffolding]
- [Configuration]
  - [Project Configuration]
  - [Note Type Configuration]
  - [Card Type Configuration]
  - [Deck Configuration]
  - [Note Configuration]
- [Note Data]
- [Templates and Styling]
- [Working with Anki Collections]
  - [Dumping Package Files]
  - [Merging Package Files]
- [License]

## Installation

You can install panki with pip:
```sh
$ pip install panki
```

You should be able to use panki now:
```sh
$ panki --help
```

If you are starting a fresh project, please see the [Getting Started] section.
This section will walk you through setting up a basic project with panki. Panki
is very flexible - if the basic project structure is too simple for your needs,
please see the [`examples/`] directory for example project structures and the
[Configuration] section for more information about configuring your project.

If you would like to convert an existing deck or group of decks into a panki
project, please see the [Converting an Anki Package] section.

## Getting Started

First, let's get some terminology out of the way. In Anki, a **Deck** contains
one-to-many **Notes**. One-to-many **Cards** are created from each **Note**.
**Notes** are created from generic **Note Types**. **Cards** are created from
generic **Card Types**. **Note Types** contain one-to-many **Card Types** and
can be used across multiple **Decks**.

See the [Anki documentation (Key Concepts)] for more information about the
various objects within Anki and how they relate to each other.

In this guide, we'll be creating a very basic deck with a single note type and a
single card type. Please see the [`examples/`] directory for more complicated
project structures, including multi-note-type and multi-card-type projects, as
well as multi-deck projects. The project we'll be building in this section is
listed under [`examples/basic`] if you need to refer to it.

Let's create a deck for memorizing the symbols of all of the elements on the
periodic table of elements. You can use the `panki create project` command to
generate a new project:
```sh
$ panki create project periodic-table
```

The project will be created under the directory name passed to
`panki create project`, so `periodic-table/` in this case. Please make sure that
this directory does not already exist - for safety reasons, panki will not
create the project if the directory already exists.

After running `panki create project`, you should have a `periodic-table/`
directory with the following contents:
```
periodic-table
├── data.csv
├── project.json
└── template.html
```

The `project.json` file contains all of the configuration for this project.
Panki uses this file to understand how your project is set up and how you'd like
the project to be built. The project configuration can also be split up among
multiple files if this file becomes too unwieldy - see the [Configuration]
section for more information.

The default `project.json` should look similar to the following (your IDs will
be different):
```json
{
  "name": "New Project",
  "noteTypes": [
    {
      "id": 1234567890123,
      "name": "Basic (panki)",
      "fields": [
        "Front",
        "Back"
      ],
      "cardTypes": [
        {
          "name": "Card",
          "template": "template.html"
        }
      ]
    }
  ],
  "decks": [
    {
      "id": 1234567890124,
      "name": "New Deck",
      "package": "deck.apkg",
      "notes": [
        {
          "type": "Basic (panki)",
          "data": "data.csv"
        }
      ]
    }
  ]
}
```

The `data.csv` file will eventually contain the data used to generate the deck's
notes. The default `data.csv` file is empty. A CSV file is generated by default,
but panki supports multiple data formats. If you would like to separate your
data across multiple files, that is also possible. See the [Note Data] section
for more information.

The `template.html` file is the HTML template for a card type. Everything within
the `<front>` tag will be used for the front (or question) side of the card.
Likewise, everything within the `<back>` tag will be used for the back (or
answer) side of the card. You can customize the styling of the card by adding
custom CSS to the `<style>` tag of this template. If you would like to apply
common styling to more than just one card type, that is also possible. See the
[Templates and Styling] section for more information.

The default `template.html` file is very similar to the Anki "Basic" template:
```html
<template>
  <style>
    .card {
      font-family: arial;
      font-size: 20px;
      text-align: center;
      color: black;
      background-color: white;
    }
  </style>
  <front>
    {{Front}}
  </front>
  <back>
    {{FrontSide}}
    <hr id="answer">
    {{Back}}
  </back>
</template>
```

The default note type contains two fields: `Front` and `Back`. If you look in
`template.html`, you'll see placeholders for `{{Front}}` and `{{Back}}`
respectively.

For this project, we will create a custom note type instead of using the default
note type. Our note type will use the field `Element` for the element name
(ex. Oxygen) and `Symbol` for the element symbol (ex. O). Let's open
`project.json` and modify the note type accordingly. We'll also update the
project name and deck name to better describe this project.

Your `project.json` file should look something like this after these changes:
```json
{
  "name": "Periodic Table of Elements",
  "noteTypes": [
    {
      "id": 1234567890123,
      "name": "Element Symbol",
      "fields": ["Element", "Symbol"],
      "cardTypes": [
        {
          "name": "Element Symbol",
          "template": "template.html"
        }
      ]
    }
  ],
  "decks": [
    {
      "id": 1234567890124,
      "name": "Element Symbols",
      "package": "deck.apkg",
      "notes": [
        {
          "type": "Element Symbol",
          "data": "data.csv"
        }
      ]
    }
  ]
}
```

Now let's update our `template.html` file to incorporate our new fields. We'll
even add some custom CSS to make the element symbol a bit more fancy.
```html
<template>
  <style>
    .card {
      font-family: Arial;
      font-size: 20px;
      text-align: center;
      color: black;
      background-color: white;
    }
    .symbol {
      font-size: 1.5em;
      border: 2px solid black;
      padding: 10px;
    }
  </style>
  <front>
    What is the symbol for <b>{{Element}}</b>?
  </front>
  <back>
    {{FrontSide}}
    <hr id="answer">
    <div class="symbol">
      {{Symbol}}
    </div>
  </back>
</template>
```

Finally, let's add our data. You can copy the contents of
[`examples/basic/data.csv`] into your `data.csv` file or add your own data.
Make sure that the header (first row) of `data.csv` contains the field names
(ex. `Element,Symbol`) and that your data is in the same order (ex. `Oxygen,O`).

We should now have everything we need to generate our deck. You can generate the
deck by running the `panki build` command from the project's root directory:
```sh
$ panki build
```

By default, the deck will be created at `deck.apkg` in the root directory of
the project. You can change the name of the file and its location by modifying
the `"package"` value in the `"deck"` section of the `project.json` file.

After building the project, you should be able to import the generated
`deck.apkg` file into Anki using the `Import File` button or `File` > `Menu`
from the main menu. **Warning: Please make sure you always backup your
collection (`File` > `Export`) before importing decks into Anki, especially if
you're updating an existing deck. If the import fails, or worse corrupts your
Anki database, then at least you will be able to restore your decks and progress
from the backup.**

Please refer to the rest of this readme for more advanced ways to use panki in
your deck building workflow.

## Converting an Anki Package

---

This functionality is not yet supported, but will be added in the future.

---

If you have an existing deck or group of decks in Anki that you'd like to
convert to a panki project, you can export them (`File` > `Export`) to an
`.apkg` file and convert them to a panki project using the `panki convert`
command.

For example, if you have one or more existing decks for memorizing information
from the periodic table of elements, you can export the decks from Anki to a
file called `periodic-table.apkg` and convert them to a panki project with the
following command:
```py
$ panki convert periodic-table.apkg periodic-table
```

This command will read `periodic-table.apkg` and create a panki project in the
`periodic-table/` directory within the current directory. Please make sure that
the `periodic-table/` directory does not already exist - for safety reasons,
panki will not create the project if the directory already exists.

Please refer to the rest of this readme for more information about managing your
project with panki.

## Scaffolding

You can scaffold out new projects and new project components using the
`panki create` command.

It is not required to use the scaffolding tool, of course - you could just
create all of the files and directories yourself if you know what you're doing -
but using the tool has several advantages. The scaffolding tool will ensure that
project files are created correctly. Commands to generate common project
structures can easily be shared with others. You can also build upon the
scaffolding commands by using them in your own tools and scripts.

### Project Scaffolding

To create a new project, execute the `panki create project` command:
```sh
$ panki create project <directory>
```

Replace `<directory>` with the name of the new project directory. Make sure that
this directory does not exist before running this command. Panki will not create
the project if the directory already exists.

You can set the name of the project by passing the `--name` option:
```sh
$ panki create project periodic-table --name "Periodic Table of Elements"
```

If you would like to package all of the decks in the project as a single `.apkg`
file, you can pass the `--package` option.
```sh
$ panki create project periodic-table \
  --name "Periodic Table of Elements" \
  --package @/packages/periodic-table.apkg
```

#### Note Type Options

You can set the name of a note type by passing the `--note-type` option:
```sh
$ panki create project periodic-table --note-type "Element Symbol"
```

You can create multiple note types by passing multiple `--note-type` options:
```sh
$ panki create project periodic-table \
    --note-type "Element Symbol" \
    --note-type "Element Atomic Number" \
    --note-type "Element Group" \
    --note-type "Element Block"
```

You can pass the following additional options to further customize a note type:
  - **`--note-type-config <note-type> <path>`**: Place the configuration for the
    note type with name `<note-type>` outside of `project.json` in an external
    file located at `<path>`.
  - **`--fields <note-type> <fields>`**: Set the fields of the note type with
    name `<note-type>` to the comma-separated list of `<fields>`
    (ex. `Element,Symbol`).
  - **`--css <note-type> <paths>`**: Set the css files of the note type with
    name `<note-type>` to the comma separated list of `<paths>`
    (ex. `common.css,symbol.css`)
  - **`--card-type <note-type> <name> <template>`**: Add a card type to the note
    type with name `<note-type>` with the provided card type `<name>` and
    `<template>` path. A minimal template file will be created at `<template>`.

If no note types are provided, a "Basic (panki)" default note type is created.

#### Deck Options

You can set the name of a deck by passing the `--deck` option:
```sh
$ panki create project periodic-table --deck "Element Symbols"
```

This will also set the deck's `package` field to a stripped-down version of the
deck name. You can change this value by providing the `--deck-package` option
(see below).

You can create multiple decks by passing multiple `--deck` options:
```sh
$ panki create project periodic-table \
    --deck "Element Symbols" \
    --deck "Element Atomic Numbers" \
    --deck "Element Groups" \
    --deck "Element Blocks"
```

You can pass the following additional options to further customize the decks:
  - **`--deck-config <deck> <path>`**: Place the deck configuration for the deck
    with name `<deck>` outside of `project.json` in an external file at
    `<path>`.
  - **`--deck-package <deck> <path>`**: Set the package path of the deck with
    name `<deck>` to `<path>`. Make sure the path ends with `.apkg`.
  - **`--notes <deck> <note-type> <data-files>`**: Add a group of notes to the
    deck with name `<deck>` which are of type `<note-type>` and are created from
    the data in the provided `<data-files>` list of paths. If you would like to
    specify multiple data files, use a comma-separated list
    (ex. `"data1.csv,data2.csv,data3.csv"`). Empty data files will be created at
    the provided paths.

If no deck configuration is provided, an empty default deck is created.

#### Examples

The following command will scaffold out a project structure in which each deck
is separated out into its own subdirectory. All of the decks are stored under a
`decks/` directory, and each deck contains a `deck.json` file with the deck's
configuration. Note data is also stored in the deck directory. All of the note
types are stored under a `note-types/` directory, and each note type contains a
`note-type.json` file with the note type's configuration. Card type templates
are also stored in the note type directory.
```sh
$ panki create project periodic-table \
    \
    --name "Periodic Table of Elements" \
    --package @/packages/periodic-table.apkg \
    \
    --note-type-config "Element Symbol" note-types/symbol/note-type.json \
    --fields "Element Symbol" Element,Symbol \
    --css "Element Symbol" @/note-types/common.css,@/note-types/symbol.css \
    --card-type "Element Symbol" Forward template.forward.html \
    --card-type "Element Symbol" Reverse template.reverse.html \
    \
    --note-type-config "Element Atomic Number" note-types/atomic-number/note-type.json \
    --fields "Element Atomic Number" Element,AtomicNumber \
    --css "Element Atomic Number" @/note-types/common.css \
    --card-type "Element Atomic Number" Forward template.forward.html \
    --card-type "Element Atomic Number" Reverse template.reverse.html \
    \
    --note-type-config "Element Year Discovered" note-types/year-discovered/note-type.json \
    --fields "Element Year Discovered" Element,YearDiscovered \
    --css "Element Year Discovered" @/note-types/common.css \
    --card-type "Element Year Discovered" Forward template.forward.html \
    --card-type "Element Year Discovered" Reverse template.reverse.html \
    \
    --deck-config "Element Symbols" decks/symbols/deck.json \
    --deck-package "Element Symbols" @/packages/symbols.apkg \
    --notes "Element Symbols" "Element Symbol" symbols.csv \
    \
    --deck-config "Element Atomic Numbers" decks/atomic-numbers/deck.json \
    --deck-package "Element Atomic Numbers" @/packages/atomic-numbers.apkg \
    --notes "Element Atomic Numbers" "Element Atomic Number" atomic-numbers.csv \
    \
    --deck-config "Element Years Discovered" decks/years-discovered/deck.json \
    --deck-package "Element Years Discovered" @/packages/years-discovered.apkg \
    --notes "Element Years Discovered" "Element Year Discovered" years-discovered.csv
```

The above command would create the following project structure:
```
periodic-table/
├── decks
│   ├── atomic-numbers
│   │   ├── atomic-numbers.csv
│   │   └── deck.json
│   ├── symbols
│   │   ├── deck.json
│   │   └── symbols.csv
│   └── years-discovered
│       ├── deck.json
│       └── years-discovered.csv
├── note-types
│   ├── atomic-number
│   │   ├── note-type.json
│   │   ├── template.forward.html
│   │   └── template.reverse.html
│   ├── symbol
│   │   ├── note-type.json
│   │   ├── template.forward.html
│   │   └── template.reverse.html
│   └── year-discovered
│       ├── note-type.json
│       ├── template.forward.html
│       └── template.reverse.html
└── project.json
```

The following command will scaffold out an alternative project structure in
which all of the project configuration is stored in `project.json` and a `data/`
directory, `templates/` directory, and `styles/` directory are used instead:
```sh
panki create project periodic-table \
    \
    --name "Periodic Table of Elements" \
    --package packages/periodic-table.apkg \
    \
    --fields "Element Symbol" Element,Symbol \
    --css "Element Symbol" styles/common.css,styles/symbol.css \
    --card-type "Element Symbol" Forward templates/symbol.forward.html \
    --card-type "Element Symbol" Reverse templates/symbol.reverse.html \
    \
    --fields "Element Atomic Number" Element,AtomicNumber \
    --css "Element Atomic Number" styles/common.css \
    --card-type "Element Atomic Number" Forward templates/atomic-number.forward.html \
    --card-type "Element Atomic Number" Reverse templates/atomic-number.reverse.html \
    \
    --fields "Element Year Discovered" Element,YearDiscovered \
    --css "Element Year Discovered" styles/common.css \
    --card-type "Element Year Discovered" Forward templates/year-discovered.forward.html \
    --card-type "Element Year Discovered" Reverse templates/year-discovered.reverse.html \
    \
    --deck-package "Element Symbols" packages/symbols.apkg \
    --notes "Element Symbols" "Element Symbol" data/symbols.csv \
    \
    --deck-package "Element Atomic Numbers" packages/atomic-numbers.apkg \
    --notes "Element Atomic Numbers" "Element Atomic Number" data/atomic-numbers.csv \
    \
    --deck-package "Element Years Discovered" packages/years-discovered.apkg \
    --notes "Element Years Discovered" "Element Year Discovered" data/years-discovered.csv
```

The above command would create the following project structure:
```
periodic-table/
├── data
│   ├── atomic-numbers.csv
│   ├── symbols.csv
│   └── years-discovered.csv
├── project.json
└── templates
    ├── atomic-number.forward.html
    ├── atomic-number.reverse.html
    ├── symbol.forward.html
    ├── symbol.reverse.html
    ├── year-discovered.forward.html
    └── year-discovered.reverse.html
```

### Component Scaffolding

...

## Configuration

The `project.json` file contains configuration for a panki project. It provides
panki with information about the project and the structure of its files. Sample
files and more information about each configuration item are provided below.

Paths in configuration files are relative to the configuration file that the
path is in. If the path is prefixed with `@/` (ex. `@/packages/deck.apkg`), then
the path will always be relative to the project's root directory, regardless of
the directory that the configuration is in.

### Project Configuration

The `project.json` file from the [Getting Started] section is provided below.
This file defines a minimal panki project. All of the configuration is in a
single file, but these items can be split out into multiples files.
```json
{
  "name": "Periodic Table of Elements",
  "noteTypes": [
    {
      "id": 1234567890123,
      "name": "Element Symbol",
      "fields": ["Element", "Symbol"],
      "cardTypes": [
        {
          "name": "Element Symbol",
          "template": "template.html"
        }
      ]
    }
  ],
  "decks": [
    {
      "id": 1234567890124,
      "name": "Element Symbols",
      "package": "deck.apkg",
      "notes": [
        {
          "type": "Element Symbol",
          "data": "data.csv"
        }
      ]
    }
  ]
}
```

The `name` field is the name of the project.

The `noteTypes` field contains a list of note type configurations for each note
type in the project. See [Note Type Configuration] for more information.

The `decks` field contains a list of deck configurations for each deck in the
project. See [Deck Configuration] for more information.

The optional `package` field can also be provided. If provided, all of the decks
in the project will be combined into a single `.apkg` file. Set the value of
this field to the path where the `.apkg` file should be created. You can also
use the `panki merge` command to merge `.apkg` files together - See the
[Merging Package Files] section for more information.

### Note Type Configuration

Note type configuration can be split out from your project configuration into a
separate file by specifying the path to a note type configuration file:
```json
{
  "name": "Periodic Table of Elements",
  ...
  "noteTypes": [
    "note-type.json"
  ]
}
```

In this case, we've specified that the note type configuration is located in a
file called `note-type.json`, but the name of this file and its location are up
to you.

A common pattern for projects with multiple note types is to create a directory
for each note type and place a `note-type.json` file in each of those
directories. The project configuration for such a project might look like the
following:
```json
{
  "name": "Periodic Table of Elements",
  ...
  "noteTypes": [
    "note-types/symbol/note-type.json",
    "note-types/atomic-number/note-type.json",
    "note-types/year-discovered/note-type.json"
  ]
}
```

The `note-type.json` file would then contain just your note type configuration:
```json
{
  "id": 1234567890123,
  "name": "Element Symbol",
  "fields": ["Element", "Symbol"],
  "cardTypes": [
    {
      "name": "Element Symbol",
      "template": "template.html"
    }
  ]
}
```

The `id` field is automatically generated by panki. This value is used to
uniquely identify the note type within a user's Anki app and (as best as
possible) among all note types in existence. Once this value is set, it should
never be changed, otherwise there will be problems importing updated version of
the note type into Anki.

If you need to generate an ID, you can use the following command:
```sh
$ panki create id
```

The `name` field is the name of the note type as it will appear in Anki. This
name will also be used to link notes in note configuration to this note type.
If you change the name of a note type, be sure to update it in the rest of your
configuration. See [Note Configuration] for more information.

The `fields` field contains a list of the field names used by this note type.
These should match the field names in the respective note data files. Only the
fields specified in `fields` will be selected from your note data files - all
other fields will be ignored.

The `cardTypes` field contains a list of configurations for each card type in
the note type. See [Card Type Configuration] for more information.

### Card Type Configuration

Card type configuration can be split out from your note type configuration into
a separate file by specifying the path to a card type configuration file:
```json
{
  "name": "Element Symbol",
  "fields": ["Element", "Symbol"],
  "cardTypes": [
    "card-type.json"
  ]
}
```

In this case, we've specified that the card type configuration is located in a
file called `card-type.json`, but the name of this file and its location are up
to you.

Separating out card type configuration into a separate file is usually overkill,
even for large projects, but this functionality is available if required.

The `card-type.json` file would then contain just your card type configuration:
```json
{
  "name": "Element Symbol",
  "template": "template.html"
}
```

The `name` field is the name of the card type as it will appear in Anki.

The `template` field is the path to the HTML template file for the card type.
See the [Templates and Styling] section for more information.

### Deck Configuration

Deck configuration can be split out from your project configuration into a
separate file by specifying the path to a deck configuration file:
```json
{
  "name": "Periodic Table of Elements",
  "decks": [
    "deck.json"
  ],
  ...
}
```

In this case, we've specified that the deck configuration is located in a file
called `deck.json`, but the name of this file and its location are up to you.

A common pattern for multi-deck projects is to create a directory for each deck
and place a `deck.json` file in each directory. The project configuration for
such a project might look like the following:
```json
{
  "name": "Periodic Table of Elements",
  "decks": [
    "decks/symbols/deck.json",
    "decks/atomic-numbers/deck.json",
    "decks/years-discovered/deck.json"
  ],
  ...
}
```

The `deck.json` file would then contain just your deck configuration:
```json
{
  "id": 1234567890124,
  "name": "Element Symbols",
  "package": "deck.apkg",
  "notes": [
    {
      "type": "Element Symbol",
      "data": "data.csv"
    }
  ]
}
```

The `id` field is automatically generated by panki. This value is used to
uniquely identify the deck within a user's Anki app and (as best as possible)
among all decks in existence. Once this value is set, it should never be
changed, otherwise there will be problems importing updated version of the deck
into Anki.

If you need to generate an ID, you can use the following command:
```sh
$ panki create id
```

The `name` field is the name of the deck as it will appear in Anki. Remember
that you can create a deck tree by naming your decks with a `::` between the
parent deck name and the subdeck names.

For example, all of the following subdecks would be nested under the
`Periodic Table` parent deck in Anki:
  - `Periodic Table :: Symbols`
  - `Periodic Table :: Atomic Numbers`
  - `Periodic Table :: Groups`
  - `Periodic Table :: Blocks`

The `package` field is the path of the file that will be generated when the
project is built with `panki build`. This file should end with `.apkg`.

The `notes` field contains a list of note configurations for each group of notes
in the deck. See the [Note Configuration] section for more information.

### Note Configuration

Note configuration can be split out from your deck configuration into a separate
file by specifying the path to a note configuration file:
```json
{
  "id": 1234567890124,
  "name": "Element Symbols",
  "package": "deck.apkg",
  "notes": [
    "note.json"
  ]
}
```

In this case, we've specified that the note configuration is located in a file
called `note.json`, but the name of this file and its location are up to you.

Separating out note configuration into a separate file is usually overkill, even
for large projects, but this functionality is available if required.

The `note.json` file would then contain just your note configuration:
```json
{
  "type": "Element Symbol",
  "data": "data.csv"
}
```

The `type` field is the name of the note type that the notes in this group will
be created from. Make sure you remember to change this value if the note type's
name ever changes, otherwise the notes will not be linked correctly.

The `data` field is the path to a note data file or a list of paths to several
note data files. Notes will be created and added to the deck in the order
specified in this field. See the [Note Data] section for more information.

The optional `guid` field can be specified to control the format of the note
GUID that will be created and assigned to the note. This field is often provided
if the default method of generating the GUID is not sufficient to guarantee the
uniqueness of a note. Note GUIDs prevent duplicate notes from being created in
certain cases and ensure that the correct notes are updated when importing
package files. They should be unique across all notes.

By default, the note GUID is created by Base64-encoding the deck ID, the note
type ID, and the value of the first field specified in the note type
configuration's `fields` field. The GUID format should be specified in the
[python string format syntax]. All of the fields in the record are available.
The deck ID and the note type ID are also available from the special
`__DeckName__` and `__NoteTypeID__` fields, respectively.

For example, to specify the GUID format as a combination of the deck ID, note
type ID, `Element` field value, and `Symbol` field value:
```json
{
  "type": "Element Symbol",
  "guid": "{__DeckID__}:{__NoteTypeID__}:{Element}:{Symbol}",
  "data": "data.csv"
}
```

The value created from this format will be Base64-encoded to create the final
note GUID.

## Note Data

Note data can be provided in one or many CSV, JSON, and/or YAML files. The order
of the cards in the generated Anki deck will correspond to the order of the data
in your data files.

### CSV Data Files

For CSV files, the header (first row) of the file should contain the field
names. These fields will be read from each row in the order specified, so make
sure that your data is in the same order as your field names.

Example:
```csv
Element,Symbol
Hydrogen,H
Helium,He
Lithium,Li
Beryllium,Be
Boron,B
Carbon,C
Nitrogen,N
Oxygen,O
Fluorine,F
Neon,Ne
...
```

### JSON/YAML Data Files

For JSON and YAML files, the contents of the file should be a list and each
item in the list should be an object. Each of these objects should have keys
corresponding to the note field names.

JSON Example:
```json
[
  {"Element": "Hydrogen", "Symbol": "H"},
  {"Element": "Helium", "Symbol": "He"},
  {"Element": "Lithium", "Symbol": "Li"},
  {"Element": "Beryllium", "Symbol": "Be"},
  {"Element": "Boron", "Symbol": "B"},
  {"Element": "Carbon", "Symbol": "C"},
  {"Element": "Nitrogen", "Symbol": "N"},
  {"Element": "Oxygen", "Symbol": "O"},
  {"Element": "Fluorine", "Symbol": "F"},
  {"Element": "Neon", "Symbol": "Ne"},
  ...
]
```

YAML Example:
```yaml
# Elements in period 1:
- Element: Hydrogen
  Symbol: H
- Element: Helium
  Symbol: He

# Elements in period 2:
- Element: Lithium
  Symbol: Li
- Element: Beryllium
  Symbol: Be
- Element: Boron
  Symbol: B
- Element: Carbon
  Symbol: C
- Element: Nitrogen
  Symbol: N
- Element: Oxygen
  Symbol: O
- Element: Fluorine
  Symbol: F
- Element: Neon
  Symbol: Ne

# ...
```

An advantage of using YAML for your note data is that you can include comments
within your data file. A disadvantage of the YAML and JSON formats is that they
are not as compact as CSV files, but this may be an acceptable tradeoff,
depending on how you manage your data. The deck generated from the data will be
the same size, regardless of the format of your data files.

### Multiple Data Files

You can choose to organize your data across multiple data files if this makes
sense for organizational purposes. You may also wish to segment your data in
order to better control the ordering of the cards in your deck, especially when
your cards are generated from multiple note types.

Sticking with the periodic table of elements theme, you may wish to segment
elements by period and store all of your data files in a `data/` directory. In
this case, your note configuration might look like the following:
```json
{
  "type": "Element Symbol",
  "data": [
    "data/period1.csv",
    "data/period2.json",
    "data/period3.yaml",
    ...
  ],
  ...
}
```

If you have multiple decks in a project, they usually have their own data files,
but there's nothing stopping you from using the same data file for each deck or
sharing a core set of data files between several decks if this makes sense for
your project. If you are using a data file multiple times within the same deck,
you will likely need to set the `guid` field in the deck's note configuration to
another pattern in order to avoid creating duplicate cards. See the
[Note Configuration] section for more information.

Remember that you can use other tools and scripts alongside panki. If your data
requires extra processing or a conversion process, you can perform that step
just prior to generating a deck.

## Templates and Styling

Templates define the structure and style of a card type. Note types contain
one-to-many card types from which cards are generated. Note types can be shared
across multiple decks.

The default panki template looks like this:
```html
<template>
  <style>
    .card {
      font-family: arial;
      font-size: 20px;
      text-align: center;
      color: black;
      background-color: white;
    }
  </style>
  <front>
    {{Front}}
  </front>
  <back>
    {{FrontSide}}
    <hr id="answer">
    {{Back}}
  </back>
</template>
```

Templates must contain a root element, `<template>`, and a `<front>` and
`<back>` element nested under this root element. You can also provide the
optional `<style>` element under the root element.

In Anki, CSS is assigned to a note type, so the CSS applies to all card types
defined in the note type. The CSS provided in the `<style>` tag of each card
type template will be merged together in the order specified in the note type
configuration in order to create the final CSS for the note type.

If there is common CSS that you'd like to place outside of your individual
templates, you can provide the optional `"css"` field in your note type
configuration with a path or list of paths to external `.css` files. The CSS in
these files will be merged together in the order specified and then the CSS from
the `<style>` elements of the card type templates will be appended to create the
final CSS for the note type.

For example, if you store your common CSS in a `common.css` file, your note type
configuration might look something like this:
```json
{
  "id": 1234567890123,
  "name": "Element Symbol",
  "fields": ["Element", "Symbol"],
  "css": "common.css",
  "cardTypes": [
    {
      "name": "Element Symbol",
      "template": "template.forward.html"
    },
    {
      "name": "Element Symbol (Reverse)",
      "template": "template.reverse.html"
    }
  ]
}
```

You can also specify multiple stylesheets and refer to the project root
directory with the `@/` path prefix:
```json
{
  "id": 1234567890123,
  "name": "Element Symbol",
  "fields": ["Element", "Symbol"],
  "css": [
    "@/styles/common.css",
    "@/styles/symbol.css"
  ],
  "cardTypes": [
    {
      "name": "Element Symbol",
      "template": "template.forward.html"
    },
    {
      "name": "Element Symbol (Reverse)",
      "template": "template.reverse.html"
    }
  ]
}
```

See the [Anki documentation (Card Templates)] for more information about
templates and styling.

## Working with Anki Collections

panki provides a few extra commands for working with Anki collections directly.
You might use these commands to take a closer look at the contents of a package
file, as part a more complicated build process, or to make changes to an Anki
package manually whether it be to fix something or for some other reason.

### Dumping Package Files

It can be very useful to dump the contents of an Anki `.apkg` or `.colpkg` file
into a more convenient format for inspection or for further processing. Panki
provides the ability to dump the contents of a package with the `panki dump`
command:
```sh
$ panki dump <package> <directory>
```

Replace `<package>` with the path to the `.apkg` or `.colpkg` file that you'd
like to dump.

Replace `<directory>` with the path to the directory that you'd like panki to
dump the contents of the package into. This directory should not already exist.

After running `panki dump`, the directory will contain a `collection.anki2` file
that you can connect to with SQLite3:
```sh
$ sqlite3 path/to/collection.anki2
```

A `tables/` directory will also be created, containing a folder for each
database table in the `collection.anki2` file. In each folder, panki will create
a `table.json` file with metadata about the table, a `schema.sql` file with the
table schema in the form of a SQL `CREATE` command, and a `rows.csv` file with
the rows of the table in CSV format, if applicable.

### Merging Package Files

...

### Packaging Anki Collections

...

## License

This project is licensed under the **MIT License**. Please see the `LICENSE`
file for more information.

This project is not affiliated with the [Anki](https://apps.ankiweb.net/)
project.


<!-- links: -->

[Installation]: #installation
[Getting Started]: #getting-started
[Converting an Anki Package]: #converting-an-anki-package

[Scaffolding]: #scaffolding
[Project Scaffolding]: #project-scaffolding
[Component Scaffolding]: #component-scaffolding

[Configuration]: #configuration
[Project Configuration]: #project-configuration
[Note Type Configuration]: #note-type-configuration
[Card Type Configuration]: #card-type-configuration
[Deck Configuration]: #deck-configuration
[Note Configuration]: #note-configuration

[Note Data]: #note-data

[Templates and Styling]: #templates-and-styling

[Working with Anki Collections]: #working-with-anki-collections
[Dumping Package Files]: #dumping-anki-packages
[Merging Package Files]: #merging-anki-packages
[Packaging Anki Collections]: #packaging-anki-collections

[License]: #license

[`examples/`]: examples
[`examples/basic`]: examples/basic
[`examples/basic/data.csv`]: examples/basic/data.csv

[python string format syntax]: https://docs.python.org/3/library/string.html#format-string-syntax

[Anki documentation (Key Concepts)]: https://docs.ankiweb.net/#/getting-started?id=key-concepts
[Anki documentation (Card Templates)]: https://docs.ankiweb.net/#/templates/intro