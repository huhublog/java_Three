## Chapter 3. Methods Common to All Objects（对象的通用方法）

### Item 12: Always override toString（总是重写toString方法）

While Object provides an implementation of the toString method, the string that it returns is generally not what the user of your class wants to see. It consists of（由…组成） the class name followed by an “at” sign (@) and the unsigned hexadecimal representation of the hash code, for example,PhoneNumber@163b91. The general contract for toString says that the returned string should be “a concise but informative representation that is easy for a person to read.” While it could be argued that PhoneNumber@163b91 is concise and easy to read, it isn’t very informative when compared to 707-867-5309. The toString contract goes on to say, “It is recommended that all subclasses override this method.” Good advice, indeed!

虽然Object提供toString方法的实现，但它返回的字符串通常不是类的用户希望看到的。它由后跟“at”符号(@)的类名和散列代码的无符号十六进制表示（例如PhoneNumber@163b91）组成。toString的通用约定（是这么描述的），返回的字符串应该是“简洁但信息丰富的表示，易于阅读”。虽然有人认为PhoneNumber@163b91简洁易懂，但与707-867-5309相比，它的信息量并不大。toString约定接着描述，“建议所有子类重写此方法。“好建议，确实!

While it isn’t as critical（adj.至关重要的） as obeying the equals and hashCode contracts (Items 10 and 11), **providing a good toString implementation makes your class much more pleasant to use and makes systems using the class easier to debug.** The toString method is automatically invoked when an object is passed to println, printf, the string concatenation（n.连结） operator, or assert, or is printed by a debugger. Even if you never call toString on an object, others may. For example, a component that has a reference to your object may include the string representation of the object in a logged error message. If you fail to override toString, the message may be all but useless.

虽然它不如遵守equals和hashCode约定（[Item-10](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-3-Item-10-Obey-the-general-contract-when-overriding-equals.md)和[Item-11](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-3-Item-11-Always-override-hashCode-when-you-override-equals.md)）那么重要，但是**提供一个好的toString实现（能）使类更易于使用，使用该类的系统（也）更易于调试。** 当对象被传递给println、printf、字符串连接操作符或断言或由调试器打印时，将自动调用toString方法。即使您从来没有调用toString对象，其他人也可能（使用）。例如，有对象引用的组件可以在日志错误消息中包含对象的字符串表示。如果您未能重写toString，则该消息可能完全无用。

If you’ve provided a good toString method for PhoneNumber,generating a useful diagnostic message is as easy as this:

如果你已经为PhoneNumber提供了一个好的toString方法，那么生成一个有用的诊断消息就像这样简单：

```
System.out.println("Failed to connect to " + phoneNumber);
```

Programmers will generate diagnostic messages in this fashion whether or not you override toString, but the messages won’t be useful unless you do. The benefits of providing a good toString method extend beyond instances of the class to objects containing references to these instances, especially collections.Which would you rather see when printing a map,{Jenny=PhoneNumber@163b91} or {Jenny=707-867-5309}?

无论您是否重写toString，程序员都会以这种方式生成诊断消息，但是除非您（重写toString），否则这些消息不会有用。提供好的toString方法的好处不仅仅是将类的实例扩展到包含对这些实例的引用的对象，特别是集合。在打印map时，您更愿意看到哪个，{Jenny=PhoneNumber@163b91}还是{Jenny=707-867-5309}？

**When practical, the toString method should return all of the interesting information contained in the object,** as shown in the phone number example. It is impractical if the object is large or if it contains state that is not conducive to string representation. Under these circumstances,toString should return a summary such as Manhattan residential phone directory (1487536 listings) or Thread[main,5,main]. Ideally, the string should be self-explanatory. (The Thread example flunks this test.) A particularly annoying penalty for failing to include all of an object’s interesting information in its string representation is test failure reports that look like this:

**当实际使用时，toString方法应该返回对象中包含的所有有趣信息，** 如电话号码示例所示。如果对象很大，或者包含不利于字符串表示的状态，那么这种方法是不切实际的。在这种情况下，toString应该返回一个摘要，例如曼哈顿住宅电话目录（1487536号清单）或Thread[main,5,main]。理想情况下，字符串应该是不言自明的。（线程示例未能通过此测试。）如果没有在字符串表示中包含所有对象的有趣信息，那么一个特别恼人的惩罚就是测试失败报告，如下所示:

```
Assertion failure: expected {abc, 123}, but was {abc, 123}.
```

One important decision you’ll have to make when implementing a toString method is whether to specify the format of the return value in the documentation. It is recommended that you do this for value classes, such as phone number or matrix. The advantage of specifying the format is that it serves as a standard, unambiguous, human-readable representation of the object. This representation can be used for input and output and in persistent human-readable data objects, such as CSV files. If you specify the format, it’s usually a good idea to provide a matching static factory or constructor so programmers can easily translate back and forth between the object and its string representation. This approach is taken by many value classes in the Java platform libraries, including BigInteger, BigDecimal, and most of the boxed primitive classes.

在实现toString方法时，您必须做的一个重要决定是是否在文档中指定返回值的格式。建议您针对值类（如电话号码或矩阵）这样做。指定格式的优点是，它可以作为对象的标准的、明确的、人类可读的表示。这种表示可以用于输入和输出，也可以用于持久的人类可读数据对象，比如CSV文件。如果指定了格式，提供一个匹配的静态工厂或构造函数通常是一个好主意，这样程序员就可以轻松地在对象及其字符串表示之间来回转换。Java平台库中的许多值类都采用这种方法，包括BigInteger、BigDecimal和大多数包装类。

