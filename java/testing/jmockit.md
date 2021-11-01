# Working with JMockit

JMockit is not my choice when it comes to mocking classes. However, it has its usages, e.g as described in [Combining JUnit, Mockito and JMockit](junitAndMockitoAndJmockit.md#why-should-i-do-that)


In this fiction, your team somehow endet up to program in singleton patern and likes static methods. There is nothing
you can do about.

## Example classes

You wrote two little classes: a `Controller` and a `SingletonManager`. Assume we are in an early stage of the project and just want
to make first steps by implementing the `Controller` without having the full functionality of the backend. In this
szenario, the `Controller` holds the "logic" while the `SingletonManager` is supposed to provide the rest of the "functionality"
later on.

**The process:**

The `Controller` checks if the database is available (via the `Service`).

If so, everything is nice.

If not, the `Controller` calls the `SingletonManager` again, evaluates current user and Time and sends an E-Mail.

The `SingletonManager` is (obviously) just a prototype. But we can already test our Controller without implementing everything
else.

```java
package example.controller;

import example.services.SingletonManager;

public class Controller {

    private final SingletonManager singletonManager;

    public Controller() {
        singletonManager = SingletonManager.getInstance();
    }

    public void callManager() {
        if(!singletonManager.checkDataBaseAvailable()){ // calling the instance method
            SingletonManager.sendAlertMail( // calling the class method
                    SingletonManager.getCurrentUserName(),  // calling the class method
                    singletonManager.giveCurrentLocalDateTime() // calling the instance method
            );
        }
    }
}
```

```java
package example.services;

import java.time.LocalDateTime;

public class SingletonManager {

    private static SingletonManager instance;

    public static SingletonManager getInstance() {
        if (instance == null) {
            instance = new SingletonManager();
        }
        return instance;
    }

    private SingletonManager() {

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

{% raw %}
```java
package example.controller;

import example.services.SingletonManager;
import mockit.Expectations;
import mockit.MockUp;
import mockit.Mocked;
import org.junit.Before;
import org.junit.Test;

import java.time.LocalDateTime;

/**
 * Test of class {@link Controller}
 */
public class ControllerTest {

    private Controller controller;

    @Mocked
    private SingletonManager singletonManager;

    @Before
    public void setUp() throws Exception {
        controller = new Controller();
    }

    @Test
    public void callService_shouldSendAlertMail_ifNotAvailable() {
        // mocking the static methods
        new MockUp<SingletonManager>() {
            @mockit.Mock
            public String getCurrentUserName (){
                return "Luke";
            }
        };
        // mocking the instance methods
        new Expectations() {{
            singletonManager.checkDataBaseAvailable();
            minTimes = 1;
            result = false;
            singletonManager.giveCurrentLocalDateTime();
            minTimes = 1;
            LocalDateTime expectedLocalDateTime = LocalDateTime.of(2020, 10, 15, 17, 30, 0);
            result = expectedLocalDateTime;
            SingletonManager.sendAlertMail("Luke", expectedLocalDateTime);
            minTimes = 1;
        }};

        controller.callService();
    }
}
```
{% endraw %}

with an anonymous class extending mockit.Expectations:

{% raw %}
```java
new Expectations(){{
        singletonManager.checkDataBaseAvailable();
        minTimes = 1;
        result = false;
        singletonManager.giveCurrentLocalDateTime();
        minTimes = 1;
        LocalDateTime expectedLocalDateTime = LocalDateTime.of(2020, 10, 15, 17, 30, 0);
        result = expectedLocalDateTime;
        SingletonManager.sendAlertMail("Luke", expectedLocalDateTime);
        minTimes = 1;
}}
```
{% endraw %}

### Conclusion of JMockit

As you may seem:

- There is no clear boundary between the mocking of a method (e.g. `singletonService.checkDataBaseAvailable(); result=false;`) and the verirfication of its call ( e.g. `minTimes=1`).
- At the same time, the order of the lines do matter a lot:
  - The anonymous class does set `minTimes` and  `result` multiple times. 
  - In our example,` result=false;` must be positioned after its method mock but before the next method mocking line (e.g. after `singletonService.checkDataBaseAvailable();`, but before `singletonService.giveCurrentLocalDateTime();`).

This gets unreadable pretty quickly and in most cases it is not very clear what happens exactly. 

Compare that to [mockito](mockito.md), which is way more readable. 

# Parent topic

This page is a subpage of [Combining JUnit, Mockito and JMockit](junitAndMockitoAndJmockit.md)