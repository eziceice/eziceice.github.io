---
title: Effective Java
date: 2019-04-30 12:03:56
tags: java
categories:
- programming languages
---
# Chapter 2. Creating and Destorying Objects
## Consider static factory methods instead of constructors
**Advantage:**

- Static factory has name which is better for people to understand it compared to constructors. If you want to create two constructors with different signature, it's better to use static factory as using the different method name is simpler for other people to distinguish the difference.

- Unlike constructors, static factory is not required to create a new object each time they're invoked. This could improve the performance if the equivalent objects are requested often. Similar to singleton. This will also improve the performance of equals() - Don't need to override hasCode() or hash() anymore.

- Static factory could return an object of any subtype of their return type. Constructor only can return the current object. This improves the flexibility of  choosing the class of the returned object. **Java 8 requires all static members of an interface to be public. Java 9 allows private static methods, but static fields and static member classes are still required to be public.**
```
Noninstantiable class - It can be initialized by using constructor or reflection. 

public class A {
    private A() throws Exception {
        throw new Exception();
    }
}
```

- Static factory could return different object based on the different input parameters. Java library EnumSet is a good example for this.
```
   public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
        Enum<?>[] universe = getUniverse(elementType);
        if (universe == null)
            throw new ClassCastException(elementType + " not an enum");

        if (universe.length <= 64)
            return new RegularEnumSet<>(elementType, universe);
        else
            return new JumboEnumSet<>(elementType, universe);
    }
``` 

- The Static factory class of the returned object doesn't need to exist when the class containing the method is written.   
  Service provider framework:

    - Component 1: Service Interface, which represents an implementation.
    - Component 2: Provider registration API, which provides use to register implementations.
    - Component 3: Service access API, which clients use to obtain instances of the service. This may allow clients to specify criteria for choosing an implementation. In the absence of such criteria, the API returns an instance of a default implementation, or allows the client to cycle through all available implementations. **The service access API is the flexible static factory that forms the basis of the service provider framework.**

**Limitation**

- Static factory methods cannot be extended compared to public or protected constructor. A class can extend from its parent class but the subclass cannot override its parent class static method. A static method is belonged to the class itself, not belonged to any instance of this class. In addition, it is defined in the compile time, not in the run time.

-  Normally a static method is hard for programmers to find. Constructor is very easy for programmers to distinguish the difference. It's better to put the appropriate Java doc there for other people to know this is a static factory method.

## Consider a builder when faced with many constructor parameters

- **Static factories and constructors share a limitation: they do not scale well to large numbers of optional parameters.** There are two alternative ways but still they have some limitations:

1. Sometimes the telescoping constructor pattern works, but it is hard to write client code when there are many parameters, and harder still to read it.
2. An alternative way is using Java bean, but again, you need to call setter several times, which prevents the possibility of the immutable class or need to ensure thread safety.

- **Build Pattern:** The NutritionFacts class is immutable, and all parameters default values are in one place. A single builders can have multiple varargs(可变参数) parameters because each parameter is specified in its own method. It is quite flexible and a single builder can be used to build multiple objects.
```
// Builder Pattern
public class NutritionFacts {
	
	private final int servingSize;
	private final int servings;
	private final int calories;
	private final int fat;
	private final int sodium;
	private final int carbohydrate;

	public static class Builder {
		// Required parameters
		private final int servingSize;
		private final int servings;
		// Optional parameters - initialized to default values
		private int calories = 0;
		private int fat = 0;
		private int sodium = 0;
		private int carbohydrate = 0;

		public Builder(int servingSize, int servings) {
			this.servingSize = servingSize;
			this.servings = servings;
		}

		public Builder calories(int val) { 
			calories = val; return this; 
		}

		public Builder fat(int val) { 
			fat = val; 
			return this; 
		}

		public Builder sodium(int val) { 
			sodium = val; 
			return this; 
		}

		public Builder carbohydrate(int val) { 
			carbohydrate = val; 
			return this; 
		}
		public NutritionFacts build() {
			return new NutritionFacts(this);
		}
	}
	
	private NutritionFacts(Builder builder) {
		servingSize = builder.servingSize;
		servings = builder.servings;
		calories = builder.calories;
		fat = builder.fat;
		sodium = builder.sodium;
		carbohydrate = builder.carbohydrate;
	}	
}
```

- Builder Pattern is not suitable for less parameters (less than 5), it can cause some performance issues. However, it is still better than using the telescoping constructors because normally the parameters will go large in the end. Moreover, it is really helpful for the optional parameters.

## Enforce the singleton property with a private constructor or an enum type

- Normally we use private constructor and static factory to implement singleton. However, if you have a singleton object, you do serialize and de-serialize on it, you could get two singleton objects. To maintain the singleton guarantee, declare all instance fields transient and provide a readResolve method.
  **The best way to preserve singleton is define the singleton object as a Enum. The limitation for this approach is you can't extent it from a superclass except Enum**
```
// Enum singleton - the preferred approach
public enum Elvis {
	INSTANCE;
	public void leaveTheBuilding() { ... }
}
```

## Enforce noninstantiability with a private constructor

- For some utility classes, normally people don't want them to be initialized and maintain them as static class. Attempting to enforce noninstantiability by making a class abstract does not work.  The best way is to add a private constructor and let it throw an exception. This will guarantee the class will never be instantiated under any circumstances.  As a side effect, this constructor also prevents the class from being subclassed.
```
// Noninstantiable utility class
public class UtilityClass {
	// Suppress default constructor for noninstantiability
	private UtilityClass() {
	throw new AssertionError();
	}
	... // Remainder omitted
}
```

