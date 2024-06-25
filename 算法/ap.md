

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