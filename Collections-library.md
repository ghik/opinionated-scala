* hierarchy overview
* read-only, immutable and mutable
* variance
* aliases, how to import
* companions, default implementations
* java interop
* implementations
* operations
 * performance safety
 * strict & non-strict operations, views, `withDefault`/`withDefaultValue`/`filterKeys`/`mapValues`
 * `CanBuildFrom`

### Java <-> Scala converters
 
Scala => Java:
* `Iterator => JIterator`
* `Iterator => Enumeration`
* `Iterable => JIterable`
* `Iterable => JCollection`
* `Seq => JList`
* `mutable.Seq => JList`
* `Buffer => JList`
* `Set => JSet`
* `mutable.Set => JSet`
* `Map => JMap`
* `mutable.Map => JMap`
* `mutable.Map => Dictionary`
* `concurrent.Map => ConcurrentMap`

Java => Scala:
* `JIterator => Iterator`
* `Enumeration => Iterator`
* `JIterable => sc.Iterable`
* `JCollection => sc.Iterable`
* `JList => Buffer`
* `JSet => mutable.Set`
* `JMap => mutable.Map`
* `ConcurrentMap => concurrent.Map`
* `Dictionary => mutable.Map`
* `Properties => mutable.Map`

### API

Basic language types:
* `scala.Any extends ` + 9 additional members
* `java.lang.Object extends scala.Any` + 8 additional members
* `scala.Equals extends scala.Any` + 1 additional members
* `scala.Function1 extends java.lang.Object` + 3 additional members
* `scala.PartialFunction extends scala.Function1` + 5 additional members

Traversables:
* `TraversableOnce extends scala.Any` + 53 additional members
* `Traversable extends TraversableOnce with java.lang.Object` + 41 additional members
* `immutable.Traversable extends Traversable` + 0 additional members
* `mutable.Traversable extends Traversable` + 0 additional members
* `Iterator extends TraversableOnce with java.lang.Object` + 33 additional members

Iterables:
* `Iterable extends scala.Equals with Traversable` + 11 additional members
* `immutable.Iterable extends Iterable with immutable.Traversable` + 0 additional members
* `mutable.Iterable extends Iterable with mutable.Traversable` + 0 additional members
* `Seq extends Iterable with scala.PartialFunction` + 42 additional members

Sequences and buffers:
* `immutable.Seq extends Seq with immutable.Iterable` + 0 additional members
* `mutable.Seq extends Seq with mutable.Iterable` + 3 additional members
* `mutable.Builder extends java.lang.Object` + 10 additional members
* `mutable.Buffer extends mutable.Seq` + 25 additional members
* `mutable.ListBuffer extends mutable.Builder with mutable.Buffer` + 1 additional members

Linear sequences:
* `LinearSeq extends Seq` + 0 additional members
* `immutable.LinearSeq extends LinearSeq with immutable.Seq` + 0 additional members
* `mutable.LinearSeq extends LinearSeq with mutable.Seq` + 0 additional members
* `immutable.List extends immutable.LinearSeq` + 8 additional members
* `immutable.Stream extends immutable.LinearSeq` + 4 additional members
* `immutable.Queue extends immutable.LinearSeq` + 5 additional members
* `mutable.Stack extends mutable.Seq` + 8 additional members
* `mutable.ArrayStack extends mutable.Builder with mutable.Seq` + 8 additional members
* `mutable.PriorityQueue extends mutable.Builder with mutable.Iterable` + 12 additional members
* `mutable.MutableList extends mutable.Builder with mutable.LinearSeq` + 3 additional members

Indexed sequences:
* `IndexedSeq extends Seq` + 0 additional members
* `immutable.IndexedSeq extends IndexedSeq with immutable.Seq` + 0 additional members
* `mutable.IndexedSeq extends IndexedSeq with mutable.Seq` + 1 additional members
* `immutable.Vector extends immutable.IndexedSeq` + 0 additional members
* `immutable.NumericRange extends immutable.IndexedSeq` + 7 additional members
* `immutable.Range extends immutable.IndexedSeq` + 10 additional members
* `immutable.WrappedString extends immutable.IndexedSeq` + 31 additional members
* `mutable.WrappedArray extends mutable.IndexedSeq` + 4 additional members
* `mutable.ArraySeq extends mutable.IndexedSeq` + 1 additional members
* `mutable.StringBuilder extends mutable.Builder with mutable.IndexedSeq` + 80 additional members
* `mutable.ArrayBuffer extends mutable.IndexedSeq with mutable.Builder with mutable.Buffer` + 1 additional members

Sets:
* `Set extends Iterable with scala.Function1` + 17 additional members
* `immutable.Set extends Set with immutable.Iterable` + 0 additional members
* `mutable.Set extends mutable.Builder with Set with mutable.Iterable` + 9 additional members
* `immutable.HashSet extends immutable.Set` + 1 additional members
* `immutable.BitSet extends BitSet with immutable.SortedSet` + 0 additional members
* `immutable.ListSet extends immutable.Set` + 0 additional members
* `mutable.HashSet extends mutable.Set` + 2 additional members
* `mutable.LinkedHashSet extends mutable.Set` + 0 additional members

