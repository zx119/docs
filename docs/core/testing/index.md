---
title: "A log has already been created with this name on this machine"
ms.date: "2015-07-20"
ms.prod: .net
ms.technology: 
  - "devlang-visual-basic"
ms.topic: "article"
ms.assetid: 3dd78d9f-890e-4409-bebb-048fdf34711b
caps.latest.revision: 10
author: dotnet-bot
ms.author: dotnetcontent
translation.priority.ht: 
  - "de-de"
  - "es-es"
  - "fr-fr"
  - "it-it"
  - "ja-jp"
  - "ko-kr"
  - "ru-ru"
  - "zh-cn"
  - "zh-tw"
translation.priority.mt: 
  - "cs-cz"
  - "pl-pl"
  - "pt-br"
  - "tr-tr"
---
# Dependency injection with Visual Basic .NET - Part 2 – IoC containers

In my previous post I wrote about the basics of dependency injection. I explained the technique to define an interface and injecting the dependencies to a client object. These dependencies contain the real implementation of that specific interface. Applying dependency injection makes your code more loosely coupled, which helps you in maintaining, extending, and testing your codebase. The example ended up with works fine, but still can be improved for some scenarios.

## Back to the basics

To explain what we can we improve, we should take a closer look at the code from the previous post. Starting with the context of the code example: When you were a child, and you were thirsty, you asked your mother for something to drink. You were happy when she handed you ‘something’. But what was that ‘something’ a cup of milk, a glass of orange juice, or something different? Your mother was in charge: you asked for a drink, but you didn’t know in advance, what you eventually get. Below you see an excerpt from the full example in Visual Basic .NET source code:

```vb
Public Interface IDrinkService

    Sub Drink()

End Interface

Public Class MilkDrinkService

    Implements IDrinkService

    Public Sub Drink() Implements IDrinkService.Drink
        Console.WriteLine("Drink a cup of milk")
    End Sub

End Class

Public Class Mother

    Public Function GetDrinkService() As IDrinkService
        Return New MilkDrinkService()
    End Function

End Class

Sub Main()

    Dim injector As New Mother
    Dim serviceObject = injector.GetDrinkService()
    Dim clientObject As New Child(serviceObject)

    clientObject.Drink()

    Console.ReadLine()

End Sub
```

In this example, the actual decision of what kind of IDrinkService is returned by the GetDrinkService() method on the Mother class is hardcoded. This is not ideal. If you want to change the behavior of the injector, you must change the code in that class. In more complex scenarios this can be quite hard, because you must figure out where the code lives, and especially if you must change more implementations, this can be very time consuming. It is not a best practice to have all these ‘decision code snippets’ all around your code. It’s much more convenient to have this composition code centrally located in one method of your application. Beside of that, you must recompile and deploy your application again. In some cases, this is not a big issue and obvious, but you can think of other examples where it would be very handy when you can do it by simply changing one or more settings in an external configuration file. For all these scenarios, you can consider using an IoC container.

## Inversion of control (IoC)

IoC stands for ‘inversion of control’. An IoC container manages the flow and logic of an application, basically in one method in your code. Mostly it is initialized during startup of your application and, depending on the defined scope, it is responsible for returning all the actual implementations for the configured interfaces. This is quite a different approach from what we are used to as more procedural programmers. You can see it as a generic framework for serving types, based on configuration. The definition of IoC on Wikipedia states more or less the same: “In software engineering, inversion of control (IoC) is a design principle in which custom-written portions of a computer program receive the flow of control from a generic framework. A software architecture with this design inverts control as compared to traditional procedural programming: in traditional programming, the custom code that expresses the purpose of the program calls into reusable libraries to take care of generic tasks, but with inversion of control, it is the framework that calls into the custom, or task-specific, code.”
There are several IoC frameworks in the market today, most of them in the open source space. If you do some searching on the internet you will find out that there are even quite many of them. I came easily to a number of around 40! The most well-known IoC frameworks for .NET are Unity, Ninject, SimpleInjector and Autofac. But which one should you choose? That depends…. Is the framework still under active development? Most of them have their repositories on GitHub, so it’s easy to verify when the last code was checked in and when the latest version was released. Another rationale could be performance of the various frameworks. It turns out that some are substantially faster than others. Or, and I think this applies in many scenarios, how good is the integration with other components (for example, with the MVVM framework you want to use in your application)?

