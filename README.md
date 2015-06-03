# Building maintainable software

---

## Hi, I'm Wouter

![Sumo Wouter](img/Sumo_Wouter.png)

:twitter: [@WouterSioen](http://twitter.com/WouterSioen)

:github: [WouterSioen](http://github.com/WouterSioen)

---

## I work at Sumocoders

---

![I mainly use Symfony](img/symfony.png)

---

# I'm a Fork core developer

---

## Topics

* What's maintainable software
* Why
* Object Calisthenics

---

# Maintainable software?

Maintainable software allows __us__ to __quickly__:

* Fix a bug
* Add new features
* Improve usability
* Increase performance
* Make a fix that prevents a bug from occurring in future

---

# Why

* Strong timeframes
* Developers leaving teams
* You read more then you code

---

> You need to write code that minimizes the time it would take someone els to understand it.
> Event if that someone is you.
>
> -- Dustin Boswell and Trevor Foucher

---

# Object calisthenics

* Programming exercises
* 9 Rules
* Originally written for Java

---

# Disclamer

* Guidelines, not rules
* I don't always follow them myself

---

# 1. Only one indentation level per method

* Single responsibility principle
* reusable code

---

````php
private function handleUrls($text, $filter = '')
{
    preg_match_all(
        '/<img.*src="(.*)".*\/>/Ui', $text, $matchesImages
    );

    if (isset($matchesImages[1]) && !empty($matchesImages[1])) {
        foreach ($matchesImages[1] as $key => $file) {
            if (!empty($filter) && !stristr($file, $filter)) {
                continue;
            }

            $noSize = preg_replace('/\-\d+x\d+/i', '', $file);
            if (isset($this->attachments[strtolower($file)])) {
                $text = str_replace($file, $this->attachments[strtolower($file)], $text);
            } elseif (isset($this->attachments[strtolower($noSize)])) {
                $text = str_replace($file, $this->attachments[strtolower($noSize)], $text);
            }
        }
    }

    return $text;
}
````

---

````php
private function handleUrls($text, $filter = '')
{
    preg_match_all(
        '/<img.*src="(.*)".*\/>/Ui', $text, $matchesImages
    );

    if (isset($matchesImages[1]) && !empty($matchesImages[1])) {
        foreach ($matchesImages[1] as $file) {
            if (!empty($filter) && !stristr($file, $filter)) {
                continue;
            }

            $noSize = preg_replace('/\-\d+x\d+/i', '', $file);
            if (isset($this->attachments[strtolower($file)])) {
                $text = str_replace($file, $this->attachments[strtolower($file)], $text);
            } elseif (isset($this->attachments[strtolower($noSize)])) {
                $text = str_replace($file, $this->attachments[strtolower($noSize)], $text);
            }
        }
    }

    return $text;
}
````

---

````php
private function handleUrls($text, $filter = '')
{
    preg_match_all(
        '/<img.*src="(.*)".*\/>/Ui', $text, $matchesImages
    );

    if (!isset($matchesImages[1]) || empty($matchesImages[1])) {
        return $text;
    }

    foreach ($matchesImages[1] as $file) {
        if (!empty($filter) && !stristr($file, $filter)) {
            continue;
        }

        $noSize = preg_replace('/\-\d+x\d+/i', '', $file);
        if (isset($this->attachments[strtolower($file)])) {
            $text = str_replace($file, $this->attachments[strtolower($file)], $text);
        } elseif (isset($this->attachments[strtolower($noSize)])) {
            $text = str_replace($file, $this->attachments[strtolower($noSize)], $text);
        }
    }

    return $text;
}
````

---

````php
private function handleUrls($text, $filter = '')
{
    preg_match_all(
        '/<img.*src="(.*)".*\/>/Ui', $text, $matchesImages
    );

    if (!isset($matchesImages[1]) || empty($matchesImages[1]) || empty($filter)) {
        return $text;
    }

    foreach ($matchesImages[1] as $file) {
        if (!stristr($file, $filter)) {
            continue;
        }

        $noSize = preg_replace('/\-\d+x\d+/i', '', $file);
        if (isset($this->attachments[strtolower($file)])) {
            $text = str_replace($file, $this->attachments[strtolower($file)], $text);
        } elseif (isset($this->attachments[strtolower($noSize)])) {
            $text = str_replace($file, $this->attachments[strtolower($noSize)], $text);
        }
    }

    return $text;
}
````

---

````php
private function handleUrls($text, $filter = '')
{
    preg_match_all(
        '/<img.*src="(.*)".*\/>/Ui', $text, $matchesImages
    );

    if (!isset($matchesImages[1]) || empty($matchesImages[1]) || empty($filter)) {
        return $text;
    }

    foreach ($matchesImages[1] as $file) {
        $text = $this->replaceImage($text, $file, $filter);
    }

    return $text;
}
````

---

````php
private function replaceImage($text, $file, $filter = '')
{
    if (!stristr($file, $filter)) {
        return $text
    }

    $noSize = preg_replace('/\-\d+x\d+/i', '', $file);
    if (isset($this->attachments[strtolower($file)])) {
        $text = str_replace(
            $file,
            $this->attachments[strtolower($file)],
            $text
        );
    } elseif (isset($this->attachments[strtolower($noSize)])) {
        $text = str_replace(
            $file,
            $this->attachments[strtolower($noSize)],
            $text
        );
    }
}
````

---

# 2. Don't use the else keyword

* Readability
* Less cyclomatic complexity
* Less duplication

---

````php
public function execute()
{
    $this->id = $this->getParameter('id', 'int');

    if (
        $this->id !== null
        && BackendContentBlocksModel::exists($this->id)
    ) {
        $this->record = BackendContentBlocksModel::get($this->id);
        BackendContentBlocksModel::delete($this->id);
        $this->redirect(
            BackendModel::createURLForAction('Index')
            . '&report=deleted
        );
    } else {
        $this->redirect(
            BackendModel::createURLForAction('Index')
            . '&error=non-existing'
        );
    }
}
````

---

````php
public function execute()
{
    $this->id = $this->getParameter('id', 'int');

    if (
        $this->id === null
        || !BackendContentBlocksModel::exists($this->id)
    ) {
        return $this->redirect(
            BackendModel::createURLForAction('Index')
            . '&error=non-existing'
        );
    }

    $this->record = BackendContentBlocksModel::get($this->id);
    BackendContentBlocksModel::delete($this->id);
    $this->redirect(
        BackendModel::createURLForAction('Index')
        . '&report=deleted
    );
}
````

---

# 3. Wrap all primitives and strings

* Type hinting
* Encapsulation
* Enforcing a contract

---

````php
$message = 'Dat we een paar collega's zoeken:'
. ' http://sumocoders.be/vacatures';

if (strlen($message) > 140) {
    throw new InvalidArgumentException('Too long');
}

$tweet = array(
    'message' => $message,
    'id' => '583599688252837888',
    'author' => 'sumocoders',
);
````

---

````php
class Tweet
{
    public function __construct($id, $message, $author)
    {
        if (strlen($message) > 140) {
            throw new InvalidArgumentException('Too long');
        }
    }
}
````

````php
$message = 'Dat we een paar collega's zoeken:'
. ' http://sumocoders.be/vacatures';

$tweet = new Tweet(
    '583599688252837888',
    $message,
    'sumocoders'
);
````

---

# 4. First class collection

* Cleaner code
* Easier to merge collections

---

### Doctrine\Common\Collections\Collection

* implements Countable
* implements IteratorAggregate
* implements ArrayAccess

---

# 5. One `->` per line

* Easier to debug
* Readability

---

# = Law of Demeter

* Each unit should have only limited knowledge about other units: only units "closely" related to the current unit.
* Each unit should only talk to its friends; don't talk to strangers.
* Only talk to your immediate friends.

---

````php
class Order
{
    public function changeOrderStatus($status, Customer $customer)
    {
        $this->status = $status;

        if ($status == self::PAID) {
            $this->sendEmail(
                $customer
                    ->getData()
                    ->getContactInformation()
                    ->getEmail(),
                'Your order has been paid'
            );
        }
    }
}
````

---

````php
class Order
{
    public function changeOrderStatus($status, Customer $customer)
    {
        $this->status = $status;

        if ($status == self::PAID) {
            $customer->sendEmail('Your order has been paid');
        }
    }
}
````

---

# 6. Don't abbreviate

* Readability

---

````php
$db = $this->get('database');

$tpl = new Template(false);

$this->frm = new Form('edit');

foreach ($translation as $module => $t) {}

private static $err = array();

$qPos = strpos($language, 'q=');

$tmp = '';

$errStr = '';

$aTemp = array();
````

---

# 7. Keep all classes small

* Single responsibility principle
* Clear objective and methods

---

````php
class Navigation
{
    public static function getBackendURLForBlock();
    public static function getFirstChildId();
    public static function getFooterLinks();
    public static function getKeys();
    public static function getNavigation();
    public static function getNavigationHTML();
    public static function getPageId();
    public static function getPageInfo();
    public static function getURL();
    public static function getURLForBlock();
    public static function getURLForExtraId();
    public static function setExcludedPageIds();
    public static function setSelectedPageIds();
    public static function getURL();
    public static function getURLForBlock();
}
````

---

````php
class Router
{
    public static function getBackendURLForBlock();
    public static function getURL();
    public static function getURLForBlock();
    public static function getURLForExtraId();
}

class Page
{
    public static function getFirstChildId();
    public static function getPageId();
    public static function getPageInfo();
}

class Navigation
{
    public static function getFooterLinks();
    public static function getKeys();
    public static function getNavigation();
    public static function getNavigationHTML();
    public static function setExcludedPageIds();
    public static function setSelectedPageIds();
}
````

---

# 8. No classes with more than five instance variables

* Easier to mock in unit tests
* Shorter dependency list

---

````php
class Widget extends \KernelLoader
{
    private $column = 'left';
    protected $header;
    private $position;
    protected $rights = array();
    private $templatePath;
    public $tpl;
}
````

---

````php
class Widget extends \KernelLoader
{
    // only the object parsing the widget should know these
    // private $column = 'left';
    // private $position;
    // protected $rights = array();

    protected $header;
    private $templatePath;
    public $tpl;
}
````

---

# 9. No getters/setters

* Open closed principle
* Single responsibility principle

---

````php
class User
{
    public function setEmail($email) {}
}

class Address
{
    public function setStreet($street) {}
    public function setNumber($number) {}
    public function setPostalCode($postalcode) {}
    public function setCity($city) {}
}


$user = new User();
$user->setEmail('wouter@sumocoders.be');

$address = new Address();
$address->setStreet('Afrikalaan');
$address->setNumber(289);
$address->setPostalCode(9000);
$address->setCity('Gent');

$user->setAddress($address);
````

---

````php
class User
{
    public function __construct($email) {}
}

class Address
{
    private function __construct($street, $number, $zip, $city) {}
    public static function fromString($string) {}
}


$user = new User('wouter@sumocoders.be');
$user->relocate(
    Address::fromString('Afrikalaan 289, 9000 Gent')
);
````

---

## Questions?

---

## Thank you!

![Thanks](img/thanks.png)

---

## Resources

<http://software.ac.uk/resources/guides/developing-maintainable-software>
<http://williamdurand.fr/2013/06/03/object-calisthenics/>
<http://en.wikipedia.org/wiki/Law_of_Demeter>
<http://www.slideshare.net/rdohms/your-code-sucks-lets-fix-it-15471808>
<http://www.slideshare.net/guilhermeblanco/object-calisthenics-applied-to-php>
