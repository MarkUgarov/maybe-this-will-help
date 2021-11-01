# Working with Mockito

I would prefer Mockito over JMockit where ever possible. 
However, as mentioned [here](junitAndMockitoAndJmockit.md) there are some usages where to combine them.  

Nevertheless, the following page will focus on the usage of mockito.


## Example classes

In this fiction, your team just started with a little project, you don't use fancy JavaEE-Features or Java Specification 
Request 330 (JSR330) yet, so you don't have to dive into that for our little example.

You wrote two little classes: a `Controller` and a `Service`. Assume we are in an early stage of the project and just want
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

import example.services.Service;
import example.services.SingletonManager;

public class Controller {
    
    private Service service;
    
    public Controller(Service service){
        this.service = service;
    }
    
    public void callService() {
        if(!service.checkDataBaseAvailable()){
            service.sendAlertMail(
                    service.getCurrentUserName(),
                    service.giveCurrentLocalDateTime()
            );
        }
    }
}
```

```java
package example.services;

import java.time.LocalDateTime;

public class Service {

    public boolean checkDataBaseAvailable() {
        return false;
    }

    public LocalDateTime giveCurrentLocalDateTime() {
        return null;
    }

    public String getCurrentUserName() {
        return null;
    }

    public void sendAlertMail(String username, LocalDateTime localDateTime) {
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

import example.services.Service;
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
public class ControllerTest {

    private Controller controller;

    @Mock
    private Service service;

    @Before
    public void setUp() throws Exception {
        // normally we do this with @Inject etc., but here we try to avoid complexity
        controller = new Controller(service);
    }

    @Test
    public void callService_shouldSendAlertMail_ifNotAvailable() {
        LocalDateTime expectedLocalDateTime = LocalDateTime.of(2020, 10, 15, 17, 30, 0);

        // mocking block
        doReturn(false).when(service).checkDataBaseAvailable();
        doReturn("Luke").when(service).getCurrentUserName();
        doReturn(expectedLocalDateTime).when(service).giveCurrentLocalDateTime();

        controller.callService();
        
        // verification block
        verify(service).checkDataBaseAvailable();
        verify(service).getCurrentUserName();
        verify(service).giveCurrentLocalDateTime();
        verify(service).sendAlertMail("Luke", expectedLocalDateTime);
    }
}
```
{% endraw %}

This is pretty simple, right?

### Conclusion of Mockito

As we see, mocking and verifying with mockito is pretty straight forward.

- There is a clear boundary between mocking a method (e.g.  `doReturn(false).when(service).checkDataBaseAvailable();`) and verifying its call (e.g. `verify(service).checkDataBaseAvailable();`).
- There is no anonymous class, instead there are two separate "blocks"
  - It's clearly visible what result is mocked on which method and what method is validated to be called.  
  - The order of lines in the "blocks" are not really important.
  

Compare that to [jmockit](jmockit.md), which is much less readable. 

### Aditional hint

Of course Mockito is more powerful than this little example. E.g. you can use 

`verify(service, Mockito.never()).checkDataBaseAvailable();`

or

`verify(service, Mockito.atMost(2)).getCurrentUserName();`

or even

`Mockito.verifyZeroInteractions(service);`
 
to have mor nuances in your verifications. 

Also you can improve your mocking in certain situations by doing something like 

```java
 doReturn(false, true)
        .when(service).checkDataBaseAvailable();
```
to return `false` on the first call, but `true` on the second. 

Those are just examples, please feel free to google. 

