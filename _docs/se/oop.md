---
layout: default
---

## Object Oriented Principles

#### Encapsulation

Encapsulation is used to hide the internal state of an object. It allows properties or methods to be `private`, being only visible to the class itself. In order to allow outside code to access the internal state of the object, `getters` and `setters` are implemented, allowing the desired property to be `get` or `set`.

*Access Modifiers*
  * Public - This modifier allows the property or method to be publicly accessible anywhere
  * Private - This modifier hides the property or method from any code outside of the object
  * Protected - This modifier hides the property or method from any code that is not a sub-class of the class, or is not within the same package as the class

*Encapsulation example*

```
    package coffee;

    public class Coffee {
        private String blend = "French";
        
        public String getBlend() {
            return blend;
        }
        
        public void setBlend(String blend) {
            this.blend = blend;
        }
    }
```

This class has one private property, `blend`, which is only accessible through the `getBlend()` and `setBlend()` methods. 

#### Inheritance

Inheritance can be used to create a sub-class of a class, inheriting the functionalities of the parent class. It can be helpful when requirements call for many similar objects, but each object requires to have some specific functionality for itself.

For example, two objects are needed that share some of the same functionality, but each one also requires some unique functionality specific to itself. In this case, a base class can be created that contains the logic for the similar functionality. Then, two other classes can be created which inherit the functionalities of the base class, while containing any specific logic it needs for itself.

Beverage as the base class, with a common functionality method `drink()`

```
    public class Beverage {
        
        public void drink() {
            System.out.println("tasty");
        }
    }
```

Two subclasses that extend Beverage, each with their own specific functionalities

```
    public class Coffee extends Beverage {
        private String blend = "Breakfast";
        
        public void changeBlend(String blend) {
            this.blend = blend;
        }
    }
```

```
    public class Milk extends Beverage {
        private boolean isChocolate = false;
        
        public void makeChocolate() {
            isChocolate = true;
        }
    }
```

#### Abstraction

Abstraction is a concept that allows code to be templated and reused. For example, there might be some components that have similar functionality. Instead of creating a class for each component and re-writing the same common functionality across all of them, abstraction allows to create one class containig the common functionality, and then create sub-classes which extend the abstract class. Each subclass will inherit the common functionality of the parent abstract class.

Example, we have an ElectricGuitar and an AcousticGuitar. Both have common functions like tuning and playing. However, Electric guitars have overdrive and other effects. So instead of having to re-write the same tuning and playing functionality, an abstract class can be created which contains this common functionality, and then Acoustic and Electric sub classes can extend the main abstract class.

```
    package guitars;

    public abstract class Guitar {
        public void tune() {
            System.out.println("Tuning guitar");
        }
        
        public void playSong() {
            System.out.println("Play a song");
        }
    }
```

```
    package guitars;

    public class ElectricGuitar extends Guitar {

        private boolean overdrive = false;
        
        public void enableOverdrive() {
            overdrive = true;
        }
        
        public void shred() {
            enableOverdrive();
            System.out.println("Melt faces");
        }
    }
```

```
    package guitars;

    public class AcousticGuitar extends Guitar {

        public void fingerStyle() {
            System.out.println("Play some beautiful fingerstyle");
        }
    }
```

An abstract class can also declare an abstract method, which requires an extending class to implement that method. Within the Guitar abstract class, playSong method could be declared abstract, since different guitars play differently.

```
    package guitars;

    public abstract class Guitar {
        public void tune() {
            System.out.println("Tuning guitar");
        }
        
        public abstract void playSong();
    }
```

```
    package guitars;

    public class ElectricGuitar extends Guitar {

        private boolean overdrive = false;
        
        public void enableOverdrive() {
            overdrive = true;
        }
        
        public void shred() {
            System.out.println("Melt faces");
        }
        
        @Override
        public void playSong() {
            if(overdrive) {
                shred();
            } else {
                System.out.println("Play some chillhop");
            }
        }
    }
```

```
    package guitars;

    public class AcousticGuitar extends Guitar {

        public void fingerStyle() {
            System.out.println("Play some beautiful fingerstyle");
        }
        
        @Override
        public void playSong() {
            System.out.println("Play some Johnny Cash");
        }
    }
```

Also note that abstract classes cannot be instantiated. The following code will not work:

```
import guitars.*;

public class JavaApplication {
    public static void main(String[] args) {
        Guitar guitar = new Guitar();  // Error: Guitar is abstract; cannot be instantiated
    }
}
```


#### Interfaces

Interfaces are similar to abstract classes, where they utilize the concept of abstraction, but they do not have functionality of themselves. Methods in an Interface are by default abstract; they are only method signatures. Interfaces are essentially blueprints for concrete classes when implemented; any method signature must be created in the sub class.

Interfaces are used to achieve abstraction, support multiple inheritance and to achieve loose coupling.

Classes can also implement multiple interfaces

