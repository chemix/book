# Jak řešit překlady flash zpráviček v Nette Framework

Sem tam chceme uživateli sdělit, že se něco stalo. V [Nette Frameworku](http://nette.org) je na to připravena podpora pomocí takzvaných [flash messages](http://doc.nette.org/cs/2.3/presenters#toc-flash-zpravy).

Nedávno jsem potřeboval vyřešit jejich multijazyčnou variantu a zde vám ukáži jak jsem to řešil.

## Klasika

Asi nejsnažší způsob, jak překládat flash message je přeložit jí v presenteru pomocí injectnutého translátoru. \(viz článek [jak na multijazyčný web](http://blog.honzacerny.com/post/5-jak-na-multijazycny-web-s-nette-framework-a-kdyby-translation)\)

```text
$this->flashMessage($this->translator->translate('flash.newArticleSuccess'));
```

Vše máme pod kontrolou a pokud potřebujeme do zprávy předat parametr nic nám v tom nebrání, případně změnit typ zprávy

```text
$this->flashMessage($this->translator->translate('flash.deleteUser', ['username'=>$username]), 'success');
```

Tím by mohl článek skončit, ale opak je pravdou, zde teprve začínáme.

## Addons, extensions, rozšíření

Když jsem se kouknul na výsledný kód, připadalo mi, že by se to dalo používat trošku ne tak ukecaně. A je jasné, že nebudu první koho to napadlo :\) Po chvilce hledání jsem narazil na dvě rozšíření

* IPub/FlashMessages - [http://addons.nette.org/ipub/flash-messages](http://addons.nette.org/ipub/flash-messages)
* librette/flash-messages - [https://github.com/librette/flash-messages/](https://github.com/librette/flash-messages/)

\(Narazil jsem ještě na Zenify/FlashMessageComponent, ale u komponenty je DEPRECATED, tak jsem ji rovnou přeskočil\)

Obě rozšíření mají composer balíček, takže jejich instalace byla velmi snadná.

## librette/flash-messages

Github: [https://github.com/librette/flash-messages/](https://github.com/librette/flash-messages/)

**Instalace**

`$ composer require librette/flash-messages @dev`

Po prozkoumání kódu zjistíte, že komponenta je velmi jednoduchá. Nebojte že nemá dokumentaci, její použití je snadné.

Začneme tak, že v našem presenteru použijeme [traitu](http://php.net/manual/en/language.oop5.traits.php) `TEnhancedFlashMessages` a přepíšeme metodu `getTranslator()`, kterou traita definuje a používá pro získání nějakého překladače.

```text
use Librette\FlashMessages\TEnhancedFlashMessages;

class HomepagePresenter extends Nette\Application\UI\Presenter
{
    use TEnhancedFlashMessages;

    ...

    /** @var \Kdyby\Translation\Translator @inject */
    public $translator;

    protected function getTranslator()
    {
        return $this->translator;
    }
```

_\(pro zjednodušení ukázky dávám kód do HomepagePresenteru ve vaší aplikaci to bude BasePresenter\)_

Použití je pak snadné. Traita přepisuje nativní chování metody `flashMessage()` a přidává i parametry pro složitější překlady.

Následující ukázky snad nepotřebují více komentovat

```text
    $this->flashMessage('ui.wow');
    $this->flashMessage('ui.wow', 'warning');
    $this->flashMessage('ui.wow', 'error');
    $this->flashMessage('ui.num', 'info', 3);
    $this->flashMessage('ui.name', 'success', ['name' => "Karel"]);
    $this->flashMessage('ui.numname', 'success', 3, ['name' => "Karel"]);
```

ukázkový slovník `ui.cs_CZ.neon`

```text
wow: "tohle je fleš zpráva"
num: "tohle jedna |tohle je fleš zpráva %count%"
name: "kuk %name%"
numname: "tohle jedna %name% |tohle je fleš zpráva %count% od %name%"
```

Vykreslování v šabloně můžeme nechat defaultní

```text
    <div n:foreach="$flashes as $flash" n:class="flash, $flash->type">{$flash->message}</div>
```

A nebo použijeme druhou traitu, která v balíčku je a to `TFlashMessages`, která vytváří komponentu pro vykreslování flash message s přidáním tříd pro CSS Bootstrap 3.

```text
use Librette\FlashMessages\TEnhancedFlashMessages;
use Librette\FlashMessages\Component\TFlashMessages;

class HomepagePresenter extends Nette\Application\UI\Presenter
{
    use TEnhancedFlashMessages;
    use TFlashMessages;

    ...
```

v šabloně použijeme

```text
{control flashMessages}
```

Zde bych jen upozornil, že tato komponenta má šablonu připravenou pro CSS Boostrap 3 a řeší "přejmenování" typu `error` na css třídu `danger` což mi přijde super.

Ukázky různého volání najdete na [GitHub/chemix/try-flashmessage - librette](https://github.com/chemix/try-flashmessage/blob/master/librette/app/presenters/HomepagePresenter.php)

## IPub/FlashMessages

Github: [https://github.com/iPublikuj/flash-messages](https://github.com/iPublikuj/flash-messages)

**Instalace**

`$ composer require ipub/flash-messages`

Tato komponenta má o dost více kodu nežli předchozí. Bonus jsou testy a dokumentace. Při letmém zkoumání \(nebo z dokumentace\) zjistíte, že se registruje do systému jako extension

```text
extensions:
    flashMessages: IPub\FlashMessages\DI\FlashMessagesExtension
```

do presenteru si ji opět přidáme použitím traity.

```text
use IPub\FlashMessages;

class HomepagePresenter extends Nette\Application\UI\Presenter
{
    use FlashMessages\TFlashMessages;
```

V šabloně je třeba **vždy** použít komponentu. \(komponenta používá vlastní řešení pro storage zpráv\)

```text
{control flashMessages}
```

Poslání jednoduché zprávy je pak

```text
    $this->flashMessage('ui.wow', 'danger');
```

Šablonu pro komponentu je možné změnit z defaultní na CSS Bootstrap nebo na UIkit v `config.neon`

```text
flashMessages:
    templateFile: bootstrap
```

Komponenta má další dvě volby nastavení. Vypnutí/zapnutí "title" pro zprávu, a možnost takzvané overlay zprávy.

```text
flashMessages:
    useTitle: TRUE  # defalní hodnota je TRUE
    useOverlay: TRUE
```

title:

\[ _/data/articles/8/title.png 500x?_ \]

overlay:

\[ _/data/articles/8/overlay.png 800x?_ \]

Bohužel na `title` zprávy se neaplikuje překladač, takže pokud jí chcete použít je třeba delší zápis \(v1.0.1\)

```text
    $this->flashMessage('ui.wow', 'danger', $this->translator->translate('ui.error'));
```

Pokud chcete použít parametry pro překlad a nepoužít title je třeba ho přeskočit včetně přepínače pro overlay zprávu.

```text
    $this->flashMessage('ui.num', 'info', NULL, FALSE, 1);
    $this->flashMessage('ui.name', 'success', NULL, FALSE, ['name' => "Karel"]);
```

To už tak pekně nevypadá.

Spolu s přepsanou metodou `flashMessege()` tato komponenta přichází s objektem `$this->flashNotifier`, který přináší pár zkratek, například:

```text
    $this->flashNotifier->overlay('ui.wow');
    $this->flashNotifier->error('ui.wow');
```

Overlay jsem zkoušel s nastavenou šablonou pro Bootstrap, kde jsem si načetl nějaký JavaScript a postaral se zobrazení overlay zprávy.

```text
    <link href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.5/css/bootstrap.min.css" rel="stylesheet">

    ...

    <script src="https://code.jquery.com/jquery-1.11.3.min.js"></script>
    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.5/js/bootstrap.min.js"></script>
    <script n:syntax="double">
        $(function(){
            $('.modal').modal();
        });
    </script>
```

_\(hodně lehká verze inicializace co nebude fungovat s flash zprávou a snippetem\)_

Další výhodou této komponenty je, že traitu můžete použít ve vaší komponentě a odesílat flash zprávičky přímo z vaší komponenty. Toto v současné době řeším přes události co probublají do presenteru a ten se o flash message postará. Pokud pracujete hodně s komponentama imho velmi zajímavá vlastnost.

Ukázky různého volání najdete na [GitHub/chemix/try-flashmessage - ipublikuj](https://github.com/chemix/try-flashmessage/blob/master/ipublikuj/app/presenters/HomepagePresenter.php)

## Závěr

Díky [Adamovi Kadlecovi](https://github.com/akadlec) a [Davidu Matějkovi](https://github.com/matej21) za jejich komponenty.

Librette mi přijde pro mé potřeby jako dostatečné řešení. V IPub bych bral inspiraci, pokud bych potřeboval něco více. Na druhou stranu i použití IPub v projektu, kde je Bootstap a chcete posílat jen krátké zprávy není špatná cesta a pokud chcete občas poslat overlay zprávu, tak je to pěkné hotové řešení. Jen pořadí parametrů mi přijde nevhodně zvolené a to že "title" si musím případně překládat sám \(v1.0.1\).

Zdrojové soubory najdete na [GitHub/chemix/try-flashmessage ](https://github.com/chemix/try-flashmessage)

PS: Jak řešíte překlady flash zpráviček vy?

