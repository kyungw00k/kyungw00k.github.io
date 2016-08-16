---
layout: post
title: JSR 223 - Scripting for the Java Platform
date: 2010-09-27 16:07:30.000000000 +09:00
type: post
published: true
status: publish
categories:
- Programming
- Java
---

### JSR 223 소개
![wpid-wpid-jw-0424-scripting3-2010-09-27-17-07-2010-09-27-17-07.gif](/images/2010/09/27/wpid-wpid-jw-0424-scripting3-2010-09-27-17-07-2010-09-27-17-07.gif)

- Java SE 6에는 Script Engine을 지원하기 위해 API를 제공한다. 이는 Java에서 특정 Script를 실행시킬 수 있게 해줄 뿐 아니라, Java의 Context와 연결하기 위해 일종의 Entry Point를 제공해, Context를 공유할 수 있다.
- 임의의 Script 언어를 사용하기 위해서는 javax.script Package를 상속받아 구현함으로서 외부 언어를 지원 할 수 있도록 한다.
- [scripting: Project Home Page](https://scripting.dev.java.net/)에서 JSR 223 API가 구현된 Script 언어들을 확인 할 수 있다. (유사한 프로젝트로는 [Jakarta BSF - Bean Scripting Framework](http://jakarta.apache.org/bsf)가 있다.)


### 간단한 Scripting API 사용 방법
`javax.script` 패키지는 아래와 같이 6개 Interface / 5개의 Class / 1개의 Exception로 구성되어 있다.

|Interface|Class|Exception|
|----|----|----|
|Bindings|AbstractScriptEngine|ScriptException|
|Compilable|CompiledScript|Invocable|
|ScriptEnginemanager|ScriptContext|SimpleBindings|
|ScriptEngine|SimpleScriptContext|ScriptEngineFactory|

위에서 가장 눈여겨 보아야할 두 객체가 있다.

- `ScriptEngineManager`는 어떤 Script Engine이 JRE에서 사용할 수 있는지 알려주는 객체다.
- `ScriptEngine`는 `ScriptEngineManager`에서 지원하는 언어들로 작성된 Script 실행할 수 있게 하는 객체이다.

위의 두 객체를 이용해 Script언어를 실행하기까지는 크게 3단계로 나뉠 수 있다.

- ScriptEngineManager 객체를 만든다.
- ScriptEngineManager 객체로 부터 사용하고자 하는 Script의 ScriptEngine 객체를 받는다.
- ScriptEngine 객체를 사용하여 Script를 실행한다.

위 단계를 사용해 간단히 JavaScript Code를 실행시키는 예제를 살펴보자.

```java
ScriptEngineManager mgr = new ScriptEngineManager();
ScriptEngine jsEngine = mgr.getEngineByName(“JavaScript”);

try {
    jsEngine.eval(“print(‘Hello, world!’)”);
} catch (ScriptException ex) {
    ex.printStackTrace();
}
```

### 사용가능한 **Script Engines**
Java 6에는 기본적으로 Mozilla Rhino engine(JavaScript)이 포함되어 있다. 그 이외에 언어를 지원하고 싶다면 일단 [https://scripting.dev.java.net](https://scripting.dev.java.net에)에 JSR 223에서 사용 가능한 Script Engine 목록이 있는지 확인 하는게 좋겠다. 만약 새로운 언어를 임의로 추가하여 사용하고 싶다면
[Making Scripting Languages JSR-223](http://today.java.net/pub/a/today/2006/09/21/making-scripting-languages-jsr-223-aware.html) 를 참고하기 바란다.

`ListScriptingtEngines.java` 에서는 JVM에서 사용 가능한 Script Engine 목록을 보여준다. ScriptEngineManager는 JVM에서 사용 가능한 ScriptEngine Class들을 찾고, 이를 Key/Value 형태로 관리한다.

`ScriptEngineManager` 객체는 Script Framework를 위한 일종의 Discovery Mechanism을 제공함으로서 ScriptEngineFactory(ScriptEngine 객체를 생성)를 찾는다. 보다 더 상세한 정보는 해당 Script Library의 JAR File Specification에 들어 있다.

```java
//
// ListScriptingEngines.java
//
import java.util.List;
import javax.script.ScriptEngineFactory;
import javax.script.ScriptEngineManager;

public class ListScriptingEngines {
    public static void main(String[] args) {
        ScriptEngineManager manager = new ScriptEngineManager();
        List engines = manager.getEngineFactories();

        if (engines.isEmpty()) {
            System.out.println("No scripting engines were found");
            return;
        }

        System.out.println("The following " + engines.size() + "scripting engines were found");
        System.out.println();

        for (ScriptEngineFactory engine : engines) {
            System.out.println("Engine name: "+engine.getEngineName());
            System.out.println("\tVersion: "+engine.getEngineVersion());
            System.out.println("\tLanguage: "+engine.getLanguageName());

            List extensions = engine.getExtensions();

            if (extensions.size() > 0) {
                System.out.println("\tEngine supports the following extensions:");

                for (String e : extensions) {
                    System.out.println("\t\t" + e);
                }
            }

            List shortNames = engine.getNames();

            if (shortNames.size() > 0) {
                System.out.println("\tEngine has the following short names:");

                for (String n : engine.getNames()) {
                    System.out.println("\t\t" + n);
                }
            }
            System.out.println("=========================");
        }
    }
}
```

`EngineManager`의 Instance 생성 후에 `getEngineFactories()` Method를 실행하였다.

이 Method는 Discovery Mechanism에 의해 발견된 ScriptEngineFactory Class를 모두 보여준다.

각 ScriptEngineFactory에는 Engine에 대한 Metadata를 가지고 있는데, 다음과 같은 Method를 사용할 수 있다.

- getEngineName()
- getEngineVersion()
- getLangugeName()
- getExtensions()
- getNames()

위 코드 실행 결과는 다음과 같다.

```
The following 1 scripting engines were found:

Engine name: Mozilla Rhino
  Version: 1.6 release 2
  Language: ECMAScript
  Engine supports the following extensions:
    js
  Engine has the following short names:
    js
    rhino
    JavaScript
    javascript
    ECMAScript
    ecmascript
=========================
```

### Script 불러오기 & 실행시키기
Python, Ruby가 JVM에서 사용할 수 있다고 가정하자(Classpath 설정 필요).
예를 들어 다음과 같이 서로 다른 언어로 작성 된 2개의 파일이 있을 때, 아래와 같은 코드를 사용해 파일의 확장자로 해당 Script Engine을 불러와 Script를 실행해 보자.

```python
// Test.py
print 'Testing Jython!'
```

```ruby
# Test.rb
puts 'Testing JRuby!'
```

```java
// In JavaCode

String ext = fileName.substring(fileName.lastIndexOf(“.”)+1);
ScriptEngine engine = manager.getEngineByExtension(ext);

if (engine != null) {
  ScriptEngineFactory factory = engine.getFactory();

  // Script 파일 이름 및 Engine의 정보 출력
  System.out.println(“Running “ + fileName + “ using engine “ +
    factory.getEngineName() + “ Version “ +
    factory.getEngineVersion() + “ for language “ +
    factory.getLanguageName());

  // Script 실행
  engine.eval(new FileReader(f));
}
```

위의 코드를 사용하여, `JavaScriptingDemo.java` 에서는 Test로 시작하는 파일 중 Script Engine이 지원하는 언어의 확장자의 파일을 실행시킨다.

```java
//
// JavaScriptingDemo.java
//
import java.io.File;
import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.FilenameFilter;
import java.util.logging.Level;
import java.util.logging.Logger;
import javax.script.ScriptEngine;
import javax.script.ScriptEngineFactory;
import javax.script.ScriptEngineManager;
import javax.script.ScriptException;

/**
 * 현재 Directory에 있는 Test로 시작하는 파일을 찾아보고,
 * <p>
 * <p>
 * 각 파일의 확장자에 맞는 엔진을 JVM에서 지원하면 실행한다.
 *
 * @author ptremblett
 */
public class JavaScriptingDemo {
    /**
     * @param args the command line arguments
     */
    public static void main(String[] args) {
        ScriptEngineManager manager = new ScriptEngineManager();
        File scriptsDir = new File("scripts");
        File[] scripts = scriptsDir.listFiles(new TestFilter());
        for (File f : scripts) {
            String fileName = f.getName();
            String ext = fileName.substring(fileName.lastIndexOf(".") + 1);
            ScriptEngine engine = manager.getEngineByExtension(ext);
            if (engine != null) {
                try {
                    ScriptEngineFactory factory = engine.getFactory();
                    System.out.println("Running " + fileName + "using engine " +
                            factory.getEngineName() + "Version " +
                            factory.getEngineVersion() + "for language " +
                            factory.getLanguageName());
                    engine.eval(new FileReader(f));
                } catch (FileNotFoundException ex) {
                    Logger.getLogger(JavaScriptingDemo.class.getName()).log(Level.SEVERE, null, ex);
                } catch (ScriptException ex) {
                    Logger.getLogger(JavaScriptingDemo.class.getName()).log(Level.SEVERE, null, ex);
                } finally {
                    System.out.println("==========================");
                }
            } else {
                System.err.println("Could not find scripting engine for " + f.getName());
            }
        }
    }
}

class TestFilter implements FilenameFilter {
    public boolean accept(File dir, String name) {
        return name.startsWith("Test.");
    }
}
```

결과는 아래와 같다.

```
Running Test.py using engine jython Version 2.5.0 for language python
Testing Jython!
============================
Running Test.rb using engine JRuby Engine Version 1.5.0 for language ruby
Testing JRuby!
============================
```

### Java 에서 Script 내 함수 호출하기

아래와 같이 `HelloGoodbye.rb`에 3개의 함수가 있다고 하자.


```ruby
# HelloGoodbye.rb
def hello()
  puts "Hello world!"
end

def goodbye()
  puts "Goodbye cruel world!"
end

def wait(seconds)
 print "Waiting ",seconds," seconds"
 sleep(seconds)
 puts "Finished waiting"
end
```

아래의 코드는 위의 코드에서 hello()와 goodbye()를 호출한 결과를 보여준다.

```java
// InvokeFunctions.java

import java.io.FileNotFoundException;
import java.io.FileReader;
import java.util.List;
import java.util.logging.Level;
import java.util.logging.Logger;
import javax.script.Invocable;
import javax.script.ScriptEngine;
import javax.script.ScriptEngineFactory;
import javax.script.ScriptEngineManager;
import javax.script.ScriptException;

/**
 * @author ptremblett
 */
public class InvokeFunctions {
    /**
     * @param args the command line arguments
     */
    public static void main(String[] args) {
        ScriptEngineManager manager = new ScriptEngineManager();
        ScriptEngine engine = manager.getEngineByName(“ruby”);
        try {
            engine.eval(new FileReader(“HelloGoodbye.rb”));
            Invocable inv = (Invocable) engine;
            try {
                System.out.println(“Invoking hello”);
                inv.invokeFunction(“hello”);
                System.out.println(“Invoking goodbye”);
                inv.invokeFunction(“goodbye”);
            } catch (NoSuchMethodException ex) {
                Logger.getLogger(InvokeFunctions.class.getName()).log(Level.SEVERE, null, ex);
            }
        } catch (FileNotFoundException ex) {
            Logger.getLogger(InvokeFunctions.class.getName()).log(Level.SEVERE, null, ex);
        } catch (ScriptException ex) {
            Logger.getLogger(InvokeFunctions.class.getName()).log(Level.SEVERE, null, ex);
        }
    }
}
```

제약사항이 있다면, 위의 기능을 사용하기 전에 해당 언어에 Invocable Interface가 구현되어 있는지 여부를 확인해야 한다.

### Binding 을 통한 공유
Java의 Object를 Script에서, 또는 Script의 값을 Java에서 사용할 수가 있다.

이는 ScriptEngine Instance의 put, get Method를 사용해서 공유하고자 하는 Object를 넣거나 가져오는 방법을 사용한다.

간단한 예제로 Java의 Object를 Ruby로 넘겨서 값을 변경한 후 출력해 보자.


Java에서는 임의의 Object를 선언해 Ruby Context에서 볼 수 있도록 하자.

```java
Double rand = new Double(0.0);

// Engine에서는 randomNumber라는 이름으로 Java의 Double Type Object가 넘어간다.
engine.put(“randomNumber”, rand);
```

```ruby
# share.rb
# randomNumber의 값을 바꾼다.
$randomNumber = rand()
```

아래는 위의 예제를 사용해 Engine Scope에 Binding되어 있는 Object명과 그 값을 보여준다.


```java
// ...
        ScriptEngineManager manager = new ScriptEngineManager();
        ScriptEngine engine = manager.getEngineByName("ruby");
        try {
            // ScriptEngine으로 넘겨질 Object
            Double rand = new Double(0.0);
            engine.put("randomNumber", rand);
            Bindings bindings = engine.getBindings(ScriptContext.ENGINE_SCOPE);
            Iterator > it = bindings.entrySet().iterator();
            System.out.println("Bindings follow:");
            while (it.hasNext()) {
                System.out.println(it.next());
            }
            System.out.println("**end of bindings list**");
            System.out.println("Generating 10random numbers using script share.rb");
            for (int i = 0; i < 10; ++i) {
                // Script를 실행시킨 후
                engine.eval(new FileReader("share.rb"));

                // Engine Scope에서 randomNumber를 가져옴.
                System.out.println(engine.get("randomNumber"));
            }
        } catch (FileNotFoundException ex) {
            Logger.getLogger(ShareObjects.class.getName()).log(Level.SEVERE, null, ex);
        } catch (ScriptException ex) {
            Logger.getLogger(ShareObjects.class.getName()).log(Level.SEVERE, null, ex);
        }
// ...
```

### 결론
Sandboxing이라는 문제가 남아있긴 하지만, 다른 언어와 Context를 공유할 경우가 있다면 JSR 223은 분명 매력적인 방법임에 틀림없다.

### References
- [SDN : Scripting for the Java Platform](http://java.sun.com/developer/technicalArticles/J2SE/Desktop/scripting/)
- [Dr Dobbs - JSR 223: Scripting for the Java Platform](http://www.drdobbs.com/java/215801163)
- [Making Scripting Languages JSR-223](http://today.java.net/pub/a/today/2006/09/21/making-scripting-languages-jsr-223-aware.html)
- [scripting: Project Home Page](https://scripting.dev.java.net/)
- [Mozilla Rhino project](http://www.mozilla.org/rhino/)