In this post, I will use Autofac. But please remember, fundamentally, all frameworks use the same concept. They differ only in syntax and the options to configure them. 
Concept of an IoC container
As said, most IoC frameworks are based on the same concept. But what is that concept? During initialization of your application you register the types to use for the various interfaces. You ‘tell’ to the IoC container what type it should return. If the application ‘asks’ for an IDrinkService, it must deliver for example the MilkDrinkService. And this is, in brief and very basically, what an IoC container does. And of course, there are several configurations possible, like for example the scope of the object. Should it be instantiated as new object every time it’s needed? Or is the singleton pattern more suitable? Or is configuration hardcoded only or can you do it also with help of an external configuration file? But at the end, the main concept is the same.
Let’s see how this works out in code. In my daily life, the different implementations of an interface are mostly implemented as separate assemblies. We will take the same example as in the previous post, but we split them up in more projects.
•	Create a new Console App and name it DependencyInjectionVBPart02. Rename Module1.vb to MainModule.vb. Go to the Properties Window of this project and set the Startup object to Sub Main.
•	Add a new Class Library project and name it DependencyInjectionVBPart02.Interfaces. Delete Class1.vb and add an Interface with name IDrinkService.vb.
•	Add a new Class Library project and name it DependencyInjectionVBPart02.Milk. Rename Class1.vb to DrinkService.vb.
•	Add a new Class Library project and name it DependencyInjectionVBPart02.OrangeJuice. Rename, just as in the previous step, Class1.vb to DrinkService.vb.
All our projects are created. Now we must set the references between them. We have created a separate project which contains the interface, in our case the IDrinkService. The reason why this is extracted to a separate assembly, is that all the other projects must have knowledge of this generic part of our software. The Console App, because that code will consume the interface, and the other two libraries, because they will implement that interface. For now, we will also set references from the Console App to the two actual implementation projects. In the last paragraph of this post, you can remove those references, to show you a very flexible scenario. But for now:
•	Add in DependencyInjectionVBPart02, DependencyInjectionVBPart02.Milk and DependencyInjectionVBPart02.OrangeJuice a project reference to DependencyInjectionVBPart02.Interfaces.
•	Add in DependencyInjectionVBPart02 a project reference to DependencyInjectionVBPart02.Milk and DependencyInjectionVBPart02.OrangeJuice.
After we have set these references we will define the IDrinkService and implement them in the other two class libraries. Change the code of IDrinkService.vb in DependencyInjectionVBPart02.Interfaces to:

```vb
Public Interface IDrinkService

    Sub Drink()

End Interface
```

In DependencyInjectionVBPart02.Milk you implement the specific business logic to drink milk.

```vb
Imports DependencyInjectionVBPart02.Interfaces

Public Class DrinkService

    Implements IDrinkService

    Public Sub Drink() Implements IDrinkService.Drink
        Console.WriteLine("Drink a cup of milk")
    End Sub

End Class
```

And do the same for the orange juice implementation in DependencyInjectionVBPart02.OrangeJuice.

```vb
Imports DependencyInjectionVBPart02.Interfaces

Public Class DrinkService

    Implements IDrinkService

    Public Sub Drink() Implements IDrinkService.Drink
        Console.WriteLine("Drink a glass of orange juice")
    End Sub

End Class
```

Now it’s time to create the object that will consume the interface, in terms of the previous post: the client object. Add a new class to the Console App and name it Child.vb. Add the following code:

```vb
Imports DependencyInjectionVBPart02.Interfaces

Public Class Child

    Private ReadOnly _drinkService As IDrinkService

    Public Sub New(drinkService As IDrinkService)
        _drinkService = drinkService
    End Sub

    Public Sub Drink()
        _drinkService.Drink()
    End Sub

End Class
```

