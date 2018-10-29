# Wiktextract

Wiktionary dump file parser and multilingual data extractor

## Overview

Thois is a Python3 package for extracting information from Wiktionary
data dumps.  It reads the
``enwiktionary-<date>-pages-articles.xml.bz2`` file (or corresponding
files from other wiktionaries) and returns Python dictionaries
containing most of the information from Wiktionary.

The library is capable of extracting information for any language.
However, so far it has mostly been tested with English and Finnish,
and to some extent German and Spanish.  Changes to extract information
for any additional languages are likely to be small.  Basic
information extraction most likely works out of the box for any
language.

The library extracts glosses, parts-of-speech, declension/conjugation
information where available, translations for all languages when
available, pronunciations (including audio file links), qualifiers
including usage notes, word forms, links between words including
hypernyms, hyponyms, holonyms, meronyms, related words, derived terms,
compounds, alternative forms, etc.  Links to Wikipedia pages, Wikidata
identifiers, and other such data are also extracted when available.
For many classes of words, a word sense is annotated with specific
information such as what word it is a form of, what is the RGB value
of the color it represents, what is the numeric value of a number,
what SI unit it represents, etc.

This library will be useful for many natural language processing,
semantic parsing, machine translation, and language generation
applications both in research and industry.

The library can be easily used to extract machine translation
dictionaries, language understanding dictionaries, semantically
annotated dictionaries, and morphological dictionaries with
declension/conjugation information (where this information is
available for the target language).  Dozens of languages have
extensive vocabulary in ``enwiktionary``, and several thousand
languages have partial coverage.

The ``wiktwords`` script makes extracting the information for use by
other tools trivial without writing a single line of code.  It
extracts the information specified by command options for languages
specified on the command line, and writes the extracted data to a file
or standard output in JSON format for processing by other tools.

As far as we know, this is the most comprehensive tool available for
extracting information from Wiktionary as of November 2018.

## Installing

To install ``wiktextract``, use:

```
git clone https://github.com/tatuylonen/wiktextract.git
cd wiktextract
python3 setup.py install
```

This will install the ``wiktextract`` module and the ``wiktwords`` script.

Note that this software has currently only been tested with Python3.

There should be no fundamental reason why back-porting to Python 2.7
would be difficult; it just hasn't been tested yet.  Please report
back if you test and make this work with Python2.

## Using the Command-Line Tool

