# csharp-code-evaluator
Dynamically evaluate C# code

This repo provides class CSharpExpression that allows the developer to run a C# expression(s) dynamically with full access to a set of externally provided objects.

The usage is very simple. Just create a CSharpExpression object, add "outside" objects in its scope, and they become accesible to the expression(s) within.

All expressions contained within CSharpExpression share a "global" dynamic instance object where state can be kept between executions and effectively share data between expressions.

Examples:

The container program declares a class:

```C#
public class TestClass
{
  public int int_Field;
}
```

later the following can be executed:

```C#
var expression = new CSharpExpression("1 + obj.int_Field");
var obj = new TestClass {int_Field = 2};
expression.AddObjectInScope("obj", obj);
Assert.AreEqual(3, expression.Execute());
```
CSharpExpression also support working with "dynamic" types via ExpandoObject.

the following also works:

```C#
var expression = new CSharpExpression("1 + obj.int_Field");
IDictionary<string, object> obj = new ExpandoObject();
obj.Add("int_Field", 2);
expression.AddObjectInScope("obj", obj);
Assert.AreEqual(3, expression.Execute());
```

you can also use "code snippets" that do more than just a be an evaluatable expresion:

```C#
var expression = new CSharpExpression();
expression.AddCodeSnippet("var i = 1; return 1 + i");
Assert.AreEqual(2, expression.Execute());
```
  
for optimal performance in cases where you have thousends of expressions, you want to bundle them together in one instance CSharpExpression class.
See this example:

```C#
var expression = new CSharpExpression();
for(var i = 1; i < 1000; i++)
  Assert.AreEqual(i - 1, expression.AddExpression($"{i} + 1"));
for (var i = 1; i < 1000; i++)
  Assert.AreEqual(i + 1, expression.Execute(i - 1));
```

you can run expressions that don't return a value:

```C#
var expression = new CSharpExpression();
Assert.AreEqual(0, expression.AddVoidReturnCodeSnippet("var i = 1; Console.WriteLine(i)"));
Assert.AreEqual(null, expression.Execute());
```

and finally, to mesmerize our JavaScript loving friends, you can keep global state between executions by doing this:

```C#
var expression = new CSharpExpression();
Assert.AreEqual(0, expression.AddVoidReturnCodeSnippet("global.a = 0")); // this injects an int field into global
Assert.AreEqual(1, expression.AddExpression("global.a++"));
Assert.AreEqual(null, expression.Execute()); // setup the global
Assert.AreEqual(0, expression.Execute(1));
Assert.AreEqual(1, expression.Execute(1));
```