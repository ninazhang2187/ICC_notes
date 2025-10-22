# Day 1
import java.util.Collections;
import java.util.List;

public class Singleton {
   //why singleton class? instance initiates once only for whole runtime

   // final(immutable=change?)     field, method, class-> finalize -> finally
 // finally: implement anyway, exception handling
// finalize: implement is before the JVM compiler
   // if you put final keyword on field
   // it cannot be changed
   // Employee e1 = new Employee("Nina", 24, "female", "NG");
   // e1 = new Employee("Nina", 22, "female", "NG"); update the content
   // the reference/address cannot be changed with final keyword


   //two modes: lazy loading and eager loading (implement differently)
   //object too big not use it every time -> lazy loading
   //big but gonna used -> eager loading
   //Use eager loading when:
   // ✅ The singleton will definitely be used. Example: Logger, DatabaseConnectionPool, ConfigLoader.
   // ✅ You want a simple, thread-safe implementation.
   // ✅ Startup cost is not an issue, and memory is not tight.
   // ✅ You want to avoid synchronization or locking logic.

   //Use lazy loading when: instance created only when needed
   //✅ You’re not sure if the singleton will ever be needed.
   //Example: ErrorReporter, HeavyCache, LargeImageLoader.
   //✅ Creating the instance is resource-intensive (e.g. memory, database).
   //✅ You want to delay heavy work until it's really needed (optimize startup).
   //❗ You're okay adding extra complexity for thread-safety, or you’re in a single-threaded environment.

   // how lazy -> eager
   // 1. private static Singleton instance = new Singleton();
   // 2. no null check
   // 3. add final:private static final Singleton instance = new Singleton(); why final?
   // we just gave the instance, but we assign the value later in the getInstance()
   // private final static Singleton instance; create instance reference, after I created,
   // you cannot reassign it to other object






   //immutability?
   //lazy loading: object/reference is not immutable
   //use get method to get the singleton instance to make sure it is immutable
   //use the getter and setter method to control the access of elts within the class

   // lazy loading:
   // public class Singleton {
   //    private static Singleton instance;
   //
   //    private Singleton() {}
   //
   //    public static Singleton getInstance() {
   //        if (instance == null) {
   //            instance = new Singleton();  // Created only when needed
   //        }
   //        return instance;
   //    }
   //}

   //eager loading:
   //public class Singleton {
   //    private static final Singleton instance = new Singleton();  // Created when class is loaded
   //
   //    private Singleton() {}
   //
   //    public static Singleton getInstance() {
   //        return instance;
   //    }
   //}

   // final method: final keyword make sure you cannot override the method
   // compile time -> overloading
   // runtime -> overriding 

   //Override = subclass redefines a method from superclass
   //same method name,parameters((method signature), return type
   // = in-between classes
   //class Animal {
   //    void speak() {
   //        System.out.println("Animal speaks");
   //    }
   //}
   //
   //class Dog extends Animal {
   //    @Override
   //    void speak() {
   //        System.out.println("Dog barks");
   //    }
   //}
   // Must be in a subclass;
   // same method name,parameters((method signature), return type
   // Run-time polymorphism (dynamic dispatch)

   //Overload = same method name, but different parameters in the same class
   // = within the same class
   //class Calculator {
   //    int add(int a, int b) {
   //        return a + b;
   //    }
   //
   //    double add(double a, double b) {
   //        return a + b;
   //    }
   //
   //    int add(int a, int b, int c) {
   //        return a + b + c;
   //    }
   //}
   //Same class (or inherited);
   //same method name, different input parameters(signatures)(name or data type), return type can be different,
   //Compile-time polymorphism (method resolution at compile time)


   // final class : final keyword makes sure you cannot extend the class

   // employee class -> how immutable class?
   // final keyword on class name
   // private final field(int age;string name;List<>;
   // getter only; no setter
   // in getter, given reference data type field, always return deep copy dummies
   // (for loop) copy all the elts in the list return the dummy list
   // or unmodified list in the collections

   //// Deep copy: return a new list
   //    public List<String> getSkills() {
   //        List<String> copy = new ArrayList<>();
   //        for (String skill : this.skills) {
   //            copy.add(skill);
   //        }
   //        return copy;
   //    }

