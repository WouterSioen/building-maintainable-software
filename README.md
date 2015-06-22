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

???

* maintainability
* readability
* testability
* comprehensibility

---

# Disclamer

* Guidelines, not rules
* I don't always follow them myself
* Some of them are controversial

???

First made as an exercise. In the exercise, you would apply all rules strictly on
a rather small project (±1000 lines of java).

The rules should change your approach in designing software.

It's good to note that it's really easy to get big discussions over these rules.

---

# 1. Only one indentation level per method

* Single responsibility principle
* reusable code
* readable code

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

???

To understand this code, you need to get your brain around it. It takes some
reading and thinking to fully understand all use cases in this method.

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

???

Let's apply an extract method refactoring. The constructor is a lot more readable
and can be understood in one glimpse. If you want to know how "adding a guesser"
works, you can still go check in the private method (that could be refactored further).

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

???

If you don't see the body of the method, do you know if the password has to
be hashed before calling the method?

---

## Techniques

* Extract class
* Value objects

---

## Value objects

* From the DDD World
* Immutable objects
* No identity
* Equal content == Equal object
* f.e. Money, DateRange, GeoLocation

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

???

We clearly show that our password is not encoded yet. We can also encapsulate
the hash method in this object, since only PlainPassword objects will need to be
hashed. This method can return f.e. a new HashedPassword object. You'll only need
instanceof checks to know if your password is hashed or not, and calling one method
will do the conversion.

Also note the final keyword. This is needed to avoid other objects extending
your value objects and being able to mess with the encapsulated data.

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
* Gives behaviour of collections a place

---

## Techniques

* use Doctrine Collections
* Write collection like repository classes

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

* Move your code to the right place
* Think about the responsibilities

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
* Comprehensibility

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

## Techniques

* Rename property
* Rename local variable

---

## Bonus: don't use meaningless names

````php
// FOS\RestBundle\EventListener\ParamFetcherListener
private function getAttributeName(array $controller)
{
    list($object, $name) = $controller;
    $method = new \ReflectionMethod($object, $name);
    foreach ($method->getParameters() as $param) {
        if ($this->isParamFetcherType($param)) {
            return $param->getName();
        }
    }

    // If there is no typehint, inject the ParamFetcher using a default name.
    return 'paramFetcher';
}
````

---

# 7. Keep all classes small

* Single responsibility principle
* Clear objective and methods
* Readibility
* Comprehensibility
* Rule = 50 lines. up to 150 is ok.

---

````php
//  JMS\I18nRoutingBundle\Router\I18Router: 251 lines
class I18nRouter extends Router
{
    public function __construct();
    public function setLocaleResolver(LocaleResolverInterface $resolver);
    public function setRedirectToHost($bool);
    public function setHostMap(array $hostMap);
    public function setI18nLoaderId($id);
    public function setDefaultLocale($locale);
    public function match($url);
    public function getRouteCollection();
    public function getOriginalRouteCollection();
    public function matchRequest(Request $request);

    // 45 lines, 216 code paths
    public function generate($name, $parameters = array(), $absolute = false);

    // 93 lines, 2680 code paths
    private function matchI18n(array $params, $url);
}
````

---

## Techniques

* Extract class
* Use value objects

---

````php
//  JMS\I18nRoutingBundle\Router\I18Router: 251 lines
class I18nRouter extends Router
{
    public function __construct();
    public function setLocaleResolver(LocaleResolverInterface $resolver);
    public function setRedirectToHost($bool);
    public function setHostMap(array $hostMap);
    public function setI18nLoaderId($id);
    public function setDefaultLocale($locale);
    public function match($url);
    public function getRouteCollection();
    public function getOriginalRouteCollection();
    public function matchRequest(Request $request);

    // 45 lines, 216 code paths
    public function generate($name, $parameters = array(), $absolute = false);

    // 93 lines, 2680 code paths
    private function matchI18n(array $params, $url);
}
````

---

````php
// &plusmn; 120 lines
class I18nRouter extends Router
{
    public function __construct();
    public function setLocaleResolver(LocaleResolverInterface $resolver);
    public function setRedirectToHost($bool);
    public function setHostMap(array $hostMap);
    public function setI18nLoaderId($id);
    public function setDefaultLocale($locale);
    public function match($url);
    public function getRouteCollection();
    public function getOriginalRouteCollection();
    public function matchRequest(Request $request);

    public function generate($name, $parameters = array(), $absolute = false);
    private function matchI18n(array $params, $url);
}

// &plusmn; 50 lines
class I18UrlGenerator {}

// &plusmn; 100 lines
class I18RouteMatcher {}
````

---

# 8. Max 5 instance variables

* Low cohesion
* Better encapsulation
* Easier to mock in unit tests
* Shorter dependency list
* Comprehensibility

???

In pure object calisthenics, the rule is only two instance variables.
This gives you two kinds of classes:

* Classes that maintain the state of one single instance variable
* Classes that coördinate two variables

It really forces you to decouple

---

````php
// FOS\RestBundle\View\ViewHandler
class ViewHandler extends ContainerAware implements ConfigurableViewHandlerInterface
{
    protected $customHandlers = array();
    protected $formats;
    protected $failedValidationCode;
    protected $emptyContentCode;
    protected $serializeNull;
    protected $forceRedirects;
    protected $defaultEngine;
    protected $exclusionStrategyGroups = array();
    protected $exclusionStrategyVersion;
    protected $serializeNullStrategy;
}
````

---

## Techniques

* Extract class
* Composition

Think about the responsibilities

---

# 9. No getters/setters

* Open closed principle
* Single responsibility principle
* Intention revealing interfaces

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
<http://c2.com/cgi/wiki?AccessorsAreEvil>