Let us reflect a bit on what we have created till now. We have a class named Child, that makes use of the IDrinkService interface. Depending on the specific scenario it will get from the injector, which would be in our example an Autofac IoC container, an instance of the Milk.DrinkService or OrangeJuice.DrinkService. As said in the first post, the client object is not aware of the actual implementations. When a new Child is instantiated, it gets one of the two DrinkServices injected.
Finally, we are at the point where the IoC container comes in place. As shown at the beginning of the post, the Mother was used as injector and it was responsible for creating the instance of for example the MilkDrinkService. Creation of a new specific objects from code is what you want to prevent when using an IoC container. So, this is where the code in this post will be really going different.
Add to our Console App the Autofac NuGet Package.

```
PM> Install-Package Autofac -Version 4.6.0
```

To use an IoC container we must define the container and build or construct it using a ContainerBuilder. Add the following code to MainModule.vb:

```vb
Imports Autofac
Imports Autofac.Core
Imports DependencyInjectionVBPart02.Interfaces

Module MainModule

    Private _container As Container

    Private Sub InitializeContainer()

        Dim builder As New ContainerBuilder()

        builder.RegisterType(Of Milk.DrinkService).As(Of IDrinkService)()

        _container = builder.Build()

    End Sub

End Module
```

The InitializeContainer() method is responsible for registering or mapping of the implementation types to the corresponding interfaces. As you can see, we are ‘telling’ the container that when we are ‘asking’ for an object of the type IDrinkService that an object of the type Milk.DrinkService must be returned. You can imagine that you can register more than one type in this method, and we will do this later.
I think that you want to see this code in action, but we are not finished yet. What is missing, is the code to call al this magic. Add the following source code to Sub Main():

```vb
Sub Main()

    InitializeContainer()

    Dim drinkService = _container.Resolve(Of IDrinkService)
    Dim child As New Child(drinkService)
    child.Drink()

    Console.ReadLine()

End Sub
```

Hit F5 and… voila!

We can go one step further and demonstrate a feature of Autofac to support constructor injection automatically. Let’s introduce a new interface IChild. Add a new interface to our class library with interfaces, DependencyInjectionVBPart02.Interfaces. Name it IChild.vb and add the following code:

```vb
Public Interface IChild

    Sub Drink()

End Interface
```

Modify the current Child.vb class to implement the interface we just created.

```vb
Public Class Child

    Implements IChild

    Private ReadOnly _drinkService As IDrinkService

    Public Sub New(drinkService As IDrinkService)
        _drinkService = drinkService
    End Sub

    Public Sub Drink() Implements IChild.Drink
        _drinkService.Drink()
    End Sub

End Class
```

Remember, introducing a new interface means also that we must register this type in our InitializeContainer() method. Modify this method and add a new line under the existing one where we register the Milk.DrinkService.

```vb
Private Sub InitializeContainer()

    Dim builder As New ContainerBuilder()

    builder.RegisterType(Of Milk.DrinkService).As(Of IDrinkService)()
    builder.RegisterType(Of Child).As(Of IChild)()

    _container = builder.Build()

End Sub
```

Now, after we have registered also the IChild interface, we can ask the IoC container to resolve the Child object. Run our application and note that this is also working as expected.

```vb
Sub Main()

    InitializeContainer()

    Dim child = _container.Resolve(Of IChild)
    child.Drink()

    Console.ReadLine()

End Sub
```

Let’s summarize what is happening here:
•	The Main() method asks the container for an IChild;
•	The container sees that IChild maps to Child, so it starts creating a Child;
•	The container sees that the Child needs an IDrinkService in its constructor;
•	The container sees that IDrinkService maps to Milk.DrinkService, so it creates a new Milk.DrinkService instance;
•	The container uses the new Milk.DrinkService instance to finish constructing Child;
•	The container returns the fully-constructed Child to the Main() method to consume.
As you can see, it’s quite impressive how smart Autofac, and most of the other IoC containers, are in dealing with this kind of dependency-resolving and -injection.

Use of a configuration file
What you have seen is pretty cool, but it can still a bit cooler. The construction of our container is still hardcoded in our application. Even though that this way of configuration is encouraged by Autofac, in some scenarios you need ultimate flexibility. An example could be that you want to change the behavior of your software, without recompiling your application. For these kind of technical requirements, you can extract the configuration out of the source code and place it in an external file.
The filetypes which are by default supported, are JSON or XML configuration files. We will use JSON for now. To establish this, we must add two other NuGet packages to the Console App: Autofac.Configuration and Microsoft.Extensions.Configuration.Json. Use the NuGet Package Manager Window in Visual Studio as earlier shown in a screenshot or enter the following commands in the Package Manager Console:

