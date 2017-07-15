# Scala-points

This is my complements to twitter's [Effective Scala](http://twitter.github.io/effectivescala/) and more:
* [Databricks Scala Style](https://github.com/databricks/scala-style-guide)
* [A list of code warts](http://www.wartremover.org/doc/warts.html) (if some parts look scary, treat them at least as warnings.)

-------------------

#### 1.val initialization order:
http://docs.scala-lang.org/tutorials/FAQ/initialization-order.html

when override hashCode, make sure every val involved is not null.

#### 2.pitfall of mapValues and filterNot etc.
https://issues.scala-lang.org/browse/SI-4776

Similar: `Map.keySet`, `Map.values`

some methods do not create new map, but instead, create a MapView

#### (Deprecated)3.disable Mylyn in eclipse to speed up scala compiling.

simply enter Preferences-> Mylyn  disable any Auto*

#### 4.lazy argument

f(x: =>B):C   now, B is something like lazy val a=(expensive-like fun)

#### 5.Enum

a.(recommended)use case object as enumeration, along side with [enumeratum](https://github.com/lloydmeta/enumeratum)

b.override Enumeration http://stackoverflow.com/questions/2507273/overriding-scala-enumeration-value  so you can inherit your own Value creation method in subclass or inner-class

#### 6.evaluation style
```scala
//BAD:
val fun=expensiveFunction.cheapFunction(_:T)  //everytime access fun, both functions are calculated
//GOOD:
val result=expensiveFunction
val fun=result.cheapFunction(_:T)
```
#### 7.use .view properly
```scala
import com.typesafe.scalalogging.LazyLogging

object TempTest extends App with LazyLogging {
  def timer(f: () => Any): Unit = {
    val time1 = System.currentTimeMillis()
    f()
    val time2 = System.currentTimeMillis()
    logger.info(f.toString + " done, time consumed:" + (time2 - time1))
  }

  val f1 = () => {
    r(List(1, 2, 3, 4, 5),1000)
  }
  val f2 = () => {
    r(List(1, 2, 3, 4, 5).view,1000).view.force
  }
  
  def r(l: Seq[Int], n: Int):Seq[Int] = {
    if (n == 0) l
    else r(l.map(_ + 1), n - 1)
  }
  
  timer(f1)
  timer(f2)
}
```
f2 is faster than f1, but, if you change run times from 1000 to 10000, it will throw overflow exception

#### 8.if procedure control
```scala
//BAD:
if(condition1)
  val a=x
else
  val a=y
  
//GOOD:
val a=if(condition1) x else y
```

#### 9.package sealed trait
```scala
package foo
sealed trait I
private[foo] trait I2 extends I
```
trait I2 can be and only can be implemented within package foo

#### 10.abuse of closure
```scala
val map1:Map[Key1,Map[Key2,Value]]=someMap1
val map2:Map[Key1,Map[Key2,Function]]=someMap2

val mapWithProcessedValue=map1.map{
  e1=>
    (e1._1,e1._2.map{
      e2=>
        val fun=map2(e1._1)(e2._1) //Bad, but not worst
        (e2._1,fun(e2._2)) //apply fun to value
    })
}

val funToFind=map2(_:Key1)(_:Key2)
val mapWithProcessedValue=map1.map{
  e1=>
    val partialFun=funToFind(e1.1)_
    (e1._1,e1._2.map{
      e2=>
        val fun=partialFun(e2._1) //Now, you suppose that this be faster, in reality, this is the slowest
        (e2._1,fun(e2._2)) //apply fun to value
    })
}

val mapWithProcessedValue=map1.map{
  e1=>
    val subMap2=map2(e1.1)
    (e1._1,e1._2.map{
      e2=>
        val fun=subMap2(e2._1) //Good.
        (e2._1,fun(e2._2)) //apply fun to value
    })
}
```
#### 11.AOP in scala
```scala
trait Channel {
def send ( x : String ) = println ( x )
}
object LogAspect {
trait LogAfter extends Channel {
// before advice
abstract override def send ( x : String ) = { log ( ) ; super.send ( x ) }
}
trait LogBefore extends Channel {
// after advice
abstract override def send ( x : String ) = { super.send ( x ) ; log ( ) }
}
def log ( ) = println ( " logging ! " )
}
def main( args : Array [ String ] ) = {
val channel = new Channel with LogAspect.LogBefore
channel.send ( "message" )
}
```

#### 12.Be careful about iterator.map
```scala
//not good(because next time you access the iterator, it may have changed):
iterator.map
//better:
iterator.toSeq.map
```
#### 13."Mixin composition"
Favor inheritance over composition for immutable objects.
Mixin of traits only lacks the ability of dynamically changing components, which is also dropped out by immutability.
```scala
trait A {def delegateDo()}
trait B extends A //B not necessary
trait B1 extends B{def delegateDo()=doSomeThing1}
trait B2 extends B{def delegateDo()=doSomeThing2}
val immutableA=condition match{
  case c1 => new A with B1
  case c2 => new A with B2
}
```
Use condition to choose what trait to mix in, instead of inserting composited ones.

#### 14.Recursive sequnce traversal
Using a cursor of indices instead of shrinking the sequence has a slight performance advantage.(scala 2.11.7)

#### 15.Type inference in generic method:
```scala
implicit class ExMap[A, B](val in: Map[A, B]) {
  def innerJoin[B2, C](that: Map[A, B2],f: (B, B2) => C): Map[A, C] // type B2 needs to be explicit
  def innerJoinInfer[B2, C](that: Map[A, B2])(f: (B, B2) => C): Map[A, C] //type B2 in f can be infered from that Map[A,B2]
}
```

#### 16.String interpolation encountering regex.
s-interpolation does't recoganize triple-qouted string parts.
see http://stackoverflow.com/a/25633978/5172925

#### 17.Arguments order of constructor.
Arguments with same type should not be put together, when there the order does not matter in other perspective.
```scala
//Some times not so good:
case class Person(name:String,motto:String,age:Int)
//Better:
case class Person(name:String,age:Int,motto:String)
```

#### 18.Beware when using java API.
The convention of java API may deviate from scala's. e.g. `java.io.File.listFiles` returns `null` when directory cannot be reached. Read java doc carefully.


#### 19.Duck typing.
[See this link](https://dzone.com/articles/duck-typing-scala-structural)

#### 20.Name tuples mapping(avoid calling ._N on Tuples):
```scala
Map(k,v).map(e=> e._1 ...  e._2 ...) //bad. low readability
Map(k,v).map{case (someKey,someValue)=> ...} //good.
```
Applies to Seq[TupleN] alike as well.

```scala
def methodThatReturnsTuple():(A,B,C)

val abc = methodThatReturnsTuple()

//context: when consume the tuple as a whole:
def consume1(abc:ABC)
consume1(abc) //no problem

//context: when consume part of the tuple:
def comsume2(a:A,b:B)
consume2(abc._1,abc._2) //bad

val (a,b,_) = methodThatReturnsTuple()
consume2(a,b) //good.
```

#### 21.Give public field explicit type signature.
See [Incremental compilation](http://www.scala-sbt.org/1.0/docs/Understanding-Recompilation.html)

Named private fields like `private[packageName]` are also considered as public.

There are exceptions: when type signature makes it too unreadable or ugly, and you can make sure this field won't change often. Well make it a `val` or `final val`

#### 22.Align overloaded methods (if you have to overload).
```scala
def method1(a:A)(b:B)
def method1(b:B)
//legal but bad. What if you want to add one more arg group:
def method1(a:A)(b:B)(implicit c:C)
def method1(b:B)(implicit c:C)

method1(aOrb)(bOrc) //What?

//Align them:
def method2(a:A)(b:B)(implicit c:C)
def method2()(b:B)(implicit c:C)

method2()(b) //clear.
```
#### 23.Returning the "Current" Type in Scala (Restriction on return subtype.)
see: http://tpolecat.github.io/2015/04/29/f-bounds.html

#### 24.Only use "return"s as guards.
see first: http://tpolecat.github.io/2014/05/09/return.html

However when `return` is used to control flow on behalf of `require(condition,"exception msg")`:
```scala
//pseudocode:
def check(cred:Credential):Boolean = {
  if(pre-condition-check1) return false
  if(pre-condition-check2) return false
  
  //specific-check..
}
```
it is more readable than:
```scala
def check(cred:Credential):Boolean = {
  if(pre-condition-check1) false
  else if(pre-condition-check2) false
  else {
    //specific-check..
  } 
}
```

#### 25.Use helper recusive function to create recusive type evidence.

```scala
case class Structure(name: String, children: Seq[Structure] = Nil)
object Structure {
  implicit val evidence: TypeClass[Structure] = (a: Structure) => {
     //one cannot recusively use this evidence.
  }
}
```
Use a helper function to help: [example](RecursiveEvidenceExample.md)

#### 26.Default empty generic parameter without Option.

Consider an interface method that you provide to client user:
```scala
def method[T](mandatory:String, optional:T = some-default-value) //how to make this optional
```
Of course, you can use an `Option`:
```scala
def method[T](mandatory:String, optional:Option[T] = None)
//client usage:
method("sth",Option(myValue)) //clients don't need to worry about null value of "myValue", good!
```
But some clients consider `Option(myValue)` as a bit verbose and want you not to use it. 
Now, what the `method` would look like?
```scala
def method[T](mandatory:String, optional:T = null) //able to compile but bad. null is explicitly used.
def method[T](mandatory:String, optional:T = ()) //not compile, Unit is not a type of T
def method[T](mandatory:String, optional:T = Nothing) //not compile, scala does not have an instance of Nothing
```
Use a helper `def` combined with non-strict-param(call-by-name) to provide empty value:
```scala
private def Empty: Nothing = throw new IllegalArgumentException("Empty default value called.")
def method[T](mandatory:String, optional: => T = Empty) 
//yeah! Codes compile and client will not see null or know what happened.
```
But notice, you need to use `try`/`Try` to check this `optional`.`
We could check type `T` exhaustively to manually provide empty value:
```scala
def method[T: ClassTag](mandatory:String, optional: => T = Empty) 
private def Empty[T: ClassTag]: T ={
    val rc = implicitly[ClassTag[T]].runtimeClass
    val empty = rc match{
      case rc if rc == classOf[String] => ""
      case rc if rc == classOf[Int] => 0
      //etc...
      case rc => throw new AssertionError(s"$rc is not supported!")
    }
    empty.asInstanceOf[T]
}
//stupid, but does solve some of the problem.
```

Macro is the ultimate solution:
```scala
private def Empty: Nothing = throw new AssertionError("This should not be triggered.")
def method[T](mandatory:String, optional: => T = Empty) = macro MacroImpl.method
//method parameters are checked and manipulated at client's compile time.
```
