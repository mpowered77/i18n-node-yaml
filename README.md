i18node
=======

## Motivations

Yet another translation module for node?

Not quite.

I created this repo for two main reasons:

### 1. Lack of YAML support in the [main translation module](https://github.com/mashpie/i18n-node)

Despite [having been PRed](https://github.com/mashpie/i18n-node/pull/79) it was never merged, which induces a lack of maintenance.

Most people agree that YAML is a better format for writing translations as content is naturally aligned, and require few special character tweaks.

It means a non-techie could definitely maintain his own version of a translation file, at very little risks.

### 2. Inverted translation position

All translation module I have ever used manage translations with a 1 language per file logic:

```
# en.yml

hello:
  world: Hello World
foo:
  bar: Foo bar
```

and

```
# fr.yml

hello:
  world: Bonjour le monde
foo:
  bar: Foo bar
```

I can't find how this structure can be easily maintainable. [I already asked the question on SO](http://stackoverflow.com/questions/25664708/rails-i18n-separate-language-key-at-the-end-of-the-tree), without much success.

Having translated myself several websites, I have always found super efficient to manage one file which would look like:

```
# translations.yml

hello:
  world:
    en: Hello World
    fr: Bonjour le monde
foo:
  bar: Foo bar
```

Here are the immediate advantages I get:
- No need to duplicate if translation is the same (eg. `foo.bar`)
- No multiple file to edit if tree structure moves (and it usually does)
- Visually obvious what translations are missing
- Instant evaluation of the necessity to translate (eg. English `Contact` will do at first then will be translated by `Kontact` in German eventually)
- Super easy to translate : the original source is only 1 line above! No need fancy software to do that.
- Files are organized by content, not translations

I'd be curious to hear some cons on this!

## Usage

### Quick start

Inspired by [its grand-bother](https://github.com/mashpie/i18n-node), the syntax goes:

```
// app.js
const i18n = require('i18node')({
  debug: app.get('environment') !== 'production',
  translationFolder: __dirname + '/translations',
  locales: ['en_US', 'fr_FR'],
  defaultLocale: 'en_US',
  queryParameters: ['lang'],
});

i18n.ready.catch(err => {
  console.error('Failed loading translations', err);
});

app.use(i18n.middleware);
```

Translations become available in the `req`/`res` objects with:

```
// Any controller

req.t('translations.hello.world');
// OR
res.locals.t('translations.foo.bar');
```

The `res.locals.t` syntax makes it accessible in an `ejs`/`pug` view immediately:

```
// index.pug
html(lang=getLanguage())
  head
    meta(property='og:locale', content=getLocale())
    for language in getLocales().filter(lang => lang !== getLocale())
      meta(property='og:locale:alternate', content=language)
  body
    p= t('translations.hello.world')
```

### API:

- `getLocale()`: returns the current locale. Eg. `'en_US'`
- `getLanguage()`: returns the current langage. Eg. `'en'`
- `getLocales()`: returns an array of available locales: Eg. `['en_US', 'fr_FR']`
- `getLanguages()`: returns an array of available languages: Eg. `['en', 'fr']`
- `t(root, path, data, locale)`:

  - **`root` (type: `Object`)** : is the object to get translations from. It is optional as by default its value is the whole translations tree. But it can be useful in the case of an array of values:

    ```
    # elements.yml

    list:
      - key: foo
        value:
          en: I am crazy
          fr: Je suis fou
      - key: bar
        value:
          en: I like bars
          fr: J'aime les bars
      - key: baz
        value:
          en: I'm downstairs
          fr: Je suis en baz
    ```

    ```
    //- index.pug
    each element in t('elements.list')
      p(data-key=t(element, 'key'))= t(element, 'value')
    ```

  - **`path` (type: `string`)** : is the dot-separated succession of keys to access the value from the root. If root is not set, `path` becomes the first argument

    You can write, back to the example above: `t('elements.list.0.value')` but you are more likely to loop over the values of the array

  - **`data` (type: `object`)** : allows to make string interpolation in translations.

    Eg.

    ```
    # emails.yml

    welcome:
      en: |
        Welcome ${name},
        Click on this [link](${link}) to continue.
        Best
      fr: |
        Bienvenue ${name},
        Cliquez sur ce [lien](${link}) pour poursuivre.
        Bien à vous
    ```

    Then `t('emails.welcome', {name: 'Robert', link: 'http://example.com'})` will work as expected.

  - **`locale` (type: `string`)** : most of the time you will use the selected locale in the middleware, but it is possible to override it: `t('translations.hello.world', {}, 'fr')`.

## TODOs

There are a lot of things to do. You are more than welcome to encourage and contribute to the development of this repo.

Here are some steps I can immediately see:

- deal with pluralization (I haven't had the need until now)
- write some tests
- spread the word
- publish to npm