Sorted sets:
* `SortedSet extends Set` + 12 additional members
* `immutable.SortedSet extends SortedSet with immutable.Set` + 0 additional members
* `mutable.SortedSet extends mutable.Set with SortedSet` + 0 additional members
* `immutable.TreeSet extends immutable.SortedSet` + 1 additional members
* `mutable.TreeSet extends mutable.SortedSet` + 0 additional members
* `BitSet extends SortedSet` + 6 additional members
* `immutable.BitSet extends BitSet with immutable.SortedSet` + 0 additional members
* `mutable.BitSet extends BitSet with mutable.SortedSet` + 5 additional members

Maps:
* `Map extends scala.PartialFunction with Iterable` + 19 additional members
* `immutable.Map extends Map with immutable.Iterable` + 3 additional members
* `mutable.Map extends mutable.Builder with Map with mutable.Iterable` + 12 additional members
* `immutable.HashMap extends immutable.Map` + 2 additional members
* `immutable.ListMap extends immutable.Map` + 0 additional members
* `mutable.HashMap extends mutable.Map` + 1 additional members
* `mutable.OpenHashMap extends mutable.Map` + 0 additional members
* `mutable.WeakHashMap extends mutable.Map` + 1 additional members
* `mutable.ListMap extends mutable.Map` + 0 additional members

Sorted maps:
* `SortedMap extends Map` + 12 additional members
* `immutable.SortedMap extends SortedMap with immutable.Map` + 0 additional members
* `immutable.TreeMap extends immutable.SortedMap` + 1 additional members

Concurrent maps:
* `concurrent.Map extends mutable.Map` + 4 additional members
* `concurrent.TrieMap extends concurrent.Map` + 14 additional members

#### Detailed API:

`scala.Any extends ` + 9 additional members:
* `!=(x$1: Any)Boolean`
* `==(x$1: Any)Boolean`
* `##()Int`
* `asInstanceOf[T0]=> T0`
* `equals(x$1: Any)Boolean`
* `getClass()java.lang.Class[_]`
* `hashCode()Int`
* `isInstanceOf[T0]=> Boolean`
* `toString()java.lang.String`

`java.lang.Object extends scala.Any` + 8 additional members:
* `eq(x$1: AnyRef)Boolean`
* `ne(x$1: AnyRef)Boolean`
* `notify()Unit`
* `notifyAll()Unit`
* `synchronized[T0](x$1: T0)T0`
* `wait()Unit`
* `wait(x$1: Long)Unit`
* `wait(x$1: Long, x$2: Int)Unit`

`scala.Equals extends scala.Any` + 1 additional members:
* `canEqual(that: Any)Boolean`

`scala.Function1 extends java.lang.Object` + 3 additional members:
* `andThen[A](g: R => A)T1 => A`
* `apply(v1: T1)R`
* `compose[A](g: A => T1)A => R`

`scala.PartialFunction extends scala.Function1` + 5 additional members:
* `applyOrElse[A1 <: A, B1 >: B](x: A1, default: A1 => B1)B1`
* `isDefinedAt(x: A)Boolean`
* `lift => A => Option[B]`
* `orElse[A1 <: A, B1 >: B](that: PartialFunction[A1,B1])PartialFunction[A1,B1]`
* `runWith[U](action: B => U)A => Boolean`

`TraversableOnce extends scala.Any` + 53 additional members:
* `:\[B](z: B)(op:(A, B) => B)B`
* `/:[B](z: B)(op:(B, A) => B)B`
* `addString(b: StringBuilder)StringBuilder`
* `addString(b: StringBuilder, sep: String)StringBuilder`
* `addString(b: StringBuilder, start: String, sep: String, end: String)StringBuilder`
* `aggregate[B](z: => B)(seqop:(B, A) => B, combop:(B, B) => B)B`
* `collectFirst[B](pf: PartialFunction[A,B])Option[B]`
* `copyToArray[B >: A](xs: Array[B])Unit`
* `copyToArray[B >: A](xs: Array[B], start: Int)Unit`
* `copyToArray[B >: A](xs: Array[B], start: Int, len: Int)Unit`
* `copyToBuffer[B >: A](dest: mutable.Buffer[B])Unit`
* `count(p: A => Boolean)Int`
* `exists(p: A => Boolean)Boolean`
* `find(p: A => Boolean)Option[A]`
* `fold[A1 >: A](z: A1)(op:(A1, A1) => A1)A1`
* `foldLeft[B](z: B)(op:(B, A) => B)B`
* `foldRight[B](z: B)(op:(A, B) => B)B`
* `forall(p: A => Boolean)Boolean`
* `foreach[U](f: A => U)Unit`
* `hasDefiniteSize => Boolean`
* `isEmpty => Boolean`
* `isTraversableAgain => Boolean`
* `max[B >: A](implicit cmp: Ordering[B])A`
* `maxBy[B](f: A => B)(implicit cmp: Ordering[B])A`
* `min[B >: A](implicit cmp: Ordering[B])A`
* `minBy[B](f: A => B)(implicit cmp: Ordering[B])A`
* `mkString => String`
* `mkString(sep: String)String`
* `mkString(start: String, sep: String, end: String)String`
* `nonEmpty => Boolean`
* `product[B >: A](implicit num: Numeric[B])B`
* `reduce[A1 >: A](op:(A1, A1) => A1)A1`
* `reduceLeft[B >: A](op:(B, A) => B)B`
* `reduceLeftOption[B >: A](op:(B, A) => B)Option[B]`
* `reduceOption[A1 >: A](op:(A1, A1) => A1)Option[A1]`
* `reduceRight[B >: A](op:(A, B) => B)B`
* `reduceRightOption[B >: A](op:(A, B) => B)Option[B]`
* `seq => sc.TraversableOnce[A]`
* `size => Int`
* `sum[B >: A](implicit num: Numeric[B])B`
* `to[Col[_]](implicit cbf: sc.generic.CanBuildFrom[Nothing,A,Col[A]])Col[A]`
* `toArray[B >: A](implicit evidence$1: scala.reflect.ClassTag[B])Array[B]`
* `toBuffer[B >: A]=> mutable.Buffer[B]`
* `toIndexedSeq => immutable.IndexedSeq[A]`
* `toIterable => Iterable[A]`
* `toIterator => Iterator[A]`
* `toList => List[A]`
* `toMap[T, U](implicit ev: <:<[A,(T, U)])immutable.Map[T,U]`
* `toSeq => Seq[A]`
* `toSet[B >: A]=> immutable.Set[B]`
* `toStream => Stream[A]`
* `toTraversable => Traversable[A]`
* `toVector => Vector[A]`

