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

## Example Scenario

The starting point is pretty exactly the one as in [How to work with JMockit](#how-to-work-with-jmockit).

In this fiction, your team somehow endet up to program in singleton patern and likes static methods. There is nothing
you can do about. 

However, in this szenario you decide to go with a combination of Mockito and JMockit. 

### Example classes

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

### Actually combining

In the test class, you use JMockit only as far as necessary (by mocking only the static methods with JMockit) 
- to keep the anonymous class (the Expectations) as small as possible
- to use Mockito whereever possible, so you can use its functionality and have as much boundary as possible between mocking and verification

```java
package example.controller;

import example.services.SingletonManager;
import mockit.Expectations;
import mockit.MockUp;
import mockit.Mocked;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.Mock;
import org.mockito.junit.MockitoJUnitRunner;

import java.time.LocalDateTime;

import static org.mockito.Mockito.doReturn;
import static org.mockito.Mockito.verify;

/**
 * Test of class {@link Controller}
 */
@RunWith(MockitoJUnitRunner.class)
public class ControllerTestWithManagerAndJMockitAndMockito {

    private Controller controller;

    @Mock
    private SingletonManager singletonManager;

    @Before
    public void setUp() throws Exception {
        // mocking the getInstance-method with JMockit, returning the instance mocked with Mockito
        new MockUp<SingletonManager>() {
            @mockit.Mock
            public SingletonManager getInstance(){
                return singletonManager;
            }

        };
        controller = new Controller();
    }

    @Test
    public void callService_shouldSendAlertMail_ifNotAvailable() {
        LocalDateTime expectedLocalDateTime = LocalDateTime.of(2020, 10, 15, 17, 30, 0);

        // mocking the instance methods
        doReturn(false).when(singletonManager).checkDataBaseAvailable();
        doReturn(expectedLocalDateTime).when(singletonManager).giveCurrentLocalDateTime();

        // mocking and verification of the static methods
        new Expectations() {{
            SingletonManager.getCurrentUserName();
            result = "Luke";
            minTimes = 1;
            SingletonManager.sendAlertMail("Luke", expectedLocalDateTime);
            minTimes = 1;
        }};

        controller.callManager();

        // verification of the call of the instance methods
        verify(singletonManager).checkDataBaseAvailable();
        verify(singletonManager).giveCurrentLocalDateTime();
    }
}
```

### Conclusion of Mockito combined with JMockit

As mentioned above, this combination is not a very good pattern.

**Plus**

As long as you know where to draw the line between JMockit and Mockito, it is still shorter and more readable than JMockit alone. 
In most cases, you will only mock a `getInstance()` - method and in that case, the rest of the code will look very Mockito-like.

**Minus**

New programmers, who don't know how to work with Mockito nor JMockit, will need a lot of explanation. 
In addition, the code can be very cluttered and gets less readable the more static methods there are.

**Suggestion**

Use this pattern only if 
1. there are a lot non-static methods and 
2. there is no way solve the problem of mocking the remaining static methods otherwise.