## Prefer dependency injection to handwriting resources

-  Static utility classes and singletons are inappropriate for classes whose behavior is parameterized by an underlying resource.
```
// Pass the dependency to the constructor to initial different field in that class - Dependency injection
public class SpellChecker {
    private final Lexicon dictionary;
    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }
    public boolean isValid(String word) { ... }
    public List<String> suggestions(String typo) { ... }
}
```
In conclusion, do not use a singleton or static utility class to implement a class that depends on one or more underlying resources whose behavior affects that of the class, and do not have the class create these resources directly. Instead, pass the resources, or factories to create them, into the constructor (or static factory or builder).

## Avoid creating unnecessary objects

- An object can always be reused if it is immutable. You can often avoid creating unnecessary objects by using static factory methods in preference to constructors on immutable classes that provide both. In addition to reusing immutable objects, you can also reuse mutable objects if you know they won't be modified. For example, it is better to use Boolean.valueOf(String) instead of use Boolean(String) as the second method always create a new object in memory.


```
// The performance can be greatly improved because every time this method is called, 
// a new Pattern is created and the regex is being compiled. It takes a lot of time. 
    static boolean isRomanNumeral(String s) {
        return s.matches("^(?=.)M*(C[MD]|D?C{0,3})"
                + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
    }

// The improvement version here, there will be only one Pattern which can be reused many times
// It also improves the readability of this class 
   private static final Pattern ROMAN =
            Pattern.compile(
                    "^(?=.)M*(C[MD]|D?C{0,3})"
                            + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
    static boolean isRomanNumeral(String s) {
        return ROMAN.matcher(s).matches();
    }

```

- Be careful of using autoboxing. Sometimes it can cause the performance issue because of creating some unnecessary objects.
```
// Try to use primitive type instead of autoboxing.
private static long sum() {
        Long sum = 0L; // many Long objects will be created in the loop, should use long instead
        for (long i = 0; i <= Integer.MAX_VALUE; i++)
              sum += i;
        return sum;
}
```

## Eliminate obsolete object references

- Consider this part code below: In a very rare situation, it may cause the failure with **OutOfMemoryError**, which can be considered as a memory leak.
```
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY =
            16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size];
    }

    /**
     * Ensure space for at least one more element,
     * roughly
     * doubling the capacity each time the array needs
     * to grow.
     */
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

- The reason above previous class is that if a stack grows and then shrinks, the objects that were popped off the stack will not be garbage collected, even if the program using the stack has no more references to them. This is because the stack maintain **obsolete references** to these projects. An obsolete reference is simply a reference that will never be referenced again.  The fix for this sort of problem is simple: **null out references once they become obsolete**.
```
public Object pop(){
    if(size==0)
        throw new EmptyStackException();
    Object result=elements[--size];
    elements[size]=null; // Eliminate obsolete reference
    return resul;
}
```

- Generally speaking, **whenever a class manages its own memory, the programmer should be alert for memory leaks**. Whenever an element is freed, any object references contained in the element should be nulled out.

- Another common source of memory leaks is **caches**. Sometimes the object references into a cache are always forgotten by programmers. Behaviors such as using WeakHashMap, ScheduledThreadPoolExecutor or LinkedHashMap should be used to ensure the caches will be cleaned after the objects are no longer used.

- A third common source of memory leaks is **listeners and other callbacks**. One way to ensure that callbacks are garbage collected promptly is to store only weak references to them. For instance, by storing them only as keys in a WeakHashMap.

## Avoid finalizers and cleaners

- In Java 9, Cleaners are created to replace the Finalizers. Although Cleaners are less dangerous than Finalizers, but still unpredictable, slow and generally unnecessary. There is no guarantee they'll be executed promptly. You should never depend on a finalizer or cleaner to update persistent state. In addition, calling finalizer may throw an exception, which could leave the object in a corrupt state and terminate the application. Moreover, there is a severe performance penalty for using finalizers and cleaners. Next, Finalizers have a serious security problem, they open your class up to finalizer attacks. If you really want to use Finalizers, just have your class implement AutoCloseable.

## Prefer try-with-resources to try-finally

- **Conclusion: always use try-with-resources instead of try-finally!!**

- Try-catch-finally is not bad currently, but it is hard to read when you try to read multiple resources, the syntax is a disaster. In addition, in the code below, if there is an exception happened in the read method and then exception happened in the close method again. Under these circumstances, the second exception completely obliterates the first one and there is no record of the first exception in the exception stack trace.
```
// resource needs to be closed in different finally block. It's hard to read and understand. 
static void copy(String src, String dst) throws IOException {
    InputStream in = new FileInputStream(src);
    try {
        OutputStream out = new FileOutputStream(dst);
        try {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        } finally {
            out.close();
        }
    } finally {
        in.close();
    }
}
```

- try-with-resources handle this situation easily. It is shorter and more readable than the originals and it will suppress the exception in the invisible close method. In fact, multiple exceptions may be suppressed in order to preserve the exception that you actually want to see.  In addition, you still can add catch block in try-with resources clause, and it will catch the exception you want and do what you want.
```
static String firstLineOfFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))){
        return br.readLine();
    }
}

static void copy(String src, String dst) throws IOException {
    try (InputStream in = new FileInputStream(src); OutputStream out = new FileOutputStream(dst)) {
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while ((n = in.read(buf)) >= 0)
            out.write(buf, 0, n);
    }
}
```