`Traversable extends TraversableOnce with java.lang.Object` + 41 additional members:
* `++[B >: A, That](that: sc.GenTraversableOnce[B])(implicit bf: sc.generic.CanBuildFrom[Repr,B,That])That`
* `++:[B >: A, That](that: Traversable[B])(implicit bf: sc.generic.CanBuildFrom[Repr,B,That])That`
* `++:[B >: A, That](that: sc.TraversableOnce[B])(implicit bf: sc.generic.CanBuildFrom[Repr,B,That])That`
* `collect[B, That](pf: PartialFunction[A,B])(implicit bf: sc.generic.CanBuildFrom[Repr,B,That])That`
* `companion => sc.generic.GenericCompanion[Traversable]`
* `drop(n: Int)Repr`
* `dropWhile(p: A => Boolean)Repr`
* `filter(p: A => Boolean)Repr`
* `filterNot(p: A => Boolean)Repr`
* `flatMap[B, That](f: A => sc.GenTraversableOnce[B])(implicit bf: sc.generic.CanBuildFrom[Repr,B,That])That`
* `flatten[B](implicit asTraversable: A => sc.GenTraversableOnce[B])CC[B]`
* `foreach[U](f: A => U)Unit`
* `genericBuilder[B]=> mutable.Builder[B,CC[B]]`
* `groupBy[K](f: A => K)immutable.Map[K,Repr]`
* `head => A`
* `headOption => Option[A]`
* `init => Repr`
* `inits => Iterator[Repr]`
* `last => A`
* `lastOption => Option[A]`
* `map[B, That](f: A => B)(implicit bf: sc.generic.CanBuildFrom[Repr,B,That])That`
* `par => ParRepr`
* `partition(p: A => Boolean)(Repr, Repr)`
* `repr => Repr`
* `scan[B >: A, That](z: B)(op:(B, B) => B)(implicit cbf: sc.generic.CanBuildFrom[Repr,B,That])That`
* `scanLeft[B, That](z: B)(op:(B, A) => B)(implicit bf: sc.generic.CanBuildFrom[Repr,B,That])That`
* `scanRight[B, That](z: B)(op:(A, B) => B)(implicit bf: sc.generic.CanBuildFrom[Repr,B,That])That`
* `slice(from: Int, until: Int)Repr`
* `span(p: A => Boolean)(Repr, Repr)`
* `splitAt(n: Int)(Repr, Repr)`
* `stringPrefix => String`
* `tail => Repr`
* `tails => Iterator[Repr]`
* `take(n: Int)Repr`
* `takeWhile(p: A => Boolean)Repr`
* `transpose[B](implicit asTraversable: A => sc.GenTraversableOnce[B])CC[CC[B]]`
* `unzip[A1, A2](implicit asPair: A =>(A1, A2))(CC[A1], CC[A2])`
* `unzip3[A1, A2, A3](implicit asTriple: A =>(A1, A2, A3))(CC[A1], CC[A2], CC[A3])`
* `view(from: Int, until: Int)sc.TraversableView[A,Repr]`
* `view => sc.TraversableView[A,Repr]`
* `withFilter(p: A => Boolean)sc.generic.FilterMonadic[A,Repr]`

`immutable.Traversable extends Traversable` + 0 additional members:

`mutable.Traversable extends Traversable` + 0 additional members:

`Iterator extends TraversableOnce with java.lang.Object` + 33 additional members:
* `++[B >: A](that: => sc.GenTraversableOnce[B])Iterator[B]`
* `buffered => sc.BufferedIterator[A]`
* `collect[B](pf: PartialFunction[A,B])Iterator[B]`
* `contains(elem: Any)Boolean`
* `corresponds[B](that: sc.GenTraversableOnce[B])(p:(A, B) => Boolean)Boolean`
* `drop(n: Int)Iterator[A]`
* `dropWhile(p: A => Boolean)Iterator[A]`
* `duplicate =>(Iterator[A], Iterator[A])`
* `filter(p: A => Boolean)Iterator[A]`
* `filterNot(p: A => Boolean)Iterator[A]`
* `flatMap[B](f: A => sc.GenTraversableOnce[B])Iterator[B]`
* `grouped[B >: A](size: Int)Iterator.this.GroupedIterator[B]`
* `hasNext => Boolean`
* `indexOf[B >: A](elem: B)Int`
* `indexWhere(p: A => Boolean)Int`
* `length => Int`
* `map[B](f: A => B)Iterator[B]`
* `next()A`
* `padTo[A1 >: A](len: Int, elem: A1)Iterator[A1]`
* `partition(p: A => Boolean)(Iterator[A], Iterator[A])`
* `patch[B >: A](from: Int, patchElems: Iterator[B], replaced: Int)Iterator[B]`
* `sameElements(that: Iterator[_])Boolean`
* `scanLeft[B](z: B)(op:(B, A) => B)Iterator[B]`
* `scanRight[B](z: B)(op:(A, B) => B)Iterator[B]`
* `slice(from: Int, until: Int)Iterator[A]`
* `sliding[B >: A](size: Int, step: Int)Iterator.this.GroupedIterator[B]`
* `span(p: A => Boolean)(Iterator[A], Iterator[A])`
* `take(n: Int)Iterator[A]`
* `takeWhile(p: A => Boolean)Iterator[A]`
* `withFilter(p: A => Boolean)Iterator[A]`
* `zip[B](that: Iterator[B])Iterator[(A, B)]`
* `zipAll[B, A1 >: A, B1 >: B](that: Iterator[B], thisElem: A1, thatElem: B1)Iterator[(A1, B1)]`
* `zipWithIndex => Iterator[(A, Int)]`

