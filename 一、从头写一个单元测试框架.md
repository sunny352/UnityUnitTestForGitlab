# Unity单元测试和Gitlab自动化集成（一）：从头写一个单元测试框架

首先，我们编写一个简单的单元测试框架，用于测试我们的代码。这个框架的功能很简单，就是能够执行一些测试用例，并且能够输出测试结果。

## 简单的单元测试框架

下面是一个简单的单元测试套件的基类

```csharp
public class TestSuite
{
    protected virtual List<Action> Tests { get; }

    protected virtual void SetupSuite()
    {
    }

    protected virtual void TearDownSuite()
    {
    }
    
    protected virtual void SetupTest()
    {
    }

    protected virtual void TearDownTest()
    {
    }

    public void Run()
    {
        if (null == Tests)
        {
            throw new Exception("Tests is null");
        }
        
        SetupSuite();
        foreach (var test in Tests)
        {
            SetupTest();
            test();
            TearDownTest();
        }
        TearDownSuite();
    }
}
```

这个类的功能很简单，就是在执行每个测试用例之前，先执行SetupTest()方法，然后执行测试用例，最后执行TearDownTest()方法。在执行所有测试用例之前，先执行SetupSuite()方法，最后执行TearDownSuite()方法。

然后，写一个断言用来判断两个整数是否相等

```csharp
public static class Assert
{
    public static void AreEqual(int expected, int actual)
    {
        if (expected != actual)
        {
            throw new Exception($"Assert.AreEqual failed. expected: {expected}, actual: {actual}");
        }
    }
}
```

最后，写一个启动器，用于启动测试套件

```csharp
public static class TestRunner
{
    private static List<TestSuite> TestSuites { get; } = new List<TestSuite>
    {
        // TODO: add test suites here
    };
    
    [MenuItem("Tools/Run Tests")]
    public static void Run()
    {
        Debug.Log("TestRunner.Run() started.");
        foreach (var testSuite in TestSuites)
        {
            testSuite.Run();
        }
        Debug.Log("TestRunner.Run() finished.");
    }
}
```

这个启动器的功能很简单，就是执行TestSuites中的所有测试套件。

以上就是一个符合最基本要求的单元测试框架。下面，我们来写一个测试用例，用于测试这个单元测试框架。

## 测试单元测试框架

我们要测试的代码是一个简单的加法计算器。代码如下：

```csharp
public class Calculator
{
    public int Add(int a, int b)
    {
        return a + b;
    }
}
```

然后，我们写一个测试套件，用于测试这个加法计算器。代码如下：

```csharp
public class CalculatorTestSuite : TestSuite
{
    protected override List<Action> Tests { get; } 
    
    public CalculatorTestSuite()
    {
        Tests = new List<Action>
        {
            TestAdd
        };
    }

    private Calculator _calculator;

    protected override void SetupSuite()
    {
        Debug.Log("CalculatorTestSuite.SetupSuite()");
        _calculator = new Calculator();
    }

    protected override void TearDownSuite()
    {
        Debug.Log("CalculatorTestSuite.TearDownSuite()");
        _calculator = null;
    }
    
    protected override void SetupTest()
    {
        Debug.Log("CalculatorTestSuite.SetupTest()");
    }
    
    protected override void TearDownTest()
    {
        Debug.Log("CalculatorTestSuite.TearDownTest()");
    }

    private void TestAdd()
    {
        Debug.Log("CalculatorTestSuite.TestAdd()");
        var result = _calculator.Add(1, 2);
        Assert.AreEqual(3, result);
    }
}
```

这个测试套件的功能很简单，就是测试Calculator.Add()方法。在执行每个测试用例之前，先执行SetupSuite()方法，然后执行测试用例，最后执行TearDownSuite()方法。

然后，我们在TestRunner中添加这个测试套件。代码如下：

```csharp
private static List<TestSuite> TestSuites { get; } = new List<TestSuite>
{
    new CalculatorTestSuite()
};
```

最后，点击Unity菜单中的Tools->Run Tests，就可以执行这个测试套件了。

## 使用反射简化测试套件的编写

上面的测试套件的代码很繁琐，因为每次都要将测试用例添加到TestSuite的Tests列表中。我们可以使用反射来简化这个过程。规定所有的Test开头的方法都是测试用例，然后使用反射来获取所有的测试用例。代码如下：

