# Combining JUnit, Mockito and JMockit

## Is this really a good idea?

**No!**

## Why should I do that?

Sometimes you need to. E.g. if your project is a mash of singleton-pattern "Manager"-classes and you can't update your
Mockito-Version (because of whatever reason).

I found myself in that situation more than once. It' not nice. But you can work with that.

Also in some cases you haven't really the option to simply rewrite everything. So you do your best.

## Example classes

In this fiction, your team somehow endet up to program in singleton patern and likes static methods. There is nothing
you can do about.

I wrote two little classes: a `Controller` and a `Service`. Assume we are in an early stage of the project and just want
to make first steps by implementing the `Controller` without having the full functionality of the backend. In this
szenario, the `Controller` holds the "logic" while the `Service` is supposed to provide the rest of the "functionality"
later on.

**The process:**

The `Controller` checks if the database is available (via the `Service`).

If so, everything is nice.

If not, the `Controller` calls the `Service` again, evaluates current user and Time and sends an E-Mail.

The Service is (obviously) just a prototype. But we can already test our Controller without implementing everything
else.

```java
package example.controller;

import example.services.SingletonService;

public class Controller {

    private final SingletonService singletonService;

    public Controller() {
        singletonService = SingletonService.getInstance();
    }

    public void callService() {
        if (!singletonService.checkDataBaseAvailable()) { // calling the instance method
            SingletonService.sendAlertMail( // calling the class method
                    SingletonService.getCurrentUserName(),  // calling the class method
                    singletonService.giveCurrentLocalDateTime() // calling the instance method
            );
        }
    }
}
```

```java
package example.services;

import java.time.LocalDateTime;

public class SingletonService {

    private static SingletonService instance;

    public static SingletonService getInstance() {
        if (instance == null) {
            instance = new SingletonService();
        }
        return instance;
    }

    private SingletonService() {

    }

    public boolean checkDataBaseAvailable() {
        return false;
    }

    public LocalDateTime giveCurrentLocalDateTime() {
        return null;
    }

    public static String getCurrentUserName() {
        return null;
    }

    public static void sendAlertMail(String username, LocalDateTime localDateTime) {
        // do something else here
    }
}

```

## How to work with JMockit

You will have to do some setup first. For the maven pom (who does everything you need if you work with maven) see
[the pom of this shitty project I wrote](https://github.com/MarkUgarov/TestProjectToTest/blob/master/pom.xml).

Hint:
***The following file may not be shown correctly. Please see the block bellow for the expectations. (This is a problem
with git pages.)***

```java
package example.controller;

import example.services.SingletonService;
import mockit.Expectations;
import mockit.MockUp;
import mockit.Mocked;
import org.junit.Before;
import org.junit.Test;

import java.time.LocalDateTime;

/**
 * Test of class {@link Controller}
 * <p>
 * Created by mgar on 15.10.2021.
 */
public class ControllerTest {

    private Controller controller;

    @Mocked
    private SingletonService singletonService;

    @Before
    public void setUp() throws Exception {
        controller = new Controller();
    }

    @Test
    public void callService_shouldSendAlertMail_ifNotAvailable() {
        // mocking a static method with MockUp<>, just to give you an example
        new MockUp<SingletonService>() {
            @mockit.Mock
            public String getCurrentUserName() {
                return "Luke";
            }
        };
        
        new Expectations() {{
            // mocking the instance methods
            singletonService.checkDataBaseAvailable();
            minTimes = 1;
            result = false;
            singletonService.giveCurrentLocalDateTime();
            minTimes = 1;
            // mocking another static method
            LocalDateTime expectedLocalDateTime = LocalDateTime.of(2020, 10, 15, 17, 30, 0);
            result = expectedLocalDateTime;
            SingletonService.sendAlertMail("Luke", expectedLocalDateTime);
            minTimes = 1;
        }};

        controller.callService();
    }
}
```

with an anonymous class extending mockit.Expectations: 

```
new Expectations(){{
  singletonService.checkDataBaseAvailable();
  minTimes=1;
  result=false;
  singletonService.giveCurrentLocalDateTime();
  minTimes=1;
  LocalDateTime expectedLocalDateTime=LocalDateTime.of(2020,10,15,17,30,0);
  result=expectedLocalDateTime;
  SingletonService.sendAlertMail("Luke",expectedLocalDateTime);
  minTimes=1;
}}
```


As you may seem:
- The anonymous class does set `minTimes` and  `result` multiple times
- There is no clear boundary between the mocking of a method (e.g. `singletonService.checkDataBaseAvailable(); result=false;`) and the verirfication of its call ( e.g. `minTimes=1`).
- At the same time, the order of the lines do matter a lot:` result=false;` must be positioned after its method mock but before the next method mocking line (e.g. after `singletonService.checkDataBaseAvailable();`, but before `singletonService.giveCurrentLocalDateTime();`).  

This gets unreadable pretty quickly and in most cases it is not very clear what happens exactly. 








