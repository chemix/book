# Jak začít a propojit Doctrine a Nette Framework

Znáte to, všude slyšíte, jak je ta [Doctrine](http://www.doctrine-project.org/) úžasná, jenže pak se někam kouknete a na první pohled to vypadá moc komplikovaně, tak odsouváte vyzkoušení a díky tomu u vás roste i pomyslná zeď mezi vámi a technologií. Naštěstí jsem včera byl na NetteFwPivo kde jsme se chvilku o Doctrine bavili a tak jsem si řekl, že to rovnou zkusím a výsledek byl pro mě překvapující, nejen že to bylo snadné, ale zárověn magické :\) a zde je popis jak jsme postupovali krok za krokem.

Nebojte se a vyzkoušejte si to také, stihnete to do 4minut ;-\)

Nainstaluji si Nette/Sandbox

`composer create-project nette/sandbox my-app`

`cd my-app`

`chmod 777 log temp`

Nainstaluji Kdyby/Doctrine

`composer require kdyby/doctrine`

Vytvořím si databázi a uživatele

```text
CREATE DATABASE `doctrine_devel` COLLATE 'utf8_czech_ci';
CREATE USER 'doctrine'@'localhost' IDENTIFIED BY 'kreslo';
GRANT USAGE ON * . * TO 'doctrine'@'localhost' IDENTIFIED BY 'kreslo' WITH MAX_QUERIES_PER_HOUR 0 MAX_CONNECTIONS_PER_HOUR 0 MAX_UPDATES_PER_HOUR 0 MAX_USER_CONNECTIONS 0 ;
GRANT ALL PRIVILEGES ON `doctrine\_%` . * TO 'doctrine'@'localhost';
FLUSH PRIVILEGES;
```

Nastavím připojení a zaregistruji Doctrine do Nette Framework

do config.neon přidám

```text
extensions:
    console: Kdyby\Console\DI\ConsoleExtension
    events: Kdyby\Events\DI\EventsExtension
    annotations: Kdyby\Annotations\DI\AnnotationsExtension
    doctrine: Kdyby\Doctrine\DI\OrmExtension

doctrine:
    user: doctrine
    password: '***'
    dbname: doctrine_online
    metadata:
        App: %appDir%
```

a v config.local.neon přidám lokální přístup do databáze

```text
doctrine:
    user: doctrine
    password: kreslo
    dbname: doctrine_devel
```

vytvořím si podle dokumentace [Kdyby\Doctrine](https://github.com/Kdyby/Doctrine/blob/master/docs/en/index.md) první entitu

app/model/Article.php

```text
namespace App;

use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity
 */
class Article extends \Kdyby\Doctrine\Entities\BaseEntity
{

    /**
     * @ORM\Id
     * @ORM\Column(type="integer")
     * @ORM\GeneratedValue
     */
    protected $id;

    /**
     * @ORM\Column(type="string")
     */
    protected $title;

}
```

V konzoli si vyzkouším zdali funguje Doctrine console

`php ./www/index.php`

a vytvořím databázovou tabulku

`php ./www/index.php orm:schema-tool:create`

Naplním tabulku daty a zkusím se k nim dostat pomocí Doctrine v presenteru.

Injectnu si EntityManager do presenteru a dumpnu si všechny články.

HomepagePresenter.php

```text
<?php

namespace App\Presenters;

use App\Article;
use Nette,
    App\Model;


/**
 * Homepage presenter.
 */
class HomepagePresenter extends BasePresenter
{

    /**
     * @inject
     * @var \Kdyby\Doctrine\EntityManager
     */
    public $EntityManager;

    public function renderDefault()
    {
        $dao = $this->EntityManager->getRepository(Article::getClassName());
        dump($dao->findAll());
        exit();
    }

}
```

A jelikož vše funguje pošlu entity do šablony a vypíšu je

```text
    public function renderDefault()
    {
        $dao = $this->EntityManager->getRepository(Article::getClassName());
//        dump($dao->findAll());exit();
        $this->template->articles = $dao->findAll();
    }
```

Homepage.latte

```text
{* This is the welcome page, you can delete it *}

{block content}

{foreach $articles as $article}

    {$article->title}<br>

{/foreach}
```

Tradááá.

Strach z nového překonán, vše se zdá krásně snadné a super. Další krok bude výroba servisy Articles ať nejsme za hulváta.

Díky [Davidovi](https://github.com/matej21), že mi pomohl překonat mentální blok vyzkoušet Doctrine a [Filipovi](https://github.com/fprochazka) za úžasnou práci na Kdyby/Doctrine.