The ``wiktwords`` script is the easiest way to extract data from
Wiktionary.  Just download the data dump file from
[dumps.wikimedia.org](https://dumps.wikimedia.org/enwiktionary/) and
run the script.  The correct dump file has a name something like
``enwiktionary-<date>-pages-articles.xml.bz2``.

The simplest command line is:

```
wiktwords data/enwiktionary-latest-pages-articles.xml.bz2 --out wikt.words --language English
```

You may want to add command line options:

* --out FILE: specifies the name of the file to write (specifying "-" as the file writes to stdout)
* --language LANGUAGE: extracts the given language (this option may be specified multiple times; by default, English and Translingual words are extracted)
* --list-languages: prints a list of supported language names
* --translations: causes translations to be captured
* --pronunciation: causes pronunciation information to be captured
* --statistics: prints useful statistics at the end
* --pages-dir DIR: save all wiktionary pages under this directory (mostly for debugging)
* --help: displays help text

## Calling the Library

The library can be called as follows:

```
import wiktextract

ctx = wiktextract.parse_wiktionary(
    path, word_cb,
    capture_cb=None,
    capture_languages=["English", "Translingual"],
    capture_translations=False,
    capture_pronunciation=False):
```

The ``parse_wiktionary`` call will call ``word_cb(data)`` for words
found on every page in Wiktionary.  ``data`` is information about a
single word and part-of-speech as a dictionary (multiple senses of the
same part-of-speech are combined into the same dictionary).  It is in the
same format as the JSON-formatted dictionaries returned by the ``wiktwords``
tool (see below).

``capture_cb(title, text)`` is called for every page before extracting any
words from it.  It should return True if the page should be analyzed, and
False if the page should be ignored.  It can also be used to write certain
pages to disk or capture certain pages for different analyses (e.g., extracting
hierarchies, classes, thesauri, or topic-specific word lists).  If this
callback is None, all pages are analyzed.

``capture_languages`` should be a list, tuple, or set of language names to
capture.  It defaults to ``["English", "Translingual"]``.

``capture_translations`` can be set to True to capture translation
information for words.  Translation information seems to be most
widely available for the English language, which has translations into
other languages.  The translation information increases the size and
loading timne of the captured data substantially, so this is disabled
by default.

``capture_pronunciation`` can be set to True t capture pronunciation
information for words.  Typically, this includes IPA transcriptions
and any audio files included in the word entries, along with other
information.  However, the type and amount of pronunciation
information varies widely between languages.  This is disabled by
default since many applications won't need the information.

## Format of the extracted word entries

Information returned for each word is a dictionary.  The dictionary has the
following keys (others may also be present or added later):

* word: the word form
* pos: part-of-speech, such as "noun", "verb", "adj", "adv", "pron", "determiner", "prep" (preposition), "postp" (postposition), and many others.
* senses: word senses for this word/part-of-speech (see below)
* conjugation: conjugation/declension entries found for the word
* heads: part-of-speech specific head tags for the word.  Useful for, e.g., obtaining comparatives, superlatives, and other inflection information for many languages.  Each value is a dictionary, basically containing the arguments of the corresponding template in Wiktionary, with the template name under "template_name".
* hyphenation: list of hyphenations for the word when available.  Each hyphenation is a vector of syllables.
* pinyin: for Chinese words, the romanized translitteration, when available
* synonyms: synonym linkages for the word (see below)
* antonyms: antonym linkages for the word (see below)
* hypernyms: hypernym linkages for the word (see below)
* holonyms: linkages indicating being part of something (see below) (not systematically encoded)
* meronyms: linkages indicating having a part (see below) (fairly rare)
* derived: derived word linkages for the word (see below)
* related: related word linkages for the word (see below)
* sounds: contains pronunciation information when collected (see below)
* translations: contains translation information when collected (see below)

### Word Senses

Each part-of-speech may have multiple glosses under the "senses" key.  Each
sense is a dictionary that may contain the following keys (among others, and more may be added in the future):

* glosses: list of gloss strings for the word sense (usually only one).  This has been cleaned, and should be straightforward text with no tagging.
* nonglosses: list of gloss-like strings but that are not traditional glossary entries describing the word's meaning
* tags: list of qualifiers and tags for the gloss.  This is a list of strings, and may include words such as "archaic", "colloquial", "present", "plural", "person", "organism", "british", "chemistry", "given name", "surname", "female", and many othes (new words may appear arbitrarily).  Some effort has been put into trying to canonicalize various sources and styles of annotation into a consistent set of tags, but it is impossible to do an exact job at this.
* senseid: list of identifiers collected for the sense.  Some entries have a Wikidata identifier (Q<numbers>) here; others may have other identifiers.  Currently sense ids are not very widely annotated in Wiktionary.
* wikipedia: link to wikipedia page from the word sense/gloss
* topics: topic categories specified for the sense (these may also be in "tags")
* taxon: links to taxonomical data
* color: specification of RGB color values (hex or CSS color name)
* value: value represented by the word (e.g., for numerals)
* unit: information about some represent units, particularly SI units
* alt_of: list of words that this sense is an alternative form or abbreviation of
* inflection_of: list of words that this sense is an inflection of
* conjugation: list of templates indicating conjugation/declension (list of dictionaries containing the arguments of the Wiktionary template, with template name under "template_name")

### Linkages to Other Words

Linkages (synonyms, antonyms, hypernyms, derived words, holonyms, meronyms, derived terms, related terms) are lists of dictionaries, where each dictionary can contain the following keys, among others:

* word: the word this links to (string)
* sense: text identifying the word sense (may not match gloss exactly) (string)
* tags: qualifiers specified for the sense (e.g., field of study, region, dialect, style).  This is a list of strings.

### Pronunciation

Pronunciation information is stored under the "sounds" key.  It is a
list of dictionaries, each of which may contain the following keys,
among others:

* audios: list of audio files referenced (list of tuples)
* ipas: pronunciation specifications in IPA format as tuples (lang, ipatext)
* special_ipas: special IPA-like specifications (sometimes macros calling code in Wiktionary), as list of dictionaries
* en_pr": pronunciations in English pronunciation format as list of strings
* tags: qualifiers associated with the pronunciation specification, such as dialect, country, etc.

### Translations

Translations, when captured, are stored under the "translations" key
in the word's data.  They are stored in a list of dictionaries, where
each dictionary has the following keys (and possibly others):

* lang: the language that the translation is for (Wiktionary's 2 or 3-letter language code)
* word: the translation in the specified language
* sense: optional sense for which the translation is (this is a free-text string, and may not match any gloss exactly)
* alt: an optional alternative form of the translation
* roman: an optional romanization of the translation.

## Dependencies

This package depends on the following other packages:

* [lxml](https://lxml.de)
* [wikitextparser](https://pypi.org/project/WikiTextParser/)

## Contributing

Please email to ylo at clausal.com if you wish to contribute, have
patches, or suggestions.

## License

This package is free for both commercial and non-commercial use.  It
is licensed under the MIT license.  See the [LICENSE](LICENSE) file
for details.

Credit and linking to the project's web site and/or citing any future
papers on the project would be appreciated.
