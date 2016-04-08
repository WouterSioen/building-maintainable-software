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
* Easy efforts
* Class design
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

# Easy efforts

### choose a code style

* PSR-2 is most used
* Can be enforced using php code sniffer
* php-cs-fixer can autofix (some) issues
* use pre-commit hooks and ci to enforce it

---

# Easy efforts

### Do code reviews

* Keep pull requests small for good reviews
* Review more than code (commit messages, documentation,...)
* Ask an extra opinion if needed
* Don't only give negative feedback
* Automate automatable stuff
* Always review your own code

???

Doing code reviews is really important because you know too much about your own code.
You should also (if possible) try to assign code reviews to different people.
This will make it easier for you to get different insights in your code.

---

# Refactoring

* Should be an integrated part of your workflow
* Automated tests can help you validate refactorings
* Look out for duplication (phpcpd)
* Static analysis can help you here

---

# Setup

* Make your project easy to set up (locally)
  * use vagrant/docker
  * add a setup guide if needed (README)
  * make code (without interface) "runnable" through tests
* Make your project easy to deploy
  * automate deployment
  * keep your master/default branch stable

---

# SOLID

Keep the SOLID rules in mind.

<https://leanpub.com/principles-of-package-design>

---

## Refactorings:

<https://github.com/WouterSioen/sir-trevor-php/tree/v1.0>

* Editor: <http://madebymany.github.io/sir-trevor-js/example.html>
* Convert output to html
* Convert html to output

---

## Library Structure:

```
▾ src/Sioen/
  ▾ Tests/
      ConverterTest.php
  ▾ Types/
      BaseConverter.php
      BlockquoteConverter.php
      ConverterInterface.php
      HeadingConverter.php
      IframeConverter.php
      ImageConverter.php
      ListConverter.php
      ParagraphConverter.php
    Converter.php
    ToHtmlContext.php
    ToJsonContext.php
```

---

<iframe src="https://www.youtube.com/embed/Qqj9oRfP6gY?vq=hd720&rel=0&showinfo=0" frameborder="0" allowfullscreen></iframe>

???

Our library has one entry point: the Converter class. This class has two
responsibilities: converting from html to json and the other way around.

We'll split the two classes to make sure each class does one thing and one
thing only.

---

<iframe src="https://www.youtube.com/embed/lXFIoUzRxMI?vq=hd720&rel=0&showinfo=0" frameborder="0" allowfullscreen></iframe>

???

Note that I've removed the "Converter" suffix from the two main classes. The
small "type" classes that convert one small item are also called converters.
The names "HtmlToJson" and "JsonToHtml" are explicit enough.

This refactoring will remove the "ToHtmlContext" and "ToJsonContext". These
classes were only used from the main entry points. They are instantiated
in there so they are coupled. All the small type converters are thus also
coupled. This refactoring makes this more explicit making it easier to
move to more decoupled code later.

---

<iframe src="https://www.youtube.com/embed/KccB3jIRE88?vq=hd720&rel=0&showinfo=0" frameborder="0" allowfullscreen></iframe>

???

We see that our classes always only use one method of our interface. We'll
split the interface into a ToJson and ToHtml version to follow the
Interface Segregation Principle.

---

<iframe src="https://www.youtube.com/embed/quLBHjEHJzg?vq=hd720&rel=0&showinfo=0" frameborder="0" allowfullscreen></iframe>

???

I've already done some refactoring before we'll go to this step. I've
split the implementation of all converters to a toJson and toHtml version.

I've also gave them a "match" method, so the main entry point classes don't
have to decide which class to use anymore. This gives it one less responsibility.

In this refactoring, we're going to introduce an "addConverter" method
which makes our class open for extension. This way, other persons can add
extra custom converters.

Note that we're not following the open closed principle yet. It's open for
extension, but not closed to modification, since we'll need to access the class
to change the order of converters or to remove a default one.

---

<iframe src="https://www.youtube.com/embed/3w7cNMn7duc?vq=hd720&rel=0&showinfo=0" frameborder="0" allowfullscreen></iframe>

???

In this refactoring, we'll fully remove the dependencies on implementations.
We're now following the dependency inversion principle: we're depending on
abstractions, not on implementation.

Our class is also closed for modification. We don't need to enter the class to
get the behaviour we want.

Note that I've made all classes final later, to avoid extending them. If
classes can be extended, they can still be modified. I want my class to be
closed to modifications, since it already has an extension point.

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

## Possible techniques

* Extract method
* Extract class
* Early returns

---

<iframe src="https://www.youtube.com/embed/4e_uJRVxmTE?vq=hd720&rel=0&showinfo=0" frameborder="0" allowfullscreen></iframe>

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

## Techniques:

* Early returns
* Switch statements
* Strategy pattern

---

<iframe src="https://www.youtube.com/embed/A_MLFW3YyP4?vq=hd720&rel=0&showinfo=0" frameborder="0" allowfullscreen></iframe>

---

# 3. Wrap all primitives and strings

* Type hinting
* Encapsulation
* Enforcing a contract
* More clear method signatures

---

````php
return array(
    'type' => 'text',
    'data' => array(
        'text' => ' ' . $this->htmlToMarkdown($html)
    )
);
````

???

Do you know what this is? What this code represents? Can you ensure this is
correct without a lot of cecks?

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
return new SirTrevorBlock(
    'text',
    array('text' => ' ' . $this->htmlToMarkdown($html))
);
````

---

# 4. First class collection

* Cleaner code
* Gives behaviour of collections a place

---

## Techniques

* Extract method
* Use a collection library (doctrine)

---

````php
final class HtmlToJson
{
    /** @var array */
    private $converters = array();

    public function addConverter(Converter $converter)
    {
        $this->converters[] = $converter;
    }

    public function toJson($html);
}
````

???

I have a violation of this rule in my codebase, but I won't change this.
If I would change this class to use a Collection (fe. a ConverterCollection),
I'd need to depend upon implementations instead of on abstractions. Thats the
only way I can enforce that only the right type of converters are available in
my ConverterCollection.

It would also require a lot more code to get me the same result.

This shows clearly that these "rules" are in fact guidelines. I think the SOLID
rules should be applied more strictly than the object calisthenics. Don't let
object calisthenics mess up your decoupling.

---

# 5. One `->` per line

* Easier to debug
* Readability
* Testability

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

        if ((string) $status === OrderStatus::PAID) {
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

        if ((string) $status === OrderStatus::PAID) {
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

        if ((string) $status === OrderStatus::PAID) {
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

> Tell, don't ask.

???

Note that getters can be ok to get the state of the object, as long as you don't
use it to decide stuff about that object outside of it.

Decisions based on the state of the object should be done inside of it.

---

````php
class User
{
    public function setId($id) {}
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
$user->setId(1);
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
    public function __construct($id, $email) {}
    public function relocateTo(Address $adress) {}
}

class Address
{
    private function __construct($street, $number, $zip, $city) {}
    public static function fromString($string) {}
}


$user = new User(1, 'wouter@sumocoders.be');
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
