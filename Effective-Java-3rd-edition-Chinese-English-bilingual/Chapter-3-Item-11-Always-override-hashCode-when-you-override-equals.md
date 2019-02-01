## Chapter 3. Methods Common to All Objects（对象的通用方法）

### Item 11: Always override hashCode when you override equals（当重写equals时，也重写hashCode）

**You must override hashCode in every class that overrides equals.** If you fail to do so, your class will violate（vt.违反） the general contract for hashCode, which will prevent it from functioning properly（adv.适当地，正确地） in collections such as HashMap and HashSet. Here is the contract, adapted from the Object specification :

**在重写equals的类中，必须重写hashCode。** 如果您没有这样做，您的类将违反hashCode的一般约定，这将阻止该类在HashMap和HashSet等集合中正常运行。以下是根据目标规范修改的约定：

- When the hashCode method is invoked on an object repeatedly during an execution of an application, it must consistently（adv.一贯地，一致地） return the same value,provided no information used in equals comparisons is modified. This value need not remain consistent from one execution of an application to another.

当在应用程序执行过程中对对象重复调用hashCode方法时，它必须一致地返回相同的值，前提是不修改用于对等比较的信息。从一个应用程序的一次执行到另一次执行，这个值不需要保持一致。

- If two objects are equal according to the equals(Object) method, then calling hashCode on the two objects must produce the same integer result.

如果根据equals(Object)方法，两个对象是相等的，那么在两个对象上调用hashCode必须产生相同的整数结果。

- If two objects are unequal according to the equals(Object) method, it is not required that calling hashCode on each of the objects must produce distinct results. However, the programmer should be aware that producing distinct results for unequal objects may improve the performance of hash tables.

如果根据equals(Object)方法（判断出）两个对象不相等，则不需要在每个对象上调用hashCode时必须产生不同的结果。但是，程序员应该知道，为不相等的对象生成不同的结果可能会提高哈希表的性能。

**The key provision（n.规定，条款） that is violated when you fail to override hashCode is the second one: equal objects must have equal hash codes.** Two distinct instances may be logically equal according to a class’s equals method, but to Object’s hashCode method, they’re just two objects with nothing much in common. Therefore, Object’s hashCode method returns two seemingly random numbers instead of two equal numbers as required by the contract.For example, suppose you attempt to use instances of the PhoneNumber class from Item 10 as keys in a HashMap:

**当您无法覆盖hashCode时，违反的关键条款是第二个：相等的对象必须具有相等的散列代码。** 根据类的equals方法，两个不同的实例在逻辑上可能是相等的，但是对于对象的hashCode方法来说，它们只是两个没有什么共同之处的对象。因此，Object的hashCode方法返回两个看似随机的数字，而不是约定要求的两个相等的数字。例如，假设您尝试使用[Item-10](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-3-Item-10-Obey-the-general-contract-when-overriding-equals.md)中的PhoneNumber类实例作为HashMap中的键：

```
Map<PhoneNumber, String> m = new HashMap<>();
m.put(new PhoneNumber(707, 867, 5309), "Jenny");
```

At this point（这时，此时此刻）, you might expect m.get(new PhoneNumber(707, 867,5309)) to return "Jenny", but instead, it returns null. Notice that two PhoneNumber instances are involved: one is used for insertion into the HashMap, and a second, equal instance is used for (attempted) retrieval（n.检索）. The PhoneNumber class’s failure to override hashCode causes the two equal instances to have unequal hash codes, in violation（n.违反） of the hashCode contract.Therefore, the get method is likely to look for the phone number in a different hash bucket from the one in which it was stored by the put method. Even if the two instances happen to hash to the same bucket, the get method will almost certainly return null, because HashMap has an optimization that caches the hash code associated with each entry and doesn’t bother checking for object equality if the hash codes don’t match.

此时，您可能期望m.get(new PhoneNumber(707, 867,5309))返回“Jenny”，但是它返回null。注意，（这里）涉及到两个PhoneNumber实例：一个用于插入到HashMap中，另一个equal实例用于（尝试）检索。PhoneNumber类未能覆盖hashCode，导致两个相等的实例具有不相等的哈希代码，这违反了hashCode约定。因此，get方法可能会在与put方法存储电话号码的散列桶不同的哈希桶中查找电话号码。即使这两个实例碰巧散列到同一个哈希桶上，get方法几乎肯定会返回null，因为HashMap有一个优化，它缓存与每个条目相关联的散列代码，如果散列代码不匹配，就不会费心检查对象是否相等。

Fixing this problem is as simple as writing a proper hashCode method for PhoneNumber. So what should a hashCode method look like? It’s trivial to write a bad one. This one, for example, is always legal but should never be used:

解决这个问题就像为PhoneNumber编写一个正确的hashCode方法一样简单。那么hashCode方法应该是什么样的呢？写一个不好的是微不足道的。举个例子，这个方法总是合法的，但是不应该被使用:

```
// The worst possible legal hashCode implementation - never use!
@Override public int hashCode() { return 42; }
```

It’s legal（adj.合法的） because it ensures that equal objects have the same hash code. It’s atrocious because it ensures that every object has the same hash code. Therefore,every object hashes to the same bucket, and hash tables degenerate to linked lists. Programs that should run in linear time instead run in quadratic time. For large hash tables, this is the difference between working and not working.

它是合法的，因为它确保了相等的对象具有相同的哈希代码。同时它也很糟糕，因为它确保每个对象都有相同的哈希代码。因此，每个对象都散到同一个桶中，哈希表退化为链表。应以线性时间替代运行的程序。对于大型哈希表，这是工作和不工作的区别。

A good hash function tends to produce unequal hash codes for unequal instances. This is exactly what is meant by the third part of the hashCode contract. Ideally, a hash function should distribute（vt.分配） any reasonable collection of unequal instances uniformly across all int values. Achieving this ideal can be difficult. Luckily it’s not too hard to achieve a fair approximation（n.接近）. Here is a simple recipe:

一个好的哈希函数倾向于为不相等的实例生成不相等的哈希代码。这正是hashCode约定的第三部分的含义。理想情况下，哈希函数应该在所有int值之间均匀分布所有不相等实例的合理集合。实现这个理想是很困难的。幸运的是，实现一个类似的并不太难。这里有一个简单的方式：

1、Declare an int variable named result, and initialize it to the hash code c for the first significant field in your object, as computed in step 2.a. (Recall from Item 10 that a significant field is a field that affects equals comparisons.)

声明一个名为result的int变量，并将其初始化为对象中第一个重要字段的哈希代码c，如步骤2.a中计算的那样。（回想一下[Item-10](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-3-Item-10-Obey-the-general-contract-when-overriding-equals.md)中的重要字段是影响相等比较的字段。）

2、For every remaining significant field f in your object, do the following:

对象中剩余的重要字段f，执行以下操作：

a. Compute an int hash code c for the field:

为字段计算一个int哈希码c：

i. If the field is of a primitive type, compute Type.hashCode(f),where Type is the boxed primitive class corresponding to f’s type.

如果字段是基本数据类型，计算Type.hashCode(f)，其中type是与f类型对应的包装类。

ii. If the field is an object reference and this class’s equals method compares the field by recursively（adv.递归地） invoking equals, recursively invoke hashCode on the field. If a more complex comparison is required,compute a “canonical representation” for this field and invoke hashCode on the canonical representation. If the value of the field is null, use 0 (or some other constant, but 0 is traditional).

如果字段是对象引用，并且该类的equals方法通过递归调用equals来比较字段，则递归调用字段上的hashCode。如果需要更复杂的比较，则为该字段计算一个“规范表示”，并在规范表示上调用hashCode。如果字段的值为空，则使用0（或其他常数，但0是传统的）。

iii. If the field is an array, treat it as if each significant element were a separate field. That is, compute a hash code for each significant element by applying these rules recursively, and combine the values per step 2.b. If the array has no significant elements, use a constant, preferably not 0. If all elements are significant, use Arrays.hashCode.

如果字段是一个数组，则将其视为每个重要元素都是一个单独的字段。也就是说，通过递归地应用这些规则计算每个重要元素的哈希代码，并将每个步骤2.b的值组合起来。如果数组中没有重要元素，则使用常量，最好不是0。如果所有元素都很重要，那么使用Arrays.hashCode。

b. Combine the hash code c computed in step 2.a into result as follows:result = 31 * result + c;

将步骤2.a中计算的哈希代码c合并到结果如下： result = 31 * result + c;

3、Return result.

返回result。

When you are finished writing the hashCode method, ask yourself whether equal instances have equal hash codes. Write unit tests to verify your intuition (unless you used AutoValue to generate your equals and hashCode methods,in which case you can safely omit these tests). If equal instances have unequal hash codes, figure out why and fix the problem.

当您完成了hashCode方法的编写之后，问问自己相同的实例是否具有相同的散列代码。编写单元测试来验证您的直觉（除非您使用AutoValue生成您的equals和hashCode方法，在这种情况下您可以安全地省略这些测试）。如果相同的实例有不相等的哈希码，找出原因并修复问题。

You may exclude（vt.排除） derived（adj.派生的） fields from the hash code computation. In other words, you may ignore any field whose value can be computed from fields included in the computation. You must exclude any fields that are not used in equals comparisons, or you risk violating the second provision of the hashCode contract.

