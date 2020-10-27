# How to start developing multilanguage website with Nette Framework and Kdyby\Translation

Many people on Nette forum asked how to develop a multilingual website. This was my question as well and I'd like to share my solution with you. Filip Prochazka has developed [Kdyby\Translation](https://github.com/Kdyby/Translation), an adapter for Symfony\Transaltion for Nette Framework, and has lots of super truper small features which you will love.

Here is a four minute long videocast, an introduction to how easy it is to start using Kdyby\Translation.

[https://www.youtube.com/watch?v=FFOO5j-zpSU](https://www.youtube.com/watch?v=FFOO5j-zpSU)

## Installation

1\) Install Nette web-project: `$ composer create-project nette/web-project`

2\) Install Kdyby/Translation: `$ composer require kdyby/translation`

## Configuration

in `config.neon` we will add an extension to enable Kdyby\Translation:

```text
extensions:
        translation: Kdyby\Translation\DI\TranslationExtension
```

## Presenter

In our BasePresenter we will add the variables `$locale` and `$tranlator`. Using \[Dependency Injection \| doc:di\] mechanism we get an injected Kdyby\Translation\Translator service. \(In my example, I don't use any BasePresenter, instead I only use the HomepagePresenter\).

The variable `$locale` is [persistent](http://doc.nette.org/en/presenters#toc-persistentni-parametry) and will now be automatically held in the url.

```text
{
        /** @persistent */
        public $locale;

        /** @var \Kdyby\Translation\Translator @inject */
        public $translator;
}
```

## Router

Update the original router definition in `RouterFactory.php`

```text
$router[] = new Route('[<locale=en cs|en>/]<presenter>/<action>', "Homepage:default");
```

\(In the video, locale defaults to cs.\)

## Dictionaries

We will add the two files `ui.cs_CZ.neon` and `ui.en_US.neon` to the folder `app/lang`. Here `ui` is the name of our dictionary and `cs_CZ` is a combination of language and state. \([ISO\_639-1](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) and [ISO 3166-1](https://en.wikipedia.org/wiki/ISO_3166-1)\)

If we want a specific translation with localization, e.g. French speaking Canadians, we will use `fr_CA`.

> Keeping both the language and the country in the culture is necessary because you may have a different French translation for users from France, Belgium or Canada, and a different Spanish content for users from Spain or Mexico. The language is a code of two lowercase characters, according to the ISO 639-1 standard \(for instance, en for English\). The country is a code of two uppercase characters, according to the ISO 3166-1 standard \(for instance, GB for Great Britain\).

[http://symfony.com/legacy/doc/book/1\_0/en/13-I18n-and-L10n](http://symfony.com/legacy/doc/book/1_0/en/13-I18n-and-L10n)

## Dictionary content

The default syntax is Neon, you can read more about its amazing syntax [http://ne-on.org](http://ne-on.org/).

ui.cs\_CZ.neon

```text
title: Vitejte
```

ui.en\_US.neon

```text
title: Hello
```

## Template

In our template, we will use the macro `{_ ui.title}` for putting translated phrase.

## Profit

It's really easy to start.

For more information read the [official documentation for Kdyby\Translation](https://github.com/Kdyby/Translation/blob/master/docs/en/index.md).

