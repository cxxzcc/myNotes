

# Reflection



## Computational System

### Definition

Computational System: a system that reasons about and acts upon a given domain.

The domain is represented by the internal structures of the system: 
* Data representing entities and relations. 
* Program prescribing data manipulation.

> Notes
>	A program is not a computational system. 
>	A program describes (part of) a computational system. 
>	A running program is a computational system.

## Computational Meta-System 
### Definition 

Computational Meta-System: a computational system that has as domain another computational system (called the Object System). 

A computational meta-system operates on data that represents the computational object-system. 

>Examples 
	A debugger is a computational meta-system. 
	A profiler is a computational meta-system. 
	A (classic) compiler is not a computational meta-system (its domain is a program and not a computational system)


## Reflection 
### Definition 

Reflection: the process of reasoning about and/or acting upon oneself. 

Reflective System: a meta-system that has itself as object-system. 

A reflective system is a system that can represent and manipulate its own structure and behavior at run time

Two levels of Reflection
* ==Introspection==: the ability of a program to examine its own structure and behavior
* ==Intercession==: the ability of a program to modify its own structure and behavior.

>Examples 
>	Introspection: How many parameters has the function foo? Intercession: Change the class of this instance to Bar!



## Reification 

### Definition 

Reification: the creation of an entity that represents, in the meta-system, an entity of the object-system. Reification is a pre-condition for reflection.

>Examples 
>	What is the class of this instance? ⇒ reification of classes. Which are the methods of this class? ⇒ reification of methods. What was the call chain that caused this bug? ⇒ reification of the stack. Which are the values of the free variables of this function? ⇒ reification of the lexical environment.

Two levels of Reification 
* ==Structural Reification==: the ability of a system to reify its own structure. 
* ==Behavioral (or computational) Reification==: the ability of a system to reify its own execution.

> Notes
> 	Behavioral reification is harder to implement than structural reification. 
> 	Intercession over behavioral reification makes compilation harder.


## Reflection in Java
Types 
* Primitive Types: boolean, byte, short, int, long, char, float, and double. 
* Reference Types: java.lang.String, java.io.Serializable, java.lang.Integer, and all the others.

Reified Types 
* For each (primitive or reference) type, there is an (unique) instance of the class java.lang.Class that represents that type. 
* The java.lang.Class class contains methods that: 
	* provide information about the class (methods, variables, etc.), 
	* create instances of the class, 
	* change variables and call methods.


To obtain an instance of java.lang.Class 
* From an object foo: foo.getClass() 
* From a type Bar: Bar.class 
* From the name of a type "foo.bar.Baz" (if not found, throws the Checked exception ClassNotFoundException): Class.forName("foo.bar.Baz")

> Example 
> 	"I am a string".getClass() 
> 	String.class 
> 	Class.forName("java.lang.String")

### Javassist

* Load-time intercession for Java. 
* Does not modify the runtime or compiler. 
* Modifies class bytecodes at class load-time.

Operation Sequence 
* Reification: Creating a CtClass (Compile time Class) object representing the bytecodes of a class. 
* Modification: Introspecting and altering the class definition. 
* Translation: Computing the bytecodes of the modified class. 
* Reflection: Loading the obtained bytecodes into the JVM or rewriting them to class files








# CLOS
Common Lisp Object System



Function Call
```lisp
Functional and Imperative Programming 
(foo a b) 
(call (function 'foo) a b)

Object-Oriented Programming - Single Dispatch 
(foo a b) ⇔ a.foo(b)
(call (function 'foo(type-of a)) a b)

Object-Oriented Programming - Multiple Dispatch 
(foo a b)
(call (function 'foo(type-of a)(type-of b)) a b)

;test
(defgeneric add(x y)) 
(defmethodadd ((x number)(y number)) 
	(+ x y))

> (add 1 3) 
4

(defmethodadd ((x list)(y list)) 
	(mapcar #'add x y)) 

>(add '(1 2 3) '(4 5 6)) 
(5 7 9) 
>(add '(1 (2 3)) '(4 (5 6))) 
(5 (7 9))


(defmethod add ((x list) (y t)) 
	(add x (make-list (length x) :initial-element y)))
 
> (add '(1 2) 3) 
(4 5)

(defmethod add ((x t) (y list)) 
	(add (make-list (length y) :initial-element x) y)) 
	
> (add 1 '(2 3)) 
(3 4)

(defmethod add ((x vector) (y vector)) 
	(map 'vector #'add x y)) 

> (add #(1 2 3) #(4 5 6)) 
#(5 7 9)

(defmethod add ((x vector) (y number)) 
	(add x (make-array (list (length x)) :initial-element y))) 
	
> (add #(1 2 3) 4) 
#(5 6 7)


```