```powershell
PM> Install-Package Autofac.Configuration -Version 4.0.1
PM> Install-Package Microsoft.Extensions.Configuration.Json -Version 1.1.2
```

With installing these two NuGet packages, a lot of other packages are also pulled in. If you need to support XML files above JSON, you can install Microsoft.Extensions.Configuration.Xml instead of the Json-variant. The basic steps to getting configuration set up with an external file and your application are:
•	Set up your configuration in JSON files that can be read by Microsoft.Extensions.Configuration. JSON configuration uses Microsoft.Extensions.Configuration.Json implementation.
•	Build the configuration using the Microsoft.Extensions.Configuration.ConfigurationBuilder.
•	Create a new Autofac.Configuration.ConfigurationModule and pass the built Microsoft.Extensions.Configuration.IConfiguration into it.
•	Register the Autofac.Configuration.ConfigurationModule with your container.
This theory translated to Visual Basic .NET source code means that we have to rewrite our InitializeContainer() method.

```vb
Private Sub InitializeContainer()

    Dim configurationBuilder = New ConfigurationBuilder()
    configurationBuilder.SetBasePath(Environment.CurrentDirectory)
    configurationBuilder.AddJsonFile("Autofac.json")

    Dim configurationModule As New ConfigurationModule(configurationBuilder.Build())

    Dim containerBuilder = New ContainerBuilder()
    containerBuilder.RegisterModule(configurationModule)

    _container = containerBuilder.Build()

End Sub
```

If you have read the basic steps very thoroughly, you know that we skipped the first step. We still need to create the configuration file self. Add to the Console App an additional item of type JSON file. Give it the name Autofac.json. Go to the properties of this file in the Solution Explorer and set the value of ‘Copy to Output Directory’ to the value of ‘Copy if newer’. This will ensure that our executable can find the JSON file in its own directory when its executed. Copy and paste the next JSON into this file.

```json
{
  "defaultAssembly": "DependencyInjectionVBPart02",
  "components": [
    {
      "type": "DependencyInjectionVBPart02.Child, DependencyInjectionVBPart02",
      "services": [
        {
          "type": "DependencyInjectionVBPart02.Interfaces.IChild, DependencyInjectionVBPart02.Interfaces"
        }
      ],
      "injectProperties": true
    },
    {
      "type": "DependencyInjectionVBPart02.OrangeJuice.DrinkService, DependencyInjectionVBPart02.OrangeJuice",
      "services": [
        {
          "type": "DependencyInjectionVBPart02.Interfaces.IDrinkService, DependencyInjectionVBPart02.Interfaces"
        }
      ]
    }
  ]
}
```

The format of this file – and all possible options - is in detail described on the website of Autofac. For now, I will not dig into these details. But what you can see in this file, is in the end the same as what we wrote in code. We register the type and map it to its corresponding interface. The first parameter of the ‘type’ attribute is the name of the class, including the full namespace. After that and separated by a comma, the second parameter. This is the name of the assembly where this type or interface lives (without the extension .EXE or .DLL).
When you run our application, you will see that this still works as expected!

In theory, you can now delete the references set in the Console App to DependencyInjectionVBPart02.Milk and DependencyInjectionVBPart02.OrangeJuice. Your application doesn’t have to know about these implementations. It’s all done by configuration in the external JSON file. But, if you do so, please be aware that if the references are missing, these class libraries will not be built automatically when you start the application. When these assemblies already exist in the output directory and nothing has changed, there is no problem. But when you changed something in a class library and when you see that still the old logic is used, don’t forget to choose Rebuild Solution. You will not the first person who encounters this ‘problem’ and you will be definitely not the last!

Happy coding!
André Obelink, @obelink
www.obelink.com

Links:
•	Dependency injection with Visual Basic .NET – Part 1
•	autofac.org
•	https://github.com/obelink/DependencyInjectionVBPart02