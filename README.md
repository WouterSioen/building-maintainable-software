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
* Value Objects

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

# 4. First class collection

* Cleaner code
* Easier to merge collections

---

# 5. One dot per line

* Easier to debug
* Readability

---

# 6. Don't abbreviate

* Readability

---

# 7. Keep all classes small

* Single responsibility principle
* Clear objective and methods

---

# 8. No classes with more than five instance variables

* Easier to mock in unit tests
* Shorter dependency list

---

# 9. No getters/setters

* Open closed principle
* Single responsibility principle

---

## Questions?

---

## Thank you!

<https://joind.in/talk/view/14182>

![Thanks](img/thanks.png)

---

## Resources

http://symfony.com/doc/current/index.html

https://speakerdeck.com/ronnylt/dic-to-the-limit-desymfonyday-barcelona-2014

https://lostechies.com/derickbailey/2009/02/11/solid-development-principles-in-motivational-pictures/

http://blog.servergrove.com/2013/10/23/symfony2-components-overview-eventdispatcher/