`Iterable extends scala.Equals with Traversable` + 11 additional members:
* `dropRight(n: Int)Repr`
* `foreach[U](f: A => U)Unit`
* `grouped(size: Int)Iterator[Repr]`
* `iterator => Iterator[A]`
* `sameElements[B >: A](that: sc.GenIterable[B])Boolean`
* `sliding(size: Int, step: Int)Iterator[Repr]`
* `sliding(size: Int)Iterator[Repr]`
* `takeRight(n: Int)Repr`
* `zip[A1 >: A, B, That](that: sc.GenIterable[B])(implicit bf: sc.generic.CanBuildFrom[Repr,(A1, B),That])That`
* `zipAll[B, A1 >: A, That](that: sc.GenIterable[B], thisElem: A1, thatElem: B)(implicit bf: sc.generic.CanBuildFrom[Repr,(A1, B),That])That`
* `zipWithIndex[A1 >: A, That](implicit bf: sc.generic.CanBuildFrom[Repr,(A1, Int),That])That`

`immutable.Iterable extends Iterable with immutable.Traversable` + 0 additional members:

`mutable.Iterable extends Iterable with mutable.Traversable` + 0 additional members:

`Seq extends Iterable with scala.PartialFunction` + 42 additional members:
* `:+[B >: A, That](elem: B)(implicit bf: sc.generic.CanBuildFrom[Repr,B,That])That`
* `+:[B >: A, That](elem: B)(implicit bf: sc.generic.CanBuildFrom[Repr,B,That])That`
* `apply(idx: Int)A`
* `combinations(n: Int)Iterator[Repr]`
* `contains[A1 >: A](elem: A1)Boolean`
* `containsSlice[B](that: sc.GenSeq[B])Boolean`
* `corresponds[B](that: sc.GenSeq[B])(p:(A, B) => Boolean)Boolean`
* `diff[B >: A](that: sc.GenSeq[B])Repr`
* `distinct => Repr`
* `endsWith[B](that: sc.GenSeq[B])Boolean`
* `indexOf[B >: A](elem: B, from: Int)Int`
* `indexOf[B >: A](elem: B)Int`
* `indexOfSlice[B >: A](that: sc.GenSeq[B], from: Int)Int`
* `indexOfSlice[B >: A](that: sc.GenSeq[B])Int`
* `indexWhere(p: A => Boolean, from: Int)Int`
* `indexWhere(p: A => Boolean)Int`
* `indices => immutable.Range`
* `intersect[B >: A](that: sc.GenSeq[B])Repr`
* `isDefinedAt(idx: Int)Boolean`
* `lastIndexOf[B >: A](elem: B, end: Int)Int`
* `lastIndexOf[B >: A](elem: B)Int`
* `lastIndexOfSlice[B >: A](that: sc.GenSeq[B], end: Int)Int`
* `lastIndexOfSlice[B >: A](that: sc.GenSeq[B])Int`
* `lastIndexWhere(p: A => Boolean, end: Int)Int`
* `lastIndexWhere(p: A => Boolean)Int`
* `length => Int`
* `lengthCompare(len: Int)Int`
* `padTo[B >: A, That](len: Int, elem: B)(implicit bf: sc.generic.CanBuildFrom[Repr,B,That])That`
* `patch[B >: A, That](from: Int, patch: sc.GenSeq[B], replaced: Int)(implicit bf: sc.generic.CanBuildFrom[Repr,B,That])That`
* `permutations => Iterator[Repr]`
* `prefixLength(p: A => Boolean)Int`
* `reverse => Repr`
* `reverseIterator => Iterator[A]`
* `reverseMap[B, That](f: A => B)(implicit bf: sc.generic.CanBuildFrom[Repr,B,That])That`
* `segmentLength(p: A => Boolean, from: Int)Int`
* `sortBy[B](f: A => B)(implicit ord: scala.math.Ordering[B])Repr`
* `sortWith(lt:(A, A) => Boolean)Repr`
* `sorted[B >: A](implicit ord: scala.math.Ordering[B])Repr`
* `startsWith[B](that: sc.GenSeq[B], offset: Int)Boolean`
* `startsWith[B](that: sc.GenSeq[B])Boolean`
* `union[B >: A, That](that: sc.GenSeq[B])(implicit bf: sc.generic.CanBuildFrom[Repr,B,That])That`
* `updated[B >: A, That](index: Int, elem: B)(implicit bf: sc.generic.CanBuildFrom[Repr,B,That])That`