可以从散列代码计算中排除派生字段。换句话说，您可以忽略任何可以从计算中包含的字段计算其值的字段。您必须排除不用于对等比较的任何字段，否则您可能会违反hashCode约定的第二个条款。

The multiplication in step 2.b makes the result depend on the order of the fields, yielding a much better hash function if the class has multiple similar fields. For example, if the multiplication were omitted from a String hash function, all anagrams would have identical hash codes. The value 31 was chosen because it is an odd prime. If it were even and the multiplication overflowed, information would be lost, because multiplication by 2 is equivalent to shifting. The advantage of using a prime is less clear, but it is traditional. A nice property of 31 is that the multiplication can be replaced by a shift and a subtraction for better performance on some architectures: 31 * i == (i <<5) - i. Modern VMs do this sort of optimization automatically.

第二步的乘法。b使结果取决于字段的顺序，如果类有多个相似的字段，则产生一个更好的哈希函数。例如，如果字符串哈希函数中省略了乘法，那么所有的字谜都有相同的哈希码。选择31是因为它是奇素数。如果是偶数，乘法运算就会溢出，信息就会丢失，因为乘法运算等于移位。使用素数的好处不太明显，但它是传统的。31的一个很好的特性是，可以用移位和减法来代替乘法，从而在某些体系结构上获得更好的性能：31 * i == (i <<5) – i。现代VMs自动进行这种优化。

Let’s apply the previous recipe to the PhoneNumber class:

让我们将前面的方法应用到PhoneNumber类：

```
// Typical hashCode method
@Override public int hashCode() {
int result = Short.hashCode(areaCode);
result = 31 * result + Short.hashCode(prefix);
result = 31 * result + Short.hashCode(lineNum);
return result;
}
```

Because this method returns the result of a simple deterministic（adj.确定性的） computation whose only inputs are the three significant fields in a PhoneNumber instance,it is clear that equal PhoneNumber instances have equal hash codes. This method is, in fact, a perfectly good hashCode implementation for PhoneNumber, on par with those in the Java platform libraries. It is simple, is reasonably fast, and does a reasonable job of dispersing unequal phone numbers into different hash buckets.

因为这个方法返回一个简单的确定性计算的结果，它的唯一输入是PhoneNumber实例中的三个重要字段，所以很明显，相等的PhoneNumber实例具有相等的哈希码。实际上，这个方法是PhoneNumber的一个非常好的hashCode实现，与Java平台库中的hashCode实现相当。它很简单，速度也相当快，并且合理地将不相等的电话号码分散到不同的散列桶中。

While the recipe in this item yields reasonably good hash functions, they are not state-of-the-art. They are comparable in quality to the hash functions found in the Java platform libraries’ value types and are adequate for most uses. If you have a bona fide need for hash functions less likely to produce collisions, see Guava’s com.google.common.hash.Hashing [Guava].

虽然本条目中的方法产生了相当不错的散列函数，但它们并不是最先进的。它们的质量可与Java平台库的值类型中的散列函数相媲美，对于大多数用途来说都是足够的。如果您确实需要不太可能产生冲突的散列函数，请参阅Guava的com.google.common.hash. hash [Guava]。

The Objects class has a static method that takes an arbitrary number of objects and returns a hash code for them. This method, named hash, lets you write one-line hashCode methods whose quality is comparable to those written according to the recipe in this item. Unfortunately, they run more slowly because they entail array creation to pass a variable number of arguments, as well as boxing and unboxing if any of the arguments are of primitive type. This style of hash function is recommended for use only in situations where performance is not critical. Here is a hash function for PhoneNumber written using this technique:

对象类有一个静态方法，它接受任意数量的对象并返回它们的哈希代码。这个名为hash的方法允许您编写一行哈希代码方法，这些方法的质量可以与根据本项中的菜谱编写的方法媲美。不幸的是，它们运行得更慢，因为它们需要创建数组来传递可变数量的参数，如果任何参数是原始类型的，则需要进行装箱和拆箱。推荐只在性能不重要的情况下使用这种散列函数。下面是使用这种技术编写的PhoneNumber哈希函数：

```
// One-line hashCode method - mediocre performance
@Override public int hashCode() {
return Objects.hash(lineNum, prefix, areaCode);
}
```

If a class is immutable and the cost of computing the hash code is significant,you might consider caching the hash code in the object rather than recalculating it each time it is requested. If you believe that most objects of this type will be used as hash keys, then you should calculate the hash code when the instance is created. Otherwise, you might choose to lazily initialize the hash code the first time hash-Code is invoked. Some care is required to ensure that the class remains thread-safe in the presence of a lazily initialized field (Item 83). Our PhoneNumber class does not merit this treatment, but just to show you how it’s done, here it is. Note that the initial value for the hashCode field (in this case, 0) should not be the hash code of a commonly created instance:

如果一个类是不可变的，并且计算哈希代码的成本非常高，那么您可以考虑在对象中缓存哈希代码，而不是在每次请求时重新计算它。如果您认为这种类型的大多数对象都将用作哈希键，那么您应该在创建实例时计算哈希代码。否则，您可能选择在第一次调用哈希代码时延迟初始化哈希代码。在一个延迟初始化的字段（[Item-83](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-11-Item-83-Use-lazy-initialization-judiciously.md)）的情况下，需要一些注意来确保该类仍然是线程安全的。我们的PhoneNumber类不值得进行这种处理，但只是为了向您展示它是如何实现的，在这里。注意，hashCode字段的初始值（在本例中为0）不应该是通常创建的实例的哈希代码:

```
// hashCode method with lazily initialized cached hash code
private int hashCode; // Automatically initialized to 0
@Override public int hashCode() {
int result = hashCode;
if (result == 0) {
result = Short.hashCode(areaCode);
result = 31 * result + Short.hashCode(prefix);
result = 31 * result + Short.hashCode(lineNum);
hashCode = result;
} return result;
}
```

**Do not be tempted to exclude significant fields from the hash code computation to improve performance.** While the resulting hash function may run faster, its poor quality may degrade hash tables’ performance to the point where they become unusable. In particular, the hash function may be confronted with a large collection of instances that differ mainly in regions you’ve chosen to ignore. If this happens, the hash function will map all these instances to a few hash codes, and programs that should run in linear time will instead run in quadratic time.

**不要试图从哈希代码计算中排除重要字段，以提高性能。** 虽然得到的哈希函数可能运行得更快，但其糟糕的质量可能会将哈希表的性能降低到无法使用的程度。特别是，哈希函数可能会遇到大量实例，这些实例主要在您选择忽略的区域不同。如果发生这种情况，哈希函数将把所有这些实例映射到一些哈希代码，应该在线性时间内运行的程序将在二次时间内运行。

This is not just a theoretical problem. Prior to Java 2, the String hash function used at most sixteen characters evenly（adv.均匀地） spaced throughout the string,starting with the first character. For large collections of hierarchical names, such as URLs, this function displayed exactly the pathological（adj.病态的） behavior described earlier.

这不仅仅是一个理论问题。在Java 2之前，字符串哈希函数在字符串中，以第一个字符开始，最多使用16个字符。对于大量的分级名称集合（如url），该函数完全显示了前面描述的病态行为。

**Don’t provide a detailed specification for the value returned by hashCode, so clients can’t reasonably depend on it; this gives you the flexibility to change it.** Many classes in the Java libraries, such as String and Integer, specify（vt.指定，详细说明） the exact value returned by their hashCode method as a function of the instance value. This is not a good idea but a mistake that we’re forced to live with: It impedes the ability to improve the hash function in future releases. If you leave the details unspecified and a flaw is found in the hash function or a better hash function is discovered, you can change it in a subsequent release.

**不要为hashCode返回的值提供详细的规范，这样客户端就不能合理地依赖它。这（也）给了您更改它的灵活性。** Java库中的许多类，例如String和Integer，都将hashCode方法返回的确切值指定为实例值的函数。这不是一个好主意，而是一个我们不得不面对的错误：它阻碍了在未来版本中改进哈希函数的能力。如果您保留了未指定的细节，并且在散列函数中发现了缺陷，或者发现了更好的散列函数，那么您可以在后续版本中更改它。

In summary, you must override hashCode every time you override equals,or your program will not run correctly. Your hashCode method must obey the general contract specified in Object and must do a reasonable job assigning unequal hash codes to unequal instances. This is easy to achieve, if slightly tedious, using the recipe on page 51. As mentioned in Item 10, the AutoValue framework provides a fine alternative to writing equals and hashCode methods manually, and IDEs also provide some of this functionality.

总之，每次重写equals时都必须重写hashCode，否则程序将无法正确运行。您的hashCode方法必须遵守Object中指定的通用约定，并且必须合理地将不相等的哈希代码分配给不相等的实例。这很容易实现，如果有点乏味，（可）使用第51页的方法。如[Item-10](https://github.com/clxering/Effective-Java-3rd-edition-Chinese-English-bilingual/blob/master/Chapter-3-Item-10-Obey-the-general-contract-when-overriding-equals.md)所述，AutoValue框架提供了一种很好的替代手动编写equals和hashCode的方法，IDE也提供了这种功能。