   //unmodified list:
   // add this to constructor: this.skills = List.copyOf(skills);  // Java 10+, safe unmodifiable copy
   // or after getter other fields do:
   // Return unmodifiable list (safe from external change)
   //    public List<String> getSkills() {
   //        return Collections.unmodifiableList(skills);
   //    }

   //public : access modifier
   //4 access modifier: public, private, protected, (default) (no keyword, also called "package-private")
   //public -> widest access level
   //          Accessible from: Anywhere (any class, any package). eg:public int number = 10;
   //                 Use case: When you want the class/member to be available universally.
   //private -> narrowest scope of elements being accessed
   //           Accessible from: Only within the same class. eg:private String name = "Nina";
   //                  Use case: For data hiding or encapsulation—restricts access to internal fields/methods.
   //protected -> default + parent/child access
   //             allow the parent child classes in different scope to access the fields and APIs
   //             Accessible from: The same class, Subclasses (even in different packages),
   //                              Other classes in the same package.
   //                    Use case: Useful in inheritance scenarios.
   //                eg:protected void show() {
   //                       System.out.println("Protected method");
   //default -> based on the package to isolate encapsulate all the elements
   //           Accessible from: Only within the same package. eg:int age = 25; // No modifier = default access
   //                  Use case: When you want to allow access to classes in the same package but hide it from outside.

   // class
   // object(instance) vs template(class)
   // object: new -> instantiate the object
   // template: singleton -> abstract?
   //           interface, class, abstract class, enum, annotation(@interface)
   // abstract class: contain abstract methods; cannot be instantiated directly; no bodies of implementation.
   //                 extend abstract class -> override

   // interface & abstract class -> inheritance mechanisms:
   //                               extend from an abstract class/ implement from the interface
   //   abstract class - extends
   //   abstract class Animal {
   //    abstract void makeSound();
   //
   //    void sleep() {
   //        System.out.println("Zzz...");
   //    }
   //}
   //
   //    class Dog extends Animal {
   //    @Override
   //    void makeSound() {
   //        System.out.println("Woof!");
   //    }
   //}  only extend one abstract class

   // interface - implements
   // interface Flyable {
   //    void fly();
   //}
   //
   //   class Bird implements Flyable {
   //    @Override
   //    public void fly() {
   //        System.out.println("Flying high!");
   //    }
   //} multiple inheritance via interfaces
   // class MyClass implement interface1,2,3 extend one class

   //*static: field, method, class, block (access elts from template, ow instantiate the object -> method area
   //loaded beofor java program starts to run
   //java program loaded/compiled -> converts the .java to .class -> loaded into JVM
   // -> static elts loaded into method area -> static block executed -> java program runs

   //static Field (a.k.a. static variable)
   //             Shared by all instances of the class. Stored in method area (not heap).
   //             Exists once per class, not per object.

   //JVM(java VM) memory map: stack, heap, PC, *method area, native method stack.
   //VM: Java??? contain those five memory areas? EC2/EKS

   //objects -> heap (static -> method area)
   //static elts (template, source code) -> method area

   //new keyword to instantiate object: check if template class(method area)
   //                                   yes? -> instantiate object & store object into the heap
   //          eg:     String s = new String("a");
   //                   s-  !!reference!! pointing to this object(new String("a"))
   //each of calling of a method has created a stack frame
   //called method(finish execution) -> PC check where to return back to

   //native method stack: backdoor to store all the c and c++ APIs

   //static block:  read your configuration/boot up web application(bootstrap)
   // static {
   // System.out.println("hello");}    loaded/executed once

   //static class?
   //inside a public class; outer class cannot be static
   private static Singleton instance;
   //-> eager: private static Singleton instance = new Singleton();
   private Singleton() {} //constructor: private? access by this class only
   //                            cannot create singleton object outside of the singleton class
   //                       class template -> singleton design panel(limited access of instance)
   //                         access it -> getter

       public static Singleton getInstance() {
       //static? have an API, not create object outside of singleton class template
           //external user wants to use it -> static handler -> call getter to get singleton object
       if (instance == null) {
           instance = new Singleton();
       }
       return instance;
   }

   public static void main(String[] args) {
       Singleton a = Singleton.getInstance();
       Singleton b = Singleton.getInstance();
       System.out.println("Same object? " + (a == b));
   }
}

