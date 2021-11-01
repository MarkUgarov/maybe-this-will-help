# Combining JUnit, Mockito and JMockit

## Is this really a good idea?

**No!**

## Why should I do that?

Sometimes you need to. E.g. if your project is a mash of singleton-pattern "Manager"-classes and you can't update your
Mockito-Version (because of whatever reason).

I found myself in that situation more than once. It' not nice. But you can work with that.

Also in some cases you haven't really the option to simply rewrite everything. So you do your best.

## How to work with JMockit

See [this page](jmockit.md) and maybe see [this project](https://github.com/MarkUgarov/TestProjectToTest) to compare.

## How to work wiht Mockito

See [this page](mockito.md) and maybe see [this project](https://github.com/MarkUgarov/TestProjectToTest) to compare.