`immutable.Seq extends Seq with immutable.Iterable` + 0 additional members:

`mutable.Seq extends Seq with mutable.Iterable` + 3 additional members:
* `clone()A`
* `transform(f: A => A)SeqLike.this.type`
* `update(idx: Int, elem: A)Unit`

`mutable.Builder extends java.lang.Object` + 10 additional members:
* `+=(elem1: A, elem2: A, elems: A*)Growable.this.type`
* `+=(elem: Elem)Builder.this.type`
* `++=(xs: sc.TraversableOnce[A])Growable.this.type`
* `clear()Unit`
* `mapResult[NewTo](f: To => NewTo)mutable.Builder[Elem,NewTo]`
* `result()To`
* `sizeHint(coll: sc.TraversableLike[_, _], delta: Int)Unit`
* `sizeHint(coll: sc.TraversableLike[_, _])Unit`
* `sizeHint(size: Int)Unit`
* `sizeHintBounded(size: Int, boundingColl: sc.TraversableLike[_, _])Unit`

`mutable.Buffer extends mutable.Seq` + 25 additional members:
* `<<(cmd: sc.script.Message[A])Unit`
* `-(elem1: A, elem2: A, elems: A*)This`
* `-(elem: A)This`
* `-=(x: A)BufferLike.this.type`
* `-=(elem1: A, elem2: A, elems: A*)Shrinkable.this.type`
* `--(xs: sc.GenTraversableOnce[A])This`
* `--=(xs: sc.TraversableOnce[A])Shrinkable.this.type`
* `+=(elem1: A, elem2: A, elems: A*)Growable.this.type`
* `+=(elem: A)BufferLike.this.type`
* `+=:(elem: A)BufferLike.this.type`
* `++(xs: sc.GenTraversableOnce[A])This`
* `++=(xs: sc.TraversableOnce[A])Growable.this.type`
* `++=:(xs: sc.TraversableOnce[A])BufferLike.this.type`
* `append(elems: A*)Unit`
* `appendAll(xs: sc.TraversableOnce[A])Unit`
* `clear()Unit`
* `insert(n: Int, elems: A*)Unit`
* `insertAll(n: Int, elems: Traversable[A])Unit`
* `prepend(elems: A*)Unit`
* `prependAll(xs: sc.TraversableOnce[A])Unit`
* `readOnly => Seq[A]`
* `remove(n: Int, count: Int)Unit`
* `remove(n: Int)A`
* `trimEnd(n: Int)Unit`
* `trimStart(n: Int)Unit`

`mutable.ListBuffer extends mutable.Builder with mutable.Buffer` + 1 additional members:
* `prependToList(xs: List[A])List[A]`

`LinearSeq extends Seq` + 0 additional members:

`immutable.LinearSeq extends LinearSeq with immutable.Seq` + 0 additional members:

`mutable.LinearSeq extends LinearSeq with mutable.Seq` + 0 additional members:

`immutable.List extends immutable.LinearSeq` + 8 additional members:
* `::[B >: A](x: B)List[B]`
* `:::[B >: A](prefix: List[B])List[B]`
* `mapConserve[B >: A <: AnyRef](f: A => B)List[B]`
* `productArity => Int`
* `productElement(n: Int)Any`
* `productIterator => Iterator[Any]`
* `productPrefix => String`
* `reverse_:::[B >: A](prefix: List[B])List[B]`

`immutable.Stream extends immutable.LinearSeq` + 4 additional members:
* `append[B >: A](rest: => sc.TraversableOnce[B])immutable.Stream[B]`
* `force => immutable.Stream[A]`
* `print(sep: String)Unit`
* `print()Unit`

`immutable.Queue extends immutable.LinearSeq` + 5 additional members:
* `dequeue =>(A, immutable.Queue[A])`
* `dequeueOption => Option[(A, immutable.Queue[A])]`
* `enqueue[B >: A](iter: immutable.Iterable[B])immutable.Queue[B]`
* `enqueue[B >: A](elem: B)immutable.Queue[B]`
* `front => A`

`mutable.Stack extends mutable.Seq` + 8 additional members:
* `clear()Unit`
* `elems => List[A]`
* `elems_=(x$1: List[A])Unit`
* `pop()A`
* `push(elem1: A, elem2: A, elems: A*)Stack.this.type`
* `push(elem: A)Stack.this.type`
* `pushAll(xs: sc.TraversableOnce[A])Stack.this.type`
* `top => A`

`mutable.ArrayStack extends mutable.Builder with mutable.Seq` + 8 additional members:
* `combine(f:(T, T) => T)Unit`
* `drain(f: T => Unit)Unit`
* `dup()Unit`
* `pop()T`
* `preserving[T](action: => T)T`
* `push(x: T)Unit`
* `reduceWith(f:(T, T) => T)Unit`
* `top => T`

`mutable.PriorityQueue extends mutable.Builder with mutable.Iterable` + 12 additional members:
* `++(xs: sc.GenTraversableOnce[A])mutable.PriorityQueue[A]`
* `clone()mutable.PriorityQueue[A]`
* `dequeue()A`
* `dequeueAll[A1 >: A, That](implicit bf: sc.generic.CanBuildFrom[_, A1, That])That`
* `enqueue(elems: A*)Unit`
* `genericOrderedBuilder[B](implicit ord: Ordering[B])mutable.Builder[B,CC[B]]`
* `length => Int`
* `ord => Ordering[A]`
* `orderedCompanion => mutable.PriorityQueue.type`
* `reverse => mutable.PriorityQueue[A]`
* `reverseIterator => Iterator[A]`
* `toQueue => mutable.Queue[A]`

