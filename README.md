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
* You read more than you code

---

> You need to write code that minimizes the time it would take someone else to understand it.
> Even if that someone is you.
>
> -- Dustin Boswell and Trevor Foucher

---

# Object calisthenics

* Programming exercises
* 9 Rules
* Originally written for Java (by Thoughtworks)

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
// Sonata\AdminBundle\Guesser\TypeGuesserChain
public function __construct(array $guessers)
{
    foreach ($guessers as $guesser) {
        if (!$guesser instanceof TypeGuesserInterface) {
            throw new UnexpectedTypeException(
                $guesser,
                'Sonata\AdminBundle\Guesser\TypeGuesserInterface'
            );
        }
        if ($guesser instanceof self) {
            $this->guessers = array_merge(
                $this->guessers,
                $guesser->guessers
            );
        } else {
            $this->guessers[] = $guesser;
        }
    }
}
````

---

## Possible techniques

* Extract method
* Extract class
* Early returns

---

````php
// Sonata\AdminBundle\Guesser\TypeGuesserChain
public function __construct(array $guessers)
{
    foreach ($guessers as $guesser) {
        if (!$guesser instanceof TypeGuesserInterface) {
            throw new UnexpectedTypeException(
                $guesser,
                'Sonata\AdminBundle\Guesser\TypeGuesserInterface'
            );
        }
        if ($guesser instanceof self) {
            $this->guessers = array_merge(
                $this->guessers,
                $guesser->guessers
            );
        } else {
            $this->guessers[] = $guesser;
        }
    }
}
````

---

````php
// Sonata\AdminBundle\Guesser\TypeGuesserChain
public function __construct(array $guessers)
{
    foreach ($guessers as $guesser) {
        $this->addGuesser($guesser);
    }
}

private function addGuesser($guesser)
{
    if (!$guesser instanceof TypeGuesserInterface) {
        throw new UnexpectedTypeException(
            $guesser,
            'Sonata\AdminBundle\Guesser\TypeGuesserInterface'
        );
    }
    if ($guesser instanceof self) {
        $this->guessers = array_merge(
            $this->guessers,
            $guesser->guessers
        );
    } else {
        $this->guessers[] = $guesser;
    }
}
````

---

## Note

Ternary operators count as an indentation level!

PSR2 shows this hidden extra depth

---

# 2. Don't use the else keyword

* Readability
* Less cyclomatic complexity
* Less duplication

---

````php
// Sonata\AdminBundle\Datagrid\Pager
public function setCursor($pos)
{
    if ($pos < 1) {
        $this->cursor = 1;
    } else {
        if ($pos > $this->nbResults) {
            $this->cursor = $this->nbResults;
        } else {
            $this->cursor = $pos;
        }
    }
}
````

---

## Techniques:

* Early returns
* Switch statements
* Strategy pattern

---

````php
// Sonata\AdminBundle\Datagrid\Pager
public function setCursor($pos)
{
    if ($pos < 1) {
        $this->cursor = 1;
    } else {
        if ($pos > $this->nbResults) {
            $this->cursor = $this->nbResults;
        } else {
            $this->cursor = $pos;
        }
    }
}
````

---

````php
// Sonata\AdminBundle\Datagrid\Pager
public function setCursor($pos)
{
    if ($pos < 1) {
        $this->cursor = 1;
        return;
    }

    if ($pos > $this->nbResults) {
        $this->cursor = $this->nbResults;
    } else {
        $this->cursor = $pos;
    }
}
````

---

````php
// Sonata\AdminBundle\Datagrid\Pager
public function setCursor($pos)
{
    if ($pos < 1) {
        $this->cursor = 1;
        return;
    }

    if ($pos > $this->nbResults) {
        $this->cursor = $this->nbResults;
        return;
    }

    $this->cursor = $pos;
}
````

---

# 3. Wrap all primitives and strings

