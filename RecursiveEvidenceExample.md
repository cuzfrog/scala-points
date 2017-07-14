#### Recursive type evidence example

```scala
//typeclass:
trait Definable[A] {
  def define(a: A): String
}

object Helper {
  implicit class DefineOps[A: Definable](a: A) {
    def defined: String = {
      val definable = implicitly[Definable[A]]
      definable.define(a)
    }
  }
}

import Helper._

//class and evidence:
case class Structure(name: String,
                     children: Seq[Structure] = Nil)
object Structure {
  //  implicit val definable: Definable[Structure] = (a: Structure) => {
  //    s"""${a.name}
  //       |+-${a.children.map(_.defined)} //compile error
  //     """.stripMargin
  //  }

  implicit val definable: Definable[Structure] = (a: Structure) => recDefine(a, "")

  private def recDefine(a: Structure, indent: String): String = {
    a.children match {
      case Nil => a.name
      case chdren =>
        a.name + "\n" +
          chdren.map(s => indent + "+-" + recDefine(s, indent + " ")).mkString("\n")
    }
  }
}

object TestStructure extends App {

  val seq = Structure(
    "top",
    Seq(
      Structure("level1-A",
        Seq(
          Structure("level2-AA")
        )
      ),
      Structure("level1-B",
        Seq(
          Structure("level2-BA",
            Seq(
              Structure("level3-BAA"),
              Structure("level3-BAB")
            )
          )
        )
      )
    )
  )

  println(seq.defined)
}
```