`mutable.MutableList extends mutable.Builder with mutable.LinearSeq` + 3 additional members:
* `+=:(elem: A)MutableList.this.type`
* `get(n: Int)Option[A]`
* `toQueue => mutable.Queue[A]`

`IndexedSeq extends Seq` + 0 additional members:

`immutable.IndexedSeq extends IndexedSeq with immutable.Seq` + 0 additional members:

`mutable.IndexedSeq extends IndexedSeq with mutable.Seq` + 1 additional members:
* `update(idx: Int, elem: A)Unit`

`immutable.Vector extends immutable.IndexedSeq` + 0 additional members:

`immutable.NumericRange extends immutable.IndexedSeq` + 7 additional members:
* `by(newStep: T)immutable.NumericRange[T]`
* `containsTyped(x: T)Boolean`
* `copy(start: T, end: T, step: T)immutable.NumericRange[T]`
* `end => T`
* `isInclusive => Boolean`
* `start => T`
* `step => T`

`immutable.Range extends immutable.IndexedSeq` + 10 additional members:
* `by(step: Int)immutable.Range`
* `contains(x: Int)Boolean`
* `end => Int`
* `inclusive => immutable.Range`
* `isInclusive => Boolean`
* `lastElement => Int`
* `numRangeElements => Int`
* `start => Int`
* `step => Int`
* `terminalElement => Int`

`immutable.WrappedString extends immutable.IndexedSeq` + 31 additional members:
* `>(that: A)Boolean`
* `>=(that: A)Boolean`
* `<(that: A)Boolean`
* `<=(that: A)Boolean`
* `*(n: Int)String`
* `capitalize => String`
* `compare(other: String)Int`
* `compareTo(that: A)Int`
* `format(args: Any*)String`
* `formatLocal(l: java.util.Locale, args: Any*)String`
* `lines => Iterator[String]`
* `linesIterator => Iterator[String]`
* `linesWithSeparators => Iterator[String]`
* `r(groupNames: String*)scala.util.matching.Regex`
* `r => scala.util.matching.Regex`
* `replaceAllLiterally(literal: String, replacement: String)String`
* `self => String`
* `split(separators: Array[Char])Array[String]`
* `split(separator: Char)Array[String]`
* `stripLineEnd => String`
* `stripMargin => String`
* `stripMargin(marginChar: Char)String`
* `stripPrefix(prefix: String)String`
* `stripSuffix(suffix: String)String`
* `toBoolean => Boolean`
* `toByte => Byte`
* `toDouble => Double`
* `toFloat => Float`
* `toInt => Int`
* `toLong => Long`
* `toShort => Short`

`mutable.WrappedArray extends mutable.IndexedSeq` + 4 additional members:
* `array => Array[T]`
* `deep => IndexedSeq[Any]`
* `elemManifest => ClassManifest[T]`
* `elemTag => scala.reflect.ClassTag[T]`

`mutable.ArraySeq extends mutable.IndexedSeq` + 1 additional members:
* `array => Array[AnyRef]`

`mutable.StringBuilder extends mutable.Builder with mutable.IndexedSeq` + 80 additional members:
* `>(that: A)Boolean`
* `>=(that: A)Boolean`
* `<(that: A)Boolean`
* `<=(that: A)Boolean`
* `+(x: Char)StringBuilder.this.type`
* `++=(s: String)StringBuilder.this.type`
* `*(n: Int)String`
* `append(x: Char)StringBuilder`
* `append(x: Double)StringBuilder`
* `append(x: Float)StringBuilder`
* `append(x: Long)StringBuilder`
* `append(x: Int)StringBuilder`
* `append(x: Short)StringBuilder`
* `append(x: Byte)StringBuilder`
* `append(x: Boolean)StringBuilder`
* `append(sb: StringBuilder)StringBuilder`
* `append(s: String)StringBuilder`
* `append(x: Any)StringBuilder`
* `appendAll(xs: Array[Char], offset: Int, len: Int)StringBuilder`
* `appendAll(xs: Array[Char])StringBuilder`
* `appendAll(xs: sc.TraversableOnce[Char])StringBuilder`
* `appendAll(xs: String)StringBuilder`
* `capacity => Int`
* `capitalize => String`
* `charAt(index: Int)Char`
* `chars()java.util.stream.IntStream`
* `codePoints()java.util.stream.IntStream`
* `compare(other: String)Int`
* `compareTo(that: A)Int`
* `delete(start: Int, end: Int)StringBuilder`
* `deleteCharAt(index: Int)StringBuilder`
* `ensureCapacity(newCapacity: Int)Unit`
* `format(args: Any*)String`
* `formatLocal(l: java.util.Locale, args: Any*)String`
* `indexOf(str: String, fromIndex: Int)Int`
* `indexOf(str: String)Int`
* `insert(index: Int, x: Char)StringBuilder`
* `insert(index: Int, x: Double)StringBuilder`
* `insert(index: Int, x: Float)StringBuilder`
* `insert(index: Int, x: Long)StringBuilder`
* `insert(index: Int, x: Int)StringBuilder`
* `insert(index: Int, x: Short)StringBuilder`
* `insert(index: Int, x: Byte)StringBuilder`
* `insert(index: Int, x: Boolean)StringBuilder`
* `insert(index: Int, x: String)StringBuilder`
* `insert(index: Int, x: Any)StringBuilder`
* `insertAll(index: Int, xs: Array[Char])StringBuilder`
* `insertAll(index: Int, xs: sc.TraversableOnce[Char])StringBuilder`
* `insertAll(index: Int, str: Array[Char], offset: Int, len: Int)StringBuilder`
* `lastIndexOf(str: String, fromIndex: Int)Int`
* `lastIndexOf(str: String)Int`
* `length_=(n: Int)Unit`
* `lines => Iterator[String]`
* `linesIterator => Iterator[String]`
* `linesWithSeparators => Iterator[String]`
* `r(groupNames: String*)scala.util.matching.Regex`
* `r => scala.util.matching.Regex`
* `replace(start: Int, end: Int, str: String)StringBuilder`
* `replaceAllLiterally(literal: String, replacement: String)String`
* `reverseContents()StringBuilder`
* `setCharAt(index: Int, ch: Char)Unit`
* `setLength(len: Int)Unit`
* `split(separators: Array[Char])Array[String]`
* `split(separator: Char)Array[String]`
* `stripLineEnd => String`
* `stripMargin => String`
* `stripMargin(marginChar: Char)String`
* `stripPrefix(prefix: String)String`
* `stripSuffix(suffix: String)String`
* `subSequence(start: Int, end: Int)CharSequence`
* `substring(start: Int, end: Int)String`
* `substring(start: Int)String`
* `toArray => Array[Char]`
* `toBoolean => Boolean`
* `toByte => Byte`
* `toDouble => Double`
* `toFloat => Float`
* `toInt => Int`
* `toLong => Long`
* `toShort => Short`