```csharp
public class TestSuite
{
    protected virtual void SetupSuite()
    {
    }

    protected virtual void TearDownSuite()
    {
    }

    protected virtual void SetupTest()
    {
    }

    protected virtual void TearDownTest()
    {
    }

    public void Run()
    {
        SetupSuite();
        var methods = GetType().GetMethods();
        foreach (var methodInfo in methods)
        {
            if (!methodInfo.Name.StartsWith("Test"))
            {
                continue;
            }
            
            SetupTest();
            methodInfo.Invoke(this, null);
            TearDownTest();
        }
        
        TearDownSuite();
    }
}
```

这样，我们就不需要在TestSuite的子类中重写Tests属性了。

## 使用Attribute来标记测试用例

上面的代码中，我们规定所有的Test开头的方法都是测试用例。但是，这样的规定并不够灵活。我们可以使用Attribute来标记测试用例。代码如下：

```csharp
public class TestAttribute : Attribute
{
}
```

```csharp
public class TestSuite
{
    protected virtual void SetupSuite()
    {
    }

    protected virtual void TearDownSuite()
    {
    }

    protected virtual void SetupTest()
    {
    }

    protected virtual void TearDownTest()
    {
    }

    public void Run()
    {
        SetupSuite();
        var methods = GetType().GetMethods();
        foreach (var methodInfo in methods)
        {
            var attributes = methodInfo.GetCustomAttributes(typeof(TestAttribute), false);
            if (attributes.Length == 0)
            {
                continue;
            }
            
            SetupTest();
            methodInfo.Invoke(this, null);
            TearDownTest();
        }
        
        TearDownSuite();
    }
}
```

然后，我们在测试用例的方法上添加TestAttribute。代码如下：

```csharp
public class CalculatorTestSuite : TestSuite
{
    protected override void SetupSuite()
    {
        Debug.Log("CalculatorTestSuite.SetupSuite()");
        _calculator = new Calculator();
    }

    protected override void TearDownSuite()
    {
        Debug.Log("CalculatorTestSuite.TearDownSuite()");
        _calculator = null;
    }
    
    protected override void SetupTest()
    {
        Debug.Log("CalculatorTestSuite.SetupTest()");
    }
    
    protected override void TearDownTest()
    {
        Debug.Log("CalculatorTestSuite.TearDownTest()");
    }

    [Test]
    private void TestAdd()
    {
        Debug.Log("CalculatorTestSuite.TestAdd()");
        var result = _calculator.Add(1, 2);
        Assert.AreEqual(3, result);
    }
}
```

## 使用Attribute来标记测试套件

上面的代码中，我们规定所有的TestSuite结尾的类都是测试套件。但是，这样的规定并不够灵活。我们可以使用Attribute来标记测试套件。代码如下：

```csharp
public class TestSuiteAttribute : Attribute
{
}
```

```csharp
public class TestRunner
{
    [MenuItem("Tools/Run Tests")]
    public static void Run()
    {
        Debug.Log("TestRunner.Run() started.");
        var types = Assembly.GetExecutingAssembly().GetTypes();
        foreach (var type in types)
        {
            var attributes = type.GetCustomAttributes(typeof(TestSuiteAttribute), false);
            if (attributes.Length == 0)
            {
                continue;
            }
            
            var testSuite = (TestSuite) Activator.CreateInstance(type);
            testSuite.Run();
        }
        Debug.Log("TestRunner.Run() finished.");
    }
}
```

然后，我们在测试套件的类上添加TestSuiteAttribute。代码如下：

```csharp
[TestSuite]
public class CalculatorTestSuite : TestSuite
{
    protected override void SetupSuite()
    {
        Debug.Log("CalculatorTestSuite.SetupSuite()");
        _calculator = new Calculator();
    }

    protected override void TearDownSuite()
    {
        Debug.Log("CalculatorTestSuite.TearDownSuite()");
        _calculator = null;
    }
    
    protected override void SetupTest()
    {
        Debug.Log("CalculatorTestSuite.SetupTest()");
    }
    
    protected override void TearDownTest()
    {
        Debug.Log("CalculatorTestSuite.TearDownTest()");
    }

    [Test]
    private void TestAdd()
    {
        Debug.Log("CalculatorTestSuite.TestAdd()");
        var result = _calculator.Add(1, 2);
        Assert.AreEqual(3, result);
    }
}
```

到这里，我们就完成了一个简单的单元测试框架。我们可以在Unity中使用这个框架来编写单元测试。