* Type hinting
* Encapsulation
* Enforcing a contract
* More clear method signatures

---

````php
// FOS\UserBundle\Util\UserManipulator
/**
 * Changes the password for the given user.
 *
 * @param string $username
 * @param string $password
 */
public function changePassword($username, $password)
{
    $user = $this->findUserByUsernameOrThrowException($username);
    $user->setPlainPassword($password);
    $this->userManager->updateUser($user);
}
````

---

## Techniques

* Extract class
* Value objects

---

````php
// FOS\UserBundle\Util\UserManipulator
/**
 * Changes the password for the given user.
 *
 * @param string $username
 * @param string $password
 */
public function changePassword($username, $password)
{
    $user = $this->findUserByUsernameOrThrowException($username);
    $user->setPlainPassword($password);
    $this->userManager->updateUser($user);
}
````

---

````php
final class Username
{
    private $username;

    public function __construct($username)
    {
        $this->validateUsername($username);
        $this->username = $username;
    }
}
````

````php
final class PlainPassword
{
    private $password;

    public function __construct($password)
    {
        $this->validatePassword($pasword);
        $this->password = $password;
    }

    public function hash();
}
````

---

````php
// FOS\UserBundle\Util\UserManipulator
/**
 * Changes the password for the given user.
 *
 * @param Username $username
 * @param PlainPassword $password
 */
public function changePassword(
    Username $username,
    PlainPassword $password
) {
    $user = $this->findUserByUsernameOrThrowException($username);
    $user->setPlainPassword($password);
    $this->userManager->updateUser($user);
}
````

---

# 4. First class collection

* Cleaner code
* Easier to merge collections

---

## Techniques

* use Doctrine Collections
* Write collection like repository classes

---

### Doctrine\Common\Collections\Collection

* implements Countable
* implements IteratorAggregate
* implements ArrayAccess

---

````php
interface UserRepository
{
    public function add(User $user);
    public function remove(User $user);

    /** @return User */
    public function find(UserId $id);
}
````

---

````php
class DoctrineUserRepository extends EntityRepository
{
    public function add(User $user)
    {
        $this->getEntityManager()->persist($user);
        $this->getEntityManager()->flush();
    }

    public function remove(User $user)
    {
        $this->getEntityManager()->remove($user);
        $this->getEntityManager()->flush();
    }

    public function find(UserId $id)
    {
        return $this->findOneById($id->getId());
    }
}
````

---

# 5. One `->` per line

* Easier to debug
* Readability

Does not apply to fluent interfaces or objects using the method chaining pattern. (f.e. QueryBuilder)

---

## = Law of Demeter

* Each unit should have only limited knowledge about other units: only units "closely" related to the current unit.
* Each unit should only talk to its friends; don't talk to strangers.
* "Only talk to your immediate friends."

> Never trust a functions return value.

---

````php
class Order
{
    public function changeOrderStatus(
        OrderStatus $status,
        Customer $customer
    ) {
        $this->status = $status;

        if ($status === OrderStatus::PAID) {
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

## Techniques

* Encapsulate funcionality in other places
* Use atomic actions
* Move your code to the right place

---

````php
class Order
{
    public function changeOrderStatus(
        OrderStatus $status,
        Customer $customer
    ) {
        $this->status = $status;

        if ($status === OrderStatus::PAID) {
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
    public function changeOrderStatus(
        OrderStatus $status,
        Customer $customer
    ) {
        $this->status = $status;

        if ($status === OrderStatus::PAID) {
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
    public function setAddress(Address $adress) {}
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



---

````php
class User
{
    public function setEmail($email) {}
    public function setAddress(Address $adress) {}
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
    public function relocateTo(Address $adress) {}
}

class Address
{
    private function __construct($street, $number, $zip, $city) {}
    public static function fromString($string) {}
}


$user = new User('wouter@sumocoders.be');
$user->relocateTo(
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
<https://github.com/TheLadders/object-calisthenics>
