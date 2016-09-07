---
title: 'Как составлять свои фильтры'
taxonomy:
    category:
        - docs
visible: true
---

* [Введение](#introduction)
* [Базовые правила](#basic-rules)
    * [Синтаксис базовых правил](#syntax-of-basic-rules)
    * [Специальные символы](#Специальные-символы)
    * [Поддержка регулярных выражений](#Поддержка-регулярных-выражений)
    * [Примеры базовых правил](#Примеры-базовых-правил)
    * [Модификаторы](#Модификаторы)
        * [Базовые модификаторы](#Базовые-модификаторы)
            * [$domain](#domain)
            * [$third-party](#third-party)
            * [$popup](#popup)
            * [$match-case](#match-case)
        * [Ограничение по типу контента](#Ограничение-по-типу-контента)
            * [Примеры ограничений](#Примеры-ограничений)
            * [$image](#image)
            * [$stylesheet](#stylesheet)
            * [$script](#script)
            * [$object](#object)
            * [$object-subrequest](#object-subrequest)
            * [$font](#font)
            * [$subdocument](#subdocument)
            * [$xmlhttrequest](#xmlhttprequest)
            * [$other](#other)
        * [Модификаторы правил-исключений](#Модификаторы-правил-исключений)
            * [$elemhide](#elemhide)
            * [$content](#content)
            * [$jsinject](#jsinject)
            * [$urlblock](#urlblock)
            * [$document](#document)
            * [$stealth](#stealth)
            * [Generic правила](#generic-rules)
                * [$generichide](#generichide)
                * [$genericblock](#genericblock)
    * [Расширенные возможности](#Расширенные-возможности)
        * [$empty](#empty)
        * [$mp4](#mp4)
        * [$replace](#replace)
* [Косметические правила](#Косметические-правила)
    * [Правила скрытия элементов](#Правила-скрытия-элемeнтов)
        * [Синтаксис правил скрытия](#Синтаксис-правил-скрытия)
        * [Примеры правил скрытия](#Примеры-правил-скрытия)
        * [Исключения для правил скрытия](#Исключения-для-правил-скрытия)
    * [Правила CSS стилей](#Правила-css-стилей)
        * [Синтаксис правил CSS стилей](#Синтаксис-правил-css-стилей)
        * [Примеры правил CSS стилей](#Примеры-правил-css-стилей)
    * [Расширенные CSS селекторы](#Расширенные-css-селекторы)
        * [Псевдо-класс :has()](#Псевдо-класс-has)
        * [Псевдо-класс :contains()](#Псевдо-класс-contains)
        * [Псевдо-класс :matches-css()](#Псевдо-класс-matches-css)
* [Правила фильтрации HTML](#Правила-фильтрации-html)
    * [Синтаксис правил фильтрации HTML](#Синтаксис-правил-фильтрации-html)
    * [Примеры правил фильтрации HTML](#Примеры-правил-фильтрации-html)
    * [Специальные атрибуты](#Специальные-атрибуты)
        * [tag-content](#tag-content)
        * [wildcard](#wildcard)
        * [max-length](#max-length)
        * [min-length](#min-length)
        * [parent-elements](#parent-elements)
        * [parent-search-level](#parent-search-level)
    * [Исключения для правил фильтрации HTML](#Исключения-для-правил-фильтрации-html)
* [Javascript правила](#javascript-правила)
    * [Синтаксис javascript правил](#Синтаксис-javascript-правил)
    * [Примеры javascript правил](#Примеры-javascript-правил)
    * [Исключения для javascript правил](#Исключения-для-javascript-правил)

## Введение<a id="introduction"></a>

Фильтр — это набор правил фильтрации рекламного контента (баннеров, всплывающих окон и тому подобного). Вместе с Adguard поставляется набор стандартных фильтров, создаваемых нами. Они постоянно дорабатываются и дополняются, и, как мы надеемся, удовлетворяют большинство пользователей.

Вместе с тем, Adguard позволяет создать ваш собственный пользовательский фильтр, используя те же самые правила составления фильтров, что используем мы в наших фильтрах.

Для описания синтаксиса правил мы используем [Augmented BNF for Syntax Specifications](https://tools.ietf.org/html/rfc5234), но мы не всегда строго следуем этой спецификации.

> Синтаксис правил Adguard изначально основан на синтаксисе правил Adblock Plus, но расширяет его, добавляя новые типы правил для лучшей фильтрации. Часть информации в этой статье о типах правил, общих для ABP и Adguard, взята из [этой статьи](http://adblockplus.org/ru/filters).

## Базовые правила<a id="basic-rules"></a>

Так называемые _"Базовые правила"_ — самый простой вид правил. Эти правила предназначены для блокировки запросов на определенный адрес. Либо, при наличии специального маркера `@@` в начале правила, для разблокировки запроса. Основной принцип для этого типа правил достаточно прост: необходимо указать адрес и дополнительные параметры, которые ограничивают или расширяют область действия правила.

> Базовые правила, блокирующие запросы, применяются **только к подзапросам**. То есть, они не будут блокировать загрузку страницы в браузере.

> Браузер определяет заблокированный подзапрос как выполненный с ошибкой.

### Синтаксис базовых правил<a id="basic-rules-syntax"></a>

```
      rule = ["@@"] pattern [ "$" modifiers ]
 modifiers = [modifier0, modifier1[, ...[, modifierN]]]
```

* **`pattern`** — маска адреса. URL каждого запроса сопостовляется с этой маской. В шаблоне вы можете использовать некоторые специальные символы, описание которых будет дано [ниже](#Специальные-символы).
* **`@@`** — маркер, который используется для обозначения правил-исключений. С такого маркера должны начинаться правила, отключающие фильтрацию для запроса.
* **`modifiers`** — Параметры, используемые для "уточнения" базового правила. Некоторые параметры ограничивают область действия правила, а некоторые могут полностью изменить принцип его работы.

### Специальные символы<a id="basic-rules-special-characters"></a>

* **`*`** — Wildcard-символ. Символ, обозначающий "произвольный набор символов". Это может быть как пустая строка, так и строка любой длины.
* **`||`** — Соответствие началу адреса. Этот специальный символ позволяет не указывать конкретный протокол и поддомен в маске адреса. То есть, `||` соответствует сразу `http://*.`, `https://*.`, `ws://*.`, `wss://*.`.
* **`^`** — указатель для разделительного символа. Разделителем может быть любой символ кроме буквы, цифры и следующих символов: `_` `-` `.` `%`. Например, в адресе `http:`**`//`**`example.com`**`/?`**`t=1`**`&`**`t2=t3` жирным выделены разделительные символы.
* **`|`** — указатель на начало или конец адреса. Значение зависит от того, находится этот символ в начале или конце маски. Например, правило `swf|` соответствует `http://example.com/annoyingflash.swf` , но не `http://example.com/swf/index.html`. `|http://example.org` соответствует `http://example.org`, но не `http://domain.com?url=http://example.org`.

### Поддержка регулярных выражений<a id="regexp-support"></a>

Если вы хотите еще большей гибкости при составлении правил, вы можете использовать [регулярные выражения](https://developer.mozilla.org/ru/docs/Web/JavaScript/Guide/Regular_Expressions) вместо упрощенной маски со специальными символами, которая используется по-умолчанию. 

> Такие правила работают медленнее обычных, поэтому рекомендуется избегать их, или, хотя бы, ограничивать их область действия конкретными доменами.

Чтобы блокировщик определил, что вы хотите использовать регулярное выражение, необходимо, чтобы `pattern` имел особый вид:
```
pattern = "/" regexp "/"
```

Например, правило `/banner\d+/$third-party` следующее правило применит регулярное выражение `banner\d+` ко всем сторонним запросам. Правила-исключения с использованием регулярных выражений выглядят вот так: `@@/banner\d+/`.

> Браузерное расширение Adguard для Safari и Adguard для iOS не полностью поддерживают регулярные выражения в силу [ограничений Content Blocking API](https://webkit.org/blog/3476/content-blockers-first-look/) (в статье по ссылке найдите раздел "The Regular expression format").

### Примеры базовых правил<a id="basic-rules-examples"></a>

* `||example.com/ads/*` — простое правило, которое просто соотвествует адресам типа `http://example.com/ads/banner.jpg` и даже `http://subdomain.example.com/ads/otherbanner.jpg`.

* `||example.org^$third-party` — правило, которое блокирует сторонние запросы к домену `example.org` и его поддоменам.

* `@@||example.com$document` — наиболее общее правило-исключение. Такое правило полностью отключает фильтрацию на домене `example.com` и всех его поддоменах. Существует ряд параметров, которые также можно использовать в правилах-исключениях. Более подробно о правилах-исключениях и параметрах, которые могут в таких правилах использоваться, написано [ниже](#exception-rules-modifiers).

### Модификаторы<a id="basic-rules-modifiers"></a>

> Возможности, описанные в этом разделе, обычно используются опытными пользователями. Они расширяют возможности «Общих правил», но для их применения необходимо иметь начальное представление о работе браузера.

Вы можете изменить поведение «общего правила», используя дополнительные модификаторы. Список этих параметров располагается в конце правила за знаком доллара `$` и разделяется запятыми.

Пример использования модификаторов:
```
||domain.com$popup,third-party
```

#### Базовые модификаторы<a id="basic-rules-common-modifiers"></a>

Приведенные ниже модификаторы являются наиболее простыми для понимания и часто применяемыми.

##### **`domain`**<a id="domain-modifier"></a>

Модификатор `domain` ограничивает область действия правила списком доменов (и их поддоменов). Для того, чтобы указать несколько доменов в одном правиле, нужно использовать символ `|` в качестве разделителя.

###### Примеры `domain`

* `||baddomain.com^$domain=example.org` — правило для блокировки запросов, которые соответствуют указанной маске, и отправленных с домена или поддомена `example.org`.
* `||baddomain.com^$domain=example.org|example.com` 

Чтобы правило не применялось на определенных доменах, перед доменным именем необходимо добавить символ `~`.

###### Примеры `domain` и `~`

* `||baddomain.com^$domain=~example.org` — правило для блокировки запросов, которые соответствуют указанной маске, и отправленных с любого домена, кроме `example.org` и его поддоменов.
* `||baddomain.com^$domain=example.org|~foo.example.org` — в данном примере правило будет соответствовать запросам, отправленных с домена `example.org` и всех его поддоменов, кроме поддомена `foo.example.org`.

##### **`third-party`**<a id="third-party-modifier"></a>

Ограничение на сторонние или собственные запросы. Сторонним является запрос, отправленный с другого сайта. Например, запрос к домену `example.org`, отправленный с домена `domain.com` является сторонним. 

> Обратите внимание, что запрос, отправленный на поддомен того же домена (или наоборот), сторонним не является. Например, запрос к `subdomain.example.org`, отправленный с домена `example.org`, не является сторонним.

Если указан модификатор `third-party`, то правило применяется только к сторонним запросам.

###### Примеры `third-party`

* `||domain.com$third-party` — правило применяется на всех сайтах, кроме `domain.com` и его поддоменов. Пример стороннего запроса: `http://example.org/banner.jpg`.

Если указан модификатор `~third-party`, то правило применяется только к запросам, которые не являются сторонними. То есть, эти запросы отправлены с того же домена.

###### Примеры `~third-party`

* `||domain.com$~third-party` — такое правило уже будет применяться только на самом `domain.com`, но не на других сайтах. Пример запроса, который не является сторонним `http://domain.com/icon.ico`.

##### **`popup`**<a id="popup-modifier"></a>

Adguard будет пытаться закрыть браузерную вкладку с любым адресом, подходящим под правило с этим модификатором. Обратите внимание, что закрыть можно не любую вкладку.

###### Примеры `popup`

* `||domain.com*^$popup` — при попытке перехода на сайт `http://domain.com` с любой страницы в браузере, новая вкладка, в которой должен открыться указанный сайт, будет закрыта.

##### **`match-case`**<a id="match-case-modifier"></a>

Определяет правило, которое применяется только к адресам с совпадением регистра букв. По умолчанию регистр символов не учитывается.

###### Примеры `match-case`

* `*/BannerAd.gif$match-case` — такео правило будет блокировать `http://example.com/BannerAd.gif`, но не `http://example.com/bannerad.gif`.

#### Ограничение по типу контента<a id="content-type-modifiers"></a>

Существует целый набор модификаторов, которые ограничивают действие правила только запросами к ресурсам, определенного типа. Эти модификаторы можно комбинировать, например покрыть одновременно картинки и скрипты.

> Обратите внимание, что существует большая разница в том, как разные версии Adguard определяют тип контента. В случае браузерных расширений тип контента для каждого запроса предоставляется самим браузером. В случае Adguard для Windows, Mac и Android, для определения используется следующая методика: вначале мы пытаемся определить тип запроса по расширению загружаемого файла. Если запрос не заблокирован на этом этапе, то тип запроса уточняется, используя заголовок `Content-Type` в начале ответа, полученного от сервера.

##### Примеры ограничений<a id="content-type-modifiers-examples"></a>

* `||example.org^$image` — соответствует всем картинкам с домена `example.org`.
* `||example.org^$script,stylesheet` — соответствует всем скриптам и стилям с домена `example.org`.
* `||example.org^$~image,~script,~stylesheet` — соответствует всем запросам к домену `example.org` кроме картинок, скриптов и стилей.

##### **`image`**<a id="image-modifier"></a>

Правило будет соответствовать запросам к изображениям.

##### **`stylesheet`**<a id="stylesheet-modifier"></a>

Правило будет соответствовать запросам к файлам CSS стилей.

##### **`script`**<a id="script-modifier"></a>

Правило будет соответствовать запросам к файлам скриптов (например javascript, vbscript).

##### **`object`**<a id="object-modifier"></a>

Правило применяется к ресурсам, управляемым браузерными плагинами (например, Java или Flash).

##### **`object-subrequest`**<a id="object-subrequest-modifier"></a>

Запросы, инициированные плагинами (чаще всего это Flash).

> Adguard для Windows, macOS и Android часто не может корректно определить этот тип, и определяет его как `other`.

##### **`font`**<a id="font-modifier"></a>

Правило будет соответствовать запросам к файлам шрифтов (например, файлы с расширением .woff).

##### **`subdocument`**<a id="subdocument-modifier"></a>

Правило будет соответствовать запросам к встроенным страницам (HTML-теги `frame` и `iframe`).

##### **`xmlhttprequest`**<a id="xmlhttprequest-modifier"></a>

Правило применяется только к ajax-запросам. То есть к запросам, отправленным с помощью javascript-объекта `XMLHttpRequest`.

> Adguard для Windows, macOS и Android не всегда может корректно определить этот тип контента, и определяет его как `other` или `script`.

##### **`other`**<a id="other-modifier"></a>

Правило применяется к запросам, для которых не был определен ни один из перечисленных выше типов.

#### Модификаторы правил-исключений

Правила-исключения отключают действие других базовых правил для адресов, которым они соответствуют. Отличаются правила исключения тем, что они начинаются со специального маркера `@@`. Для таких правил работают все базовые модификаторы, перечисленные выше. Также добавляются несколько специальных модификаторов, которые будут описаны ниже.

##### **`elemhide`**

Отключает косметические правила на страницах, подходящих под правило. О косметических правилах речь [пойдет ниже](#Косметические-правила).

###### Примеры `elemhide`

* `@@||example.com^$elemhide` — отменяет все косметические правила для страниц на сайте `example.com` и всех его поддоменов.

##### **`content`**

Отключает правила фильтрации HTML-элементов на страницах, подходящих под правило. О правилах фильтрации HTML-элементов речь [пойдет ниже](#html-filtering-rules).

###### Примеры `content`

* `@@||example.com^$content` — отменяет все правила фильтрации HTML-элементов для страниц на сайте `example.com` и всех его поддоменов.

##### **`jsinject`**

Запрещает добавление javascript-кода на страницу. О JS-правилах речь пойдет ниже.

###### Примеры `jsinject`

* `@@||example.com^$jsinject` — отменяет все JS-правилах для страниц на сайте `example.com` и всех его поддоменов.

##### **`urlblock`**

Отключает блокировку всех запросов, отправленных со страниц подходящих под это правило.

###### Примеры `urlblock`

* `@@||example.com^$urlblock` — запросы, отправленные со страниц на сайте `example.com` и всех его поддоменов, не будет блокироваться.

##### **`document`**

Полностью отключает блокировку на страницах, подходящих под это правило. Эквивалентно одновременному использованию модификаторов `elemhide`, `content`, `urlblock` и `jsinject`.

###### Примеры `document`

* `@@||example.com^$document` — полностью отключает блокировку для всех страниц на домене `example.com` и всех его поддоменах.

##### **`stealth`**

Отключает модуль "Антитрекинг" для всех страниц и запросов, подходящих под это правило.

> Модуль "Антитрекинг" на данный момент доступен только в Adguard для Windows. В будущем мы планируем добавить его во все продукты Adguard.

> В продуктах без поддержки модуля "Антитрекинг" правила с этим модификатором игнорируются.

###### Примеры `stealth`

* `@@||example.com^$stealth` — полностью отключает модуль `Антитрекинг` для всех страниц на домене `example.com` и всех его поддоменах, а также для всех запросов и подзапросов.

##### Generic правила

Перед тем, как перейти к описанию следующих модификаторов, необходимо ввести определение _"generic правил"_. Правило является "generic", если его действие не ограничены конкретными доменами.

Например, следующие правила являются "generic":
```
###banner
~domain.com###banner
||domain.com^
||domain.com^$domain=~example.com
```

А вот такие правила уже не являются "generic":
```
domain.com###banner
||domain.com^$domain=example.com
```

###### **`generichide`**

Отключает все "generic" [косметические правила](#Косметические-правила) на страницах, подходящих под правило-исключение.

* `@@||example.com^generichide` — отключает "generic" косметические правила на страницах сайта `example.com` и всех его поддоменах.

###### **`genericblock`**

Отключает все generic базовые правила на страницах, подходящих под правило-исключение.

* `@@||example.com^$genericblock` — отключает "generic" базовые правила на страницах сайта `example.com` и всех его поддоменах.

### Расширенные возможности

Модификаторы, описанные в этом разделе, полностью меняют поведение базовых правил, к которым эти модификаторы применены.

> Модификаторы из этого раздела доступны только в Adguard для Windows, macOS и Android. Браузерные расширения ограничены возможностями, предоставляемыми браузерами, и такие им просто недоступны.

##### **`empty`**

Обычно, заблокированный запрос выглядит для браузера как ошибка сервера. В случае применения модификатора `empty`, Adguard эмулирует пустой ответ сервера со статусом `200 OK`.

###### Примеры `empty`

* `||example.org^$empty` — возвращает пустой ответ для всех запросов к домену `example.org` и всех его поддоменов.

##### **`mp4`**

В качестве ответа на заблокированный запрос Adguard возвращает короткое видео-заглушку.

###### Примеры `mp4`

* `||example.com/videos/$mp4` — блокирует загрузку видео с адресов `||example.com/videos/*` и заменяет ответ на видео-заглушку.

##### **`replace`**

Этот модификатор полностью изменяет поведение правила. Если он применен, правило не будет блокировать запрос. Вместо этого содержимое ответа будет модифицировано в соответствии со значением модификатора.

###### Особенности `replace` правил

* `replace` правила применяются к любому текстовому ответу, но не будут применяться к бинарным (`media`, `image`, `object`, etc).
* `replace` правила не применяются если размер оригинального ответа больше 3МБ.
* Если к коду страницы применено `replace` правило, то другие правила (например, косметические) к коду страницы применены не будут.
* `replace` правила имеют более высокий приоритет, чем другие базовые правила (кроме исключений). То есть, если запрос соответствует двум правилам, и одно из них имеет модификатор `replace`, то именно это правило и будет применено.

###### Синтаксис `replace`

В целом, синтаксис `replace` напоминает то, как замены регулярными выражениями [работают в Perl](http://perldoc.perl.org/perlrequick.html#Search-and-replace).

```
replace = "/" regex "/" replacement "/" modifiers
```

* `regex` — регулярное выражение.
* `replacement` — строка, которая будет использоваться для замены строки, соответствующей `regex`.
* `modifiers` — флаги регулярного выражения. Например, `i` - регистро-независимый поиск, или `s` - single-line режим.

В значении `replace` два символа должны быть экранированы: запятая (`,`) и (`$`). Для экранирования используется символ косой черты (`\`). Например, экранированная запятая: `\,`.

###### Примеры `replace`

```
://*.damoh.golem.de/$replace=/(<VAST(.|\s)*?>)(.|\s)*<\/VAST>/\$1<\/VAST>/i,domain=video.golem.de
```

В этом правиле можно выделить три части:

* Регулярное выражение: `(<VAST(.|\s)*?>)(.|\s)*<\/VAST>`
* Замена `\$1<\/VAST>` (обратите внимание, что символ `$` экранировано)
* Флаги регулярного выражения: `i` (регистро-независимый поиск)

Посмотрите, как работает это правило:
http://regexr.com/3cesk

## Косметические правила

Принцип работы косметических правил понятен из их названия. В отличие от базовых правил, косметические предназначены не для блокирования рекламных запросов, а для обработки внешнего вида страниц. Например, они могут скрывать элементы страниц или даже преобразовывать общий стиль страниц.

> Для работы с косметическими правилами необходимы знания HTML и CSS. Так что если вы хотите научиться самостоятельно составлять такие правила, нужно получить хотя бы базовое знание этих технологий. Рекомендуем ознакомиться с [этой документацией](https://developer.mozilla.org/ru/docs/Web/Guide/CSS/Getting_started/What_is_CSS).

### Правила скрытия элемeнтов

Правила скрытия элементов предназначены, как это следует из названия, для скрытия элементов веб-страниц. По сути, это аналогично применению стиля `{ display: none; }` к выбранному элементу 

#### Синтаксис правил скрытия

```
   rule = [domains] "##" selector
domains = [domain0, domain1[, ...[, domainN]]]
```

* **`selector`** — [CSS селектор](https://developer.mozilla.org/ru/docs/Web/Guide/CSS/Getting_Started/Selectors), задающий элементы, которые должны быть скрыты.
* **`domains`** — ограничение на домены, на страницах которых будет применено правило.

Если вы хотите ограничить область действие одним или более доменами, просто перечислите их через запятую. Например: `example.org,example.com##selector`.

> Обратите внимание, что это правило будет работать также на всех поддоменах `example.org` и `example.com`.

Если вы хотите запретить действие правила на определенном домене, используйте символ `~` перед именем домена. Например: `~example.org##selector`.

Обратите внимание, что вы можете использовать оба подхода в одном правиле. Например, правило `example.org,~subdomain.example.org##domain` будет работать для домена `example.org` и всех его поддоменов, **кроме `subdomain.example.org`**.

> Правила скрытия не зависят друг от друга. Если в фильтре есть правило `example.org##selector`, и вы добавляете правило `~example.org##selector`, то оба этих правила будут применены независимо друг от друга.

#### Примеры правил скрытия

* `example.com##.textad` — скроет элемент `div` с классом `textad` на домене `example.com` и всех его поддоменах.
* `example.com,example.org###adblock` - скроет элемент с атрибутом `id` равным `adblock` на доменах `example.com`, `example.org` и всех их поддоменах.
* `~example.com##.textad` - скроет элемент `div` с классом `textad` на всех доменах, кроме `example.com` и всех его поддоменах.

#### Исключения для правил скрытия

Существует специальный тип правил, "выключающий" отдельные правила скрытия на определенных доменах. Синтаксис таких правил очень похож на обычные правила скрытия, но вместо маркера `##` используется `#@#`.

Например, пусть в фильтре есть следующее правило.
```
##.textad
```

Если вы хотите отключить это правило для домена `example.com`, можно воспользоваться правилом-исключением.
```
example.com#@#.textad
```

Применять такие исключения рекомендуется только в случае, когда изменить само правило скрытия не представляется возможным. Во всех остальных случаях лучше изменить исходное правило, используя ограничение на домены.

> Правило-исключение без указания конкретных доменов не имеет смысла и будет проигнорировано.

### Правила CSS стилей

Иногда недостаточно просто скрыть какой-либо элемент, чтобы заблокировать рекламу. Например, блокировка рекламного элемента может просто сломать верстку сайта. Для таких случаев Adguard позволяет использовать гораздо более гибкие правила, чем обычные правила сокрытия. По сути, с помощью таких правил вы можете добавить на страницу любой CSS стиль.

#### Синтаксис правил CSS стилей

```
   rule = [domains] "#$#" selector "{" style "}"
domains = [domain0, domain1[, ...[, domainN]]]
```

* **`selector`** — [CSS селектор](https://developer.mozilla.org/ru/docs/Web/Guide/CSS/Getting_Started/Selectors), задающий элементы, которые должны быть скрыты.
* **`domains`** — ограничение на домены, на страницах которых будет применено правило. Строится по тем же правилам, что и в случае правил скрытия элементов.
* **`style`** — CSS стиль, который мы хотим применить к элементам, выбранным `selector`.

#### Примеры правил CSS стилей

```
example.com#$#body { background-color: #333!important; }
```

Это правило применит стиль `background-color: #333!important;` к элементу `body` для домена `example.com` и всех его поддоменов.

#### Исключения для правил CSS стилей

По аналогии с правилами скрытия, существует специальный тип правил, отключающий действие выбранного правила CSS стилей для определенных доменов. Синтаксис правил-исключения практически такой же, только маркер `#$#` заменяется на `#@$#`.

Например, пусть в фильтре есть следующее правило.
```
#$#.textad { visibility: hidden; }
```

Если вы хотите отключить это правило для домена `example.com`, можно воспользоваться правилом-исключением.
```
example.com#@$#.textad { visibility: hidden; }
```

Применять такие исключения рекомендуется только в случае, когда изменить само правило CSS стиля не представляется возможным. Во всех остальных случаях лучше изменить исходное правило, используя ограничение на домены.

### Расширенные CSS селекторы

Возможностей CSS 3.0 не всегда хватает для блокировки рекламы. Чтобы решить эту проблему, Adguard расширяет возможности CSS, добавляя поддержку новых псевдо-элементов. Для поддержки расширенных CSS селекторов нами был разработан отдельный модуль [с открытым кодом](https://github.com/AdguardTeam/ExtendedCss).

> В общедоступных фильтрах мы используем так называемый обратно-совместимый синтаксис. Дело в том, что использование расширенных псевдо-классов может помешать применению косметических правил в старых версиях Adguard или других блокировщиках рекламы, не поддерживающих расширенный CSS. Например, вместо псевдо-класса `:has(selector)` можно использовать атрибут `[-ext-has="selector"]`.

#### Псевдо-класс `:has()`

Спецификация CSS 4.0 описывает [псевдо-элемент `:has`](https://drafts.csswg.org/selectors/#relational). К сожалению, пока что он не поддерживается браузерами.

##### Синтаксис `:has()`

```
:has(selector)
```

Обратно-совместимый синтаксис:
```
[-ext-has="selector"]
```

Псевдо-класс `:has()` выбирает те элементы, внутри которых есть элементы, подходящие под `selector`.

##### Примеры `:has()`

###### Выбор всех элементов `div`, которые содержат элемент с классом `banner`.

**HTML code**
```
<div>Do not select this div</div>
<div>Select this div<span class="banner"></span></div>
```

**Селектор**
```
div:has(.banner)
```

Обратно-совместимый синтаксис:
```
div[-ext-has=".banner"]
```

#### Псевдо-класс `:contains()`

Принцип действия этого псевдо-класса очень прост: он позволяет выбрать элементы, которые содержат внутри заданный текст. Обратите внимание, что это должен быть именно текст, а не код. Для проверки используется свойство `innerText`.

##### Синтаксис `:contains()`

```
:contains(text)
```

Обратно-совместимый синтаксис:
```
[-ext-contains="text"]
```

Псевдо-класс `:contains()` выбирает те элементы, внутри которых есть текст `text`.

##### Примеры `:contains()`

###### Выбор всех элементов `div`, которые содержат текст `banner`.

**HTML code**
```
<div>Do not select this div</div>
<div id="selected">Select this div (banner)</div>
<div>Do not select this div <div class="banner"></div></div>
```

**Селектор**
```
div:contains(banner)
```

Обратно-совместимый синтаксис:
```
div[-ext-contains="banner"]
```

Обратите внимание, что в этом примере будет выбран только `div` с `id=selected`, так как следующий за ним элемент не содержит текст (`banner` является частью кода, а не текста).

#### Псевдо-класс `:matches-css()`

Этот псевдокласс позволяет выбрат элемент по его текущему свойству стиля.

##### Синтаксис `:matches-css()`

```
/* element style matching */
selector:matches-css(property-name ":" pattern)

/* ::before pseudo-element style matching */
selector:matches-css-before(property-name ":" pattern)

/* ::after pseudo-element style matching */
selector:matches-css-after(property-name ":" pattern)
```

Обратно-совместимый синтаксис:
```
selector[-ext-matches-css="property-name ":" pattern"]
selector[-ext-matches-css-after="property-name ":" pattern"]
selector[-ext-matches-css-before="property-name ":" pattern"]
```

###### `property-name`
Название CSS свойства, которое будет проверено у элемента.

###### `pattern`
Маска значения. Мы используем такой же синтаксис, как и в `pattern` в базовых правилах.

##### Примеры `:matches-css()`

###### Выбор элементов `div` с псевдо-элементом `::before` с указанным контентом.

**HTML code**
```
<style type="text/css">
    #to-be-blocked::before {
        content: "Block me"
    }
</style>
<div id="to-be-blocked" class="banner"></div>
<div id="not-to-be-blocked" class="banner"></div>
```

**Селектор**
```
div.banner:matches-css-before(content: block me)
```

Обратно-совместимый синтаксис:
```
div.banner[-ext-matches-css-before="content: block me"]
```

## Правила фильтрации HTML

В большинстве случаев достаточно базовых и косметических. Но иногда для фильтрации рекламы необходимо изменять HTML-код самой страницы до того, как он будет загружен самим браузером. Для того, чтобы сделать это применяются правила фильтрации HTML-контента. Они позволяют указать, какие HTML-элементы необходимо вырезать из страницы перед тем, как страница попадет в браузер.

> **Совместимость:** Adguard для Windows, Mac и Android (при установке "способа фильтрации" в "Качественный").
> Для работы правил фильтрации HTML необходима возможность модификации контента на сетевом уровне, поэтому этот тип правил невозможно реализовать в браузерном расширении.

### Синтаксис правил фильтрации HTML

```
      rule = [domains] "$$" tagName [attributes]
   domains = [domain0, domain1[, ...[, domainN]]]      
attributes = "[" name0 = value0 "]" "[" name1 = value2 "]" ... "[" nameN = valueN "]"
```

* **`tagName`** — имя элемента в нижнем регистре, например `div` или `script`.
* **`domains`** — ограничение на домены, на страницах которых будет применено правило. Строится по тем же правилам, что и в случае правил скрытия элементов.
* **`attributes`** — список атрибутов, ограничивающих выбор элементов. `name` - название атрибута, `value` - подстрока, которая содержится в значении атрибута.

### Примеры правил фильтрации HTML

**HTML code**
```
<script data-src="/banner.js"></script>
```

**Правило**
```
example.org$$script[data-src="banner"]
```

Это правило удалит из кода страниц все элементы `script` со значением `data-src`, содержащим подстроку `banner`. При этом правило будет работать только для домена `example.org` и всех его поддоменов.

### Специальные атрибуты

Помимо обычных атрибутов, значение которых проверяется у каждого элемента, существует набор специальных атрибутов правила, которые изменяют способ работы правила. Ниже мы перечислим все эти атрибуты.

##### `tag-content`

Пожалуй наиболее часто используемый специальный атрибут. Он ограничивает выбор теми элементами, внутренний HTML код которых (innerHTML) содержит указанную подстроку.

Например, рассмотрим такой HTML код:
```
<script type="text/javascript">
    document.write('<div>banner text</div>" />');
</script>
```

Следующее правило удалит со страницы все элементы `script`, код которых содержит подстроку `banner`.
```
$$script[tag-content="banner"]
```

> Если мы имеем дело с несколькими вложенными друг в друга элементами, каждый из которых подходит под одно и то же правило фильтрации HTML контента, то удалены будут все эти элементы.

##### `wildcard`

Этот специальный атрибут работает практически также как `tag-content` и позволяет проверить внутренний HTML-код элемента. Правило проверит, удовлетворяет ли HTML-код элемента заданному [шаблону поиска](https://ru.wikipedia.org/wiki/%D0%A8%D0%B0%D0%B1%D0%BB%D0%BE%D0%BD_%D0%BF%D0%BE%D0%B8%D1%81%D0%BA%D0%B0).

Возьмем для примера следующее правило:
`$$script[wildcard="*banner*text*"]`

Оно проверит, что код элемента содержит две последовательные подстроки `banner` и `text`.

##### `max-length`

Задает максимальную длину содержимого HTML-элемента. Если этот параметр задан, и длина содержимого превышает заданное значение — правило не применяется к элементу.

> Если этот параметр не задан, то `max-length` считается равным 8192.

Например, возьмем следующее правило:
```
$$div[tag-content="banner"][max-length="400"]
```

Это правило удалит все элементы `div`, код которых содержит подстроку `banner`, и длина которых не превышает `400` символов.

##### `min-length`

Задает минимальную длину содержимого HTML-элемента. Если этот параметр задан, и длина содержимого меньше заданного значения — правило не применяется к элементу.

Например, возьмем следующее правило:
```
$$div[tag-content="banner"][min-length="400"]
```

Это правило удалит все элементы `div`, код которых содержит подстроку `banner`, и длина которых превышает `400` символов.

##### `parent-elements`

Этот атрибут сильно изменяет поведение правила. Обычное правило фильтрации HTML с помощью указанных атрибутов находит и удаляет элемент страницы. В случае, если задан атрибут `parent-elements`, то удален будет не тот элемент, который был найден, а его родительский элемент с именем, заданным атрибутом `parent-elements`.

Рассмотрим пример.

**HTML code**
```
<table style="background: url('http://domain.com/banner.gif')">
    <tr>
        <td>
            <a href="http://example.org/ads">TEXT ADS</a>
        </td>
    </tr>
</table>
```

Проблема этого HTML-кода в том, что нам недостаточно вырезать ссылку на рекламу. Сам баннер показывается с помощью родительской таблицы (как ее `background`).  Тут нам и приходит на помощь атрибут `parent-elements`. 

Мы используем вот такое правило, чтобы заблокировать всю таблицу: 
```
$$a[href="example.org/ads"][parent-elements="table"]
```

Когда Adguard найдет на странице элемент `a` с атрибутом `href`, содержащим `example.org/ads`, то, вместо того, чтобы вырезать его, он будет искать ближайший родительский элемент `table`, и, если найдет — вырежет его.

Вы можете указать несколько искомых родительских элементов через запятую. Заблокирован будет ближайший.

##### `parent-search-level`

Задает максимальную глубину поиска родительского элемента. По умолчанию максимальная глубина поиска равна `3`. 
Это сделано для того, чтобы не вырезать лишнего, если `HTML` страницы поменяется. Не используйте слишком большие значения для этого атрибута.

### Исключения для правил фильтрации HTML

По аналогии с правилами скрытия, существует специальный тип правил, отключающий действие выбранного правила фильтрации HTML для определенных доменов. Синтаксис правил-исключения такой же, только маркер `$$` заменяется на `$@$`.

Например, пусть в фильтре есть следующее правило.
```
$$script[tag-content="banner"]
```

Если вы хотите отключить это правило для домена `example.com`, можно воспользоваться правилом-исключением.
```
example.com$@$script[tag-content="banner"]
```

## Javascript правила

Adguard поддерживает специальный тип правил, позволяющий вставить любой javascript-код на страницы интернет-сайтов.

> Обратите внимание, что этот тип правил может быть использован **только в доверенных фильтрах**. В эту категорию попадает ваш собственный **пользовательский фильтр**, и фильтры, которые создаются командой Adguard.

### Синтаксис javascript правил

```
rule = [domains]  "#%#" script
```

* **`domains`** — ограничение на домены, на страницах которых будет применено правило. Строится по тем же правилам, что и в случае правил скрытия элементов.
* **`script`** — произвольный javascript-код **в одну строку**.

### Примеры javascript правил

* `example.org#%#window.__gaq = undefined;` — выполняет код `window.__gaq = undefined;` на всех страницах сайта `example.org` и всех его поддоменов.

### Исключения для javascript правил

По аналогии с правилами скрытия, существует специальный тип правил, отключающий действие выбранного javascript правила для определенных доменов. Синтаксис правил-исключения такой же, только маркер `#%#` заменяется на `#@%#`.

Например, пусть в фильтре есть следующее правило.
```
#%#window.__gaq = undefined;
```

Если вы хотите отключить это правило для домена `example.com`, можно воспользоваться правилом-исключением.
```
example.com#@%#window.__gaq = undefined;
```