`mutable.ArrayBuffer extends mutable.IndexedSeq with mutable.Builder with mutable.Buffer` + 1 additional members:
* `reduceToSize(sz: Int)Unit`

`Set extends Iterable with scala.Function1` + 17 additional members:
* `&(that: sc.GenSet[A])Repr`
* `&~(that: sc.GenSet[A])Repr`
* `|(that: sc.GenSet[A])Repr`
* `-(elem1: A, elem2: A, elems: A*)Repr`
* `-(elem: A)This`
* `--(xs: sc.GenTraversableOnce[A])Repr`
* `+(elem1: A, elem2: A, elems: A*)This`
* `+(elem: A)This`
* `++(elems: sc.GenTraversableOnce[A])This`
* `contains(elem: A)Boolean`
* `diff(that: sc.GenSet[A])This`
* `empty => CC[A]`
* `intersect(that: sc.GenSet[A])Repr`
* `subsetOf(that: sc.GenSet[A])Boolean`
* `subsets()Iterator[This]`
* `subsets(len: Int)Iterator[This]`
* `union(that: sc.GenSet[A])This`

`immutable.Set extends Set with immutable.Iterable` + 0 additional members:

`mutable.Set extends mutable.Builder with Set with mutable.Iterable` + 9 additional members:
* `<<(cmd: sc.script.Message[A])Unit`
* `-=(elem1: A, elem2: A, elems: A*)Shrinkable.this.type`
* `-=(elem: A)SetLike.this.type`
* `--=(xs: sc.TraversableOnce[A])Shrinkable.this.type`
* `add(elem: A)Boolean`
* `clone()This`
* `remove(elem: A)Boolean`
* `retain(p: A => Boolean)Unit`
* `update(elem: A, included: Boolean)Unit`

`immutable.HashSet extends immutable.Set` + 1 additional members:
* `updated0(key: A, hash: Int, level: Int)immutable.HashSet[A]`

`immutable.BitSet extends BitSet with immutable.SortedSet` + 0 additional members:

`immutable.ListSet extends immutable.Set` + 0 additional members:

`mutable.HashSet extends mutable.Set` + 2 additional members:
* `initialSize => Int`
* `useSizeMap(t: Boolean)Unit`

`mutable.LinkedHashSet extends mutable.Set` + 0 additional members:

`SortedSet extends Set` + 12 additional members:
* `compare(k0: K, k1: K)Int`
* `firstKey => A`
* `from(from: A)This`
* `iteratorFrom(start: A)Iterator[A]`
* `keySet => This`
* `keysIteratorFrom(start: K)Iterator[K]`
* `lastKey => A`
* `ordering => Ordering[A]`
* `range(from: A, until: A)This`
* `rangeImpl(from: Option[A], until: Option[A])This`
* `to(to: K)This`
* `until(until: A)This`

`immutable.SortedSet extends SortedSet with immutable.Set` + 0 additional members:

`mutable.SortedSet extends mutable.Set with SortedSet` + 0 additional members:

`immutable.TreeSet extends immutable.SortedSet` + 1 additional members:
* `insert(elem: A)immutable.TreeSet[A]`

`mutable.TreeSet extends mutable.SortedSet` + 0 additional members:

`BitSet extends SortedSet` + 6 additional members:
* `&(other: sc.BitSet)This`
* `&~(other: sc.BitSet)This`
* `|(other: sc.BitSet)This`
* `^(other: sc.BitSet)This`
* `subsetOf(other: sc.BitSet)Boolean`
* `toBitMask => Array[Long]`

`immutable.BitSet extends BitSet with immutable.SortedSet` + 0 additional members:

`mutable.BitSet extends BitSet with mutable.SortedSet` + 5 additional members:
* `&=(other: mutable.BitSet)BitSet.this.type`
* `&~=(other: mutable.BitSet)BitSet.this.type`
* `|=(other: mutable.BitSet)BitSet.this.type`
* `^=(other: mutable.BitSet)BitSet.this.type`
* `toImmutable => immutable.BitSet`

`Map extends scala.PartialFunction with Iterable` + 19 additional members:
* `-(elem1: A, elem2: A, elems: A*)Repr`
* `-(key: A)This`
* `--(xs: sc.GenTraversableOnce[A])Repr`
* `+[B1 >: B](kv1:(A, B1), kv2:(A, B1), kvs:(A, B1)*)sc.Map[A,B1]`
* `+[B1 >: B](kv:(A, B1))sc.Map[A,B1]`
* `++[B1 >: B](xs: sc.GenTraversableOnce[(A, B1)])sc.Map[A,B1]`
* `contains(key: A)Boolean`
* `default(key: A)B`
* `empty => sc.Map[A,B]`
* `filterKeys(p: A => Boolean)sc.Map[A,B]`
* `get(key: A)Option[B]`
* `getOrElse[B1 >: B](key: A, default: => B1)B1`
* `keySet => sc.Set[A]`
* `keys => Iterable[A]`
* `keysIterator => Iterator[A]`
* `mapValues[C](f: B => C)sc.Map[A,C]`
* `updated[B1 >: B](key: A, value: B1)sc.Map[A,B1]`
* `values => Iterable[B]`
* `valuesIterator => Iterator[B]`

`immutable.Map extends Map with immutable.Iterable` + 3 additional members:
* `transform[C, That](f:(A, B) => C)(implicit bf: sc.generic.CanBuildFrom[This,(A, C),That])That`
* `withDefault[B1 >: B](d: A => B1)immutable.Map[A,B1]`
* `withDefaultValue[B1 >: B](d: B1)immutable.Map[A,B1]`

`mutable.Map extends mutable.Builder with Map with mutable.Iterable` + 12 additional members:
* `-=(elem1: A, elem2: A, elems: A*)Shrinkable.this.type`
* `-=(key: A)MapLike.this.type`
* `--=(xs: sc.TraversableOnce[A])Shrinkable.this.type`
* `clone()This`
* `getOrElseUpdate(key: A, op: => B)B`
* `put(key: A, value: B)Option[B]`
* `remove(key: A)Option[B]`
* `retain(p:(A, B) => Boolean)MapLike.this.type`
* `transform(f:(A, B) => B)MapLike.this.type`
* `update(key: A, value: B)Unit`
* `withDefault(d: A => B)mutable.Map[A,B]`
* `withDefaultValue(d: B)mutable.Map[A,B]`

`immutable.HashMap extends immutable.Map` + 2 additional members:
* `merged[B1 >: B](that: immutable.HashMap[A,B1])(mergef: immutable.HashMap.MergeFunction[A,B1])immutable.HashMap[A,B1]`
* `split => immutable.Seq[immutable.HashMap[A,B]]`

`immutable.ListMap extends immutable.Map` + 0 additional members:

`mutable.HashMap extends mutable.Map` + 1 additional members:
* `useSizeMap(t: Boolean)Unit`

`mutable.OpenHashMap extends mutable.Map` + 0 additional members:

`mutable.WeakHashMap extends mutable.Map` + 1 additional members:
* `underlying => java.util.Map[A,B]`

`mutable.ListMap extends mutable.Map` + 0 additional members:

`SortedMap extends Map` + 12 additional members:
* `compare(k0: K, k1: K)Int`
* `firstKey => A`
* `from(from: K)This`
* `iteratorFrom(start: A)Iterator[(A, B)]`
* `keysIteratorFrom(start: K)Iterator[K]`
* `lastKey => A`
* `ordering => Ordering[A]`
* `range(from: K, until: K)This`
* `rangeImpl(from: Option[A], until: Option[A])This`
* `to(to: K)This`
* `until(until: K)This`
* `valuesIteratorFrom(start: A)Iterator[B]`

`immutable.SortedMap extends SortedMap with immutable.Map` + 0 additional members:

`immutable.TreeMap extends immutable.SortedMap` + 1 additional members:
* `insert[B1 >: B](key: A, value: B1)immutable.TreeMap[A,B1]`

`concurrent.Map extends mutable.Map` + 4 additional members:
* `putIfAbsent(k: A, v: B)Option[B]`
* `remove(k: A, v: B)Boolean`
* `replace(k: A, v: B)Option[B]`
* `replace(k: A, oldvalue: B, newvalue: B)Boolean`

`concurrent.TrieMap extends concurrent.Map` + 14 additional members:
* `CAS_ROOT(ov: AnyRef, nv: AnyRef)Boolean`
* `RDCSS_READ_ROOT(abort: Boolean)sc.concurrent.INode[K,V]`
* `computeHash(k: K)Int`
* `equality => Equiv[K]`
* `hashing => scala.util.hashing.Hashing[K]`
* `isReadOnly => Boolean`
* `lookup(k: K)V`
* `nonReadOnly => Boolean`
* `readOnlySnapshot()sc.Map[K,V]`
* `readRoot(abort: Boolean)sc.concurrent.INode[K,V]`
* `root => AnyRef`
* `root_=(x$1: AnyRef)Unit`
* `snapshot()sc.concurrent.TrieMap[K,V]`
* `string => String`