# Zkrášlujte formulářové prvky, snadno pomocí setOption

Mám rád, když zjistím něco nového. Ten pocit dočasného prozření, že jsem něco nědělal úplně správně.

Objevil jsem Ameriku v podobě "setOption\(\)". Snažím se s pomocí JavaScriptu sem tam zlepšit ovládání nějakých klasických prvků. Tak si vemte třeba klasický input type password. Standardně velmi "nudný" formulářový prvek. Co ho tak okořenit o možnost vygenerovat náhodné heslo, zobrazit sílu zadaného hesla, nebo si místo hvězdiček nechat zobrazit heslo v čitelné podobě.

Do včerejška jsem tyto "zlepšováky" řešil pomocí css tříd nebo data atributů. Dnes je vše jinak. Již po několikáté jsem se spálil s používáním Nette Html Elementu. Nevím proč, asi to je tím, že tuto třídu zas tak aktivně nepoužívám a žiju si ve světe, kde druhý řádek _vrátí_ hodnotu. Ale tak to není [https://api.nette.org/2.4/Nette.Utils.Html.html\#\_data](https://api.nette.org/2.4/Nette.Utils.Html.html#_data), asi zlozvyk z jQuery světa [https://api.jquery.com/data/](https://api.jquery.com/data/).

```text
$htmlElement->data('generator', TRUE);
$htmlElement->data('generator'); // vrací Html Element, nikoliv hodnotu data-generator
```

Dalším bodem úrazu je pro mě pravidelně zaměnění getControl [https://api.nette.org/2.4/Nette.Forms.Controls.BaseControl.html\#\_getControl](https://api.nette.org/2.4/Nette.Forms.Controls.BaseControl.html#_getControl) vs getControlPrototype... [https://api.nette.org/2.4/Nette.Forms.Controls.BaseControl.html\#\_getControlPrototype](https://api.nette.org/2.4/Nette.Forms.Controls.BaseControl.html#_getControlPrototype)

```text
$input = $form->addPassword('password', "Password:")->getControlPrototype();
$input->data('generator', 'init');
$input->data('strength', 'init');
$input->data('switcher', 'init');
```

No když se na to kouknete tak je to takové ... makové ;-\) No a teď se podíváme na "ameriku" `setOption()`

```text
$form->addTextarea('content', "Content:")
    ->setOption('generator', TRUE)
    ->setOption('strength', TRUE)
    ->setOption('switcher', TRUE);
```

Prostě lahoda pro oči ;-\)

V šabloně se kód také píše lépe. Z původního:

```text
{if isset($input->control->attrs['data-generator'])}...{/if}
```

a nová verze

```text
{if $input->getOption('generator')}...{/if}
```

K čemu to třeba používám? Tak třeba náhled obrázku u input file, nebo pro nápovědu, co by prvek měl obsahovat. Fantazii se meze nekladou.

No a jako další krok vidím směr vlastních formulářových prvků. Tak se pak rád podělím o zkušenosti.

PS: takhle pak může vypadat šablona

```text
<div class="form-field">
    {label $input class=>"label--main"}

    {input $input}

    {if $input->getOption('generator')}
        <a class="generator" href="#">generate</a>
    {/if}
    {if $input->getOption('strength')}
        <div class="strength">strength: <span class="indicator">...</span></div>
    {/if}
    {if $input->getOption('forgot')}
        {if $input->getOption('loginCounter') > 2}
            <a n:href="Recovery:" class=”huge”>Forgot password?</a>
        {else}
            <a n:href="Recovery:" class=”small”>Forgot password?</a>
        {/if}
    {/if}
</div>
```

Ukázka v gifu: \[ __[https://cldup.com/qTMvqIHaCw.gif](https://cldup.com/qTMvqIHaCw.gif) __\]

