

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