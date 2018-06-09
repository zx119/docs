---
title: "XML documentation comments (C# Programming Guide)"
description: "Learn how to create API documentation for your C# code directly in the source code."
ms.date: 06/08/2018
f1_keywords: 
  - "cs.xml"
helpviewer_keywords: 
  - "XML [C#], code comments"
  - "comments [C#], XML"
  - "documentation comments [C#]"
  - "C# source code files"
  - "C# language, XML code comments"
  - "XML documentation comments [C#]"
ms.assetid: 803b7f7b-7428-4725-b5db-9a6cff273199
---
# XML documentation comments (C# Programming Guide)

In Visual C#, you can create documentation for your code by including XML elements in special comment fields (indicated by triple slashes) in the source code directly before the code block to which the comments refer, for example:  

```csharp
/// <summary>
///  This class performs an important function.
/// </summary>
public class MyClass {
    ...
}
```

When you compile with the [/doc](../../../csharp/language-reference/compiler-options/doc-compiler-option.md) option, the compiler searches for all XML tags in the source code and create an XML documentation file. To create the final documentation based on the compiler-generated file, you can create a custom tool or use a tool such as [DocFx](https://dotnet.github.io/docfx/) or [Sandcastle](https://github.com/EWSoftware/SHFB).

To refer to XML elements (for example, your function processes specific XML elements that you want to describe in an XML documentation comment), you can use the standard quoting mechanism (`<` and `>`). To refer to generic identifiers in code reference (`cref`) elements, you can use either the escape characters (for example, `cref="List&lt;T&gt;"`) or braces (`cref="List{T}"`). As a special case, the compiler parses the braces as angle brackets to make the documentation comment less cumbersome to author when referring to generic identifiers.  
  
> [!NOTE]
> The XML documentation comments aren't metadata; they aren't included in the compiled assembly and therefore they aren't accessible through reflection.  
  
## In this section

- [Recommended Tags for Documentation Comments](recommended-tags-for-documentation-comments.md)

- [Processing the XML File](processing-the-xml-file.md)

- [Delimiters for Documentation Tags](delimiters-for-documentation-tags.md)

- [How to: Use the XML Documentation Features](how-to-use-the-xml-documentation-features.md)
  
## Related sections

For more information, see [/doc (Process Documentation Comments)](../../../csharp/language-reference/compiler-options/doc-compiler-option.md).

## C# language specification

[!INCLUDE[CSharplangspec](~/includes/csharplangspec-md.md)]  
  
## See also

[C# Programming Guide](../../../csharp/programming-guide/index.md)
