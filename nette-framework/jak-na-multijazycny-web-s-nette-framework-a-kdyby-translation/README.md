# Jak na multijazyčný web s Nette Framework a Kdyby\Translation

Na Nette fóru se objevila otázka [Ako jednoducho spravit viacjazycny portal?](http://forum.nette.org/cs/23649-ako-jednoducho-spravit-viacjazycny-portal). Také jsem měl před časem podobnou otázku a řekl si, že už to nebudu pytlíkovat "tak nějak podle sebe", ale použiji řešení od Filipa Procházky [Kdyby\Translation](https://github.com/Kdyby/Translation) a musím říci, že jsem udělal dobře. Bylo to po Filipově [přednášce](https://www.youtube.com/watch?v=T7vKUwqSOiU) ve které mě nalákal na spoustu super drobností.

Rozhodl jsem se tedy, že natočím video, ve kterém Kdyby\Translation použiji a zkusím se vejít do 4 minut ať je vidět, že začít je opravdu snadné.

[https://www.youtube.com/watch?v=9u7S8tDWRxY](https://www.youtube.com/watch?v=9u7S8tDWRxY)

## Instalace

instalace Nette web project `$ composer create-project nette/web-project`

instalace Kdyby/Translation `$ composer require kdyby/translation`

## Konfigurace

do `config.neon` přidáme extension

```text
extensions:
    translation: Kdyby\Translation\DI\TranslationExtension
```

## Presenter

Do BasePresenteru přidáme proměnou $locale a necháme si injectnout službu Translator, která nám vše bude překládat \(v ukázce nemám BasePresenter, ale pouze jeden presenter HomepagePresenter\)

Proměnná $locale je [persistentní](http://doc.nette.org/cs/presenters#toc-persistentni-parametry), proto aby se "neztratila" během procházení webem.

```text
{
    /** @persistent */
    public $locale;

    /** @var \Kdyby\Translation\Translator @inject */
    public $translator;

}
```

## Router

Nahradíme původní definici defaultního routeru

```text
$router[] = new Route('[<locale=cs cs|en>/]<presenter>/<action>', "Homepage:default");
```

## Slovníky

Do adresáře `app/lang` přidáme dva soubory `ui.cs_CZ.neon` a `ui.en_US.neon`

kde `ui` je pojmenování našeho slovníku a cs\_CZ je kombinace jazyka a státu \([ISO\_639-1](https://cs.wikipedia.org/wiki/Seznam_k%C3%B3d%C5%AF_ISO_639-1) a [ISO 3166-1](https://cs.wikipedia.org/wiki/ISO_3166-1)\)

takže pokud bych chtěl třeba ruštinu tak použiji koncovku `ru_RU`

> Keeping both the language and the country in the culture is necessary because you may have a different French translation for users from France, Belgium, or Canada, and a different Spanish content for users from Spain or Mexico. The language is coded in two lowercase characters, according to the ISO 639-1 standard \(for instance, en for English\). The country is coded in two uppercase characters, according to the ISO 3166-1 standard \(for instance, GB for Great Britain\).

[http://symfony.com/legacy/doc/book/1\_0/en/13-I18n-and-L10n](http://symfony.com/legacy/doc/book/1_0/en/13-I18n-and-L10n)

## Obsah slovníku

Používám formát [neon](http://ne-on.org/)

ui.cs\_CZ.neon

```text
title: Vitejte
```

ui.en\_US.neon

```text
title: Hello
```

## Šablona

V šabloně použijeme makro `{_ ui.title}` pro vypsání překladu.

## Profit

Začít je opravdu snadné. Více najdete v dokumentaci na [https://github.com/Kdyby/Translation/blob/master/docs/en/index.md](https://github.com/Kdyby/Translation/blob/master/docs/en/index.md)