## Generic Functions

Generic Function Application
1. Computes the effective method.
2. If it exists,calls the effective method with the same arguments of the generic function call.
3. If it does not exist,calls no-applicable-method using,as arguments, the original generic function and the arguments of the original call.

Effective Method Computation
1. Selects the applicable methods.
2. Sorts applicable methods by precedence order.
3. Combines applicable methods,producing the effective method.



Applicable Methods 
* Given a generic function and required arguments a0,…,an, an applicable method is a method whose parameter specializers p0,…,pn are satisfied by their corresponding arguments. 
* A parameter specializer pi is satisfied by their corresponding argument ai if (typep ai 'pi)

Qualifiers 
* Each method can have zero or more qualifiers. 
* Each qualifier can be any object except a list (so that qualifiers can be distinguished from parameter lists). 
* A standard method combination distinguishes: 
	* Primary Methods: non-qualified methods. 
	* Auxiliary Methods: methods qualified with the symbol :before, :after, or :around. 
* Other method combinations can use other categories. 
* New method combinations can be defined.

Sorting the Applicable Methods 
* Sorts applicable methods by precedence order from most specific to least specific. 
* Given two applicable methods: 
	* Their parameter specializers are examined in order (by default, from left to right). 
	* When two specializers differ, the highest precedence method is the one whose parameter specializer occurs first in the class precedence list of the corresponding argument. 
	* When one specializer is an instance specializer ((eql object)), the highest precedence method is the one whose parameter contains that specializer. 
	* When all specializers are identical, the two methods must have different qualifiers and either one can be selected to precede the other.


### Method Combination 
* Happens after selecting and sorting applicable methods. 
* Creates the effective method that will be applied to the generic function arguments. 
* There are many pre-defined method combinations (known as method combination types): 
	* Simple append, nconc, list, progn, max, min, +, and, or Requires using the same method combination type in the generic function and all methods of the generic function. 
	* Standard standard Used by default when nothing is specified on the generic function. Implicitly used when the generic function is not specified.

#### Standard Method Combination 
* Primary methods define the main action of the effective method. 
	* Only the most specific is (automatically) executed. 
	* It can execute the next most specific method using call-next-method. 
* Auxiliary methods modify that behavior: 
	* :before Methods called before primary methods. 
	* :after Methods called after primary methods. 
	* :around Method called instead of other applicable methods but that can call some of them by using call-next-method.

If there are no applicable :around methods: 
1. All :before methods are called, from most specific to least specific, and their values are ignored. 
2. The most specific primary method is called. 
	1. If that method calls call-next-method, the next most specific method is called and their values are returned to the caller. 
	2. The values returned by the most specific primary method become the values returned by the generic function call. 
3. All :after methods are called, from least specific to most specific, and their values are ignored.

If there are applicable :around methods, the most specific one is called. If that method calls call-next-method: 
1. If there are more applicable :around methods, the next most specific :around method is called. 
2. If there are no more applicable :around methods: 
	1. All :before methods are called, from most specific to least specific, and their values are ignored. 
	2. The most specific primary method is called. 
		1. If that method calls call-next-method, the next most specific method is called and their values are returned to the caller. 
		2. The values returned by the most specific primary method become the values returned by the generic function call. 
	3. All :after methods are called, from least specific to most specific, and their values are ignored.



* call-next-method might be called 
	* without arguments: it uses the same arguments that were used in the method call. 
	* with arguments: it uses the provided arguments but these should produce the same ordered sequence of applicable methods that was produced by the arguments used in the method call. 
* If there are no more applicable methods, call-next-method calls the generic function no-next-method using, as arguments: 
	* The generic function that contains the method that called call-next-method. 
	* The method that called call-next-method. 
	* The arguments that were used for calling call-next-method. 
* next-method-p can be used in a method to determine whether a next applicable method exists.


#### Simple Method Combination 
* Primary Methods: methods qualified with the combination type (append, nconc, list, progn, max, min, +, and, or). 
* Auxiliary Methods: methods qualified with :around.

If there are no applicable :around methods: 
1. The effective method is the application of the combination type (the operator) to the results of calling all the applicable primary methods sorted by precedence order

If there are applicable :around methods, the most specific one is called. If that method calls call-next-method: 
1. If there are more applicable :around methods, the next most specific :around method is called. 
2. If there are no more applicable :around methods: 
	1. The effective method is the application of the combination type (the operator) to the results of calling all the applicable primary methods sorted by precedence order.



User-Defined Method Combination
Definition
* Name of the method combination. 
* Parameters of the method combination (e.g., sorting order for applicable methods). 
* Local variable to contain methods whose qualifiers … 
* …satisfy this pattern 
* Calls each applicable method in the effective method



# Meta-Circular Evaluators

Linguistic Abstraction
A new programming language is invented to facilitate thedescription of the problem


Evaluator 
An evaluator (or interpreter) of a language is a procedure that, when applied to an expression of that language, computes the operations described in that expression. 

Fundamental Ideia 
An evaluator is a program that computes the meaning of a program. 

Meta-Circular Evaluator 
An evaluator written in the language that it evaluates

Abstract Syntax 
Our evaluator will operate in terms of an abstract syntax: the syntax is recognized by predicates that abstract from the concrete syntax used. 
* Example: Concrete Syntax for Assignments
	*  a := b a = b a ← b (set! a b) 
* Example: Abstract Syntax for Assignments 
	* (define (set? expression) (eq? (car expression) 'set!))

Naming 
* Naming is the fundamental abstraction operator. 
* A name abstracts a value and creates a scope where the name has a meaning. 
* A name is not a first class value but denotes a first class value. 
* Names are maintained in an environment: a collection of names. 
* The environment is implemented as an association list: ((name0.value0)(name1.value1)…(namen.valuen))

Pre-defined Names 
* The REPL starts in a clean state. 
* Forcing us to introduce all needed names. 
* Make it better: we can provide some pre-defined names.

flet 
* Augment the environment to contain associations between the names and the functions. 
call 
* Obtain the function associated with the name. 
* Evaluate all argument expressions. 
* Augment the environment to contain the associations between function parameters and the evaluated arguments. 
* Evaluate the function body in the augmented environment.




## Dynamic Scope

### Downwards Funarg Problem

* Passing a function with free variables as argument causes the downwards funarg problem… 
* … where the function is called in an environment where the free variables might be shadowed by other variables of the calling context.

Solving
* Functions must know the correct environment for resolving the free variables. 
* One potential solution might be to use name-mangling techniques to avoid name clashes.

Downwards Funarg Problem 
* The correct solution is to associate the downward function to the scope where the function was created:
	* That scope is still active when the passed function is called…
	* …unless it is possible to store the function somewhere and call it later. 
* Some languages solve this last problem by restricting the language: 
	* Pascal forbids storing functions in variables. 
	* C forbids inner functions. 
* Other languages (e.g., Scheme, Haskel, Common Lisp) implement environments with indefinite extent. 
* But there is a more difficult problem



### Upwards Funarg Problem 
* Returning a function with free variables as result causes the upwards funarg problem… 
* … where the function is called in an environment where the free variables are not longer bound (or are bound to different values)

Solving the Upwards Funarg Problem 
* Some languages avoid the upwards funarg problem by restricting the language: 
	* Pascal forbids functional returns. 
	* C forbids inner functions. 
	* Java forbids non-final free variables in anonymous inner classes. 
* Other languages (e.g., Scheme, Haskel, Common Lisp) implement environments with indefinite extent.






reasons about 推理
acts upon     操作
Program prescribing 规定 data manipulation 操作  
reflective  反射
examine     检查
Introspection   自省
Intercession    干预
Reification     具体化
lexical         词汇的
reify           具体化
purpose          目的
Inheritance      继承
Generic Functions  泛型函数
Functional and Imperative Programming 函数式和命令式编程
Object-Oriented Programming 面向对象编程