The disadvantage of specifying the format of the toString return value is that once you’ve specified it, you’re stuck with it for life, assuming your class is widely used. Programmers will write code to parse the representation, to generate it, and to embed it into persistent data. If you change the representation in a future release, you’ll break their code and data, and they will yowl. By choosing not to specify a format, you preserve the flexibility to add information or improve the format in a subsequent release.

指定toString返回值的格式的缺点是，一旦指定了它，就会终生使用它，假设您的类被广泛使用。程序员将编写代码来解析表示、生成表示并将其嵌入持久数据中。如果您在将来的版本中更改了表示形式，您将破坏它们的代码和数据，它们将发出大量的消息。通过选择不指定格式，您可以保留在后续版本中添加信息或改进格式的灵活性。

**Whether or not you decide to specify the format, you should clearly document your intentions.** If you specify the format, you should do so precisely. For example, here’s a toString method to go with the PhoneNumber class in Item 11:

**无论您是否决定指定格式，您都应该清楚地记录您的意图。** 如果指定了格式，则应该精确地指定格式。例如，这里有一个toString方法用于[Item-11](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-3-Item-11-Always-override-hashCode-when-you-override-equals.md)中的PhoneNumber类：

```
/**
* Returns the string representation of this phone number.
* The string consists of twelve characters whose format is
* "XXX-YYY-ZZZZ", where XXX is the area code, YYY is the
* prefix, and ZZZZ is the line number. Each of the capital
* letters represents a single decimal digit.
**
If any of the three parts of this phone number is too small
* to fill up its field, the field is padded with leading zeros.
* For example, if the value of the line number is 123, the last
* four characters of the string representation will be "0123".
*/
@Override public String toString() {
return String.format("%03d-%03d-%04d",
areaCode, prefix, lineNum);
}
```

If you decide not to specify a format, the documentation comment should read something like this:

如果您决定不指定一种格式，文档注释应该如下所示：

```
/**
* Returns a brief description of this potion. The exact details
* of the representation are unspecified and subject to change,
* but the following may be regarded as typical:
**
"[Potion #9: type=love, smell=turpentine, look=india ink]"
*/
@Override public String toString() { ... }
```

After reading this comment, programmers who produce code or persistent data that depends on the details of the format will have no one but themselves to blame when the format is changed.

在阅读了这篇文档注释之后，当格式被更改时，生成依赖于格式细节的代码或持久数据的程序员将只能怪他们自己。

Whether or not you specify the format, **provide programmatic access to the information contained in the value returned by toString.** For example, the PhoneNumber class should contain accessors for the area code, prefix, and line number. If you fail to do this, you force programmers who need this information to parse the string. Besides reducing performance and making unnecessary work for programmers, this process is error-prone and results in fragile systems that break if you change the format. By failing to provide accessors, you turn the string format into a de facto API, even if you’ve specified that it’s subject to change.

无论您是否指定了格式，都要**提供对toString返回值中包含的信息的程序性访问。** 例如，PhoneNumber类应该包含区域代码、前缀和行号的访问器。如果做不到这一点，就会迫使需要这些信息的程序员解析字符串。除了降低性能和使程序员不必要的工作之外，这个过程很容易出错，并且会导致脆弱的系统在您更改格式时崩溃。由于没有提供访问器，您可以将字符串格式转换为事实上的API，即使您已经指定了它可能会发生更改。

It makes no sense to write a toString method in a static utility class (Item 4). Nor should you write a toString method in most enum types (Item 34) because Java provides a perfectly good one for you. You should, however, write a toString method in any abstract class whose subclasses share a common string representation. For example, the toString methods on most collection implementations are inherited from the abstract collection classes.

在静态实用程序类中编写toString方法是没有意义的（[Item-4](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-2-Item-4-Enforce-noninstantiability-with-a-private-constructor.md)），在大多数enum类型中也不应该编写toString方法（[Item-34](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-6-Item-34-Use-enums-instead-of-int-constants.md)），因为Java为您提供了一个非常好的方法。但是，您应该在任何抽象类中编写toString方法，该类的子类共享公共的字符串表示形式。例如，大多数集合实现上的toString方法都继承自抽象集合类。

Google’s open source AutoValue facility, discussed in Item 10, will generate a toString method for you, as will most IDEs. These methods are great for telling you the contents of each field but aren’t specialized to the meaning of the class. So, for example, it would be inappropriate to use an automatically generated toString method for our PhoneNumber class (as phone numbers have a standard string representation), but it would be perfectly acceptable for our Potion class. That said, an automatically generated toString method is far preferable to the one inherited from Object, which tells you nothing about an object’s value.

谷歌的开放源码自动值工具（在[Item-10](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-3-Item-10-Obey-the-general-contract-when-overriding-equals.md)中讨论）将为您生成toString方法，大多数IDE也是如此。这些方法可以很好地告诉您每个字段的内容，但并不专门针对类的含义。因此，例如，对于PhoneNumber类使用自动生成的toString方法是不合适的（因为电话号码具有标准的字符串表示形式），但是对于Potion类来说它是完全可以接受的。也就是说，一个自动生成的toString方法要比从对象继承的方法好得多，对象继承的方法不会告诉你对象的值。

To recap, override Object’s toString implementation in every instantiable class you write, unless a superclass has already done so. It makes classes much more pleasant to use and aids in debugging. The toString method should return a concise, useful description of the object, in an aesthetically pleasing format.

回顾一下，在您编写的每个实例化类中重写对象的toString实现，除非超类已经这样做了。它使类更易于使用，并有助于调试。toString方法应该以美观的格式返回对象的简明、有用的描述。
