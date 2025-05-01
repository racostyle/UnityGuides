# Guide: Integrating NUnit and Moq into Unity

This guide explains how to set up **unit testing** in Unity using **NUnit** and **Moq**. Unity has built-in support for NUnit, but Moq needs to be added manually.

---

## Setting up NUnit

1. Open the Test Runner:  
   Go to `Window → General → Test Runner`.

2. Create a folder for tests:  
   Inside `Assets`, create a new folder named `UnitTests`.

3. Create Edit Mode Tests:  
   Select `UnitTests` folder and in the Test Runner window, click **Create EditMode Test Assembly Folder**.  
   This will create a folder with an `Assembly Definition` file used for settings and references. Name it something like `EditTests` or `EditMode`.

![2](https://github.com/user-attachments/assets/f4e5805d-8e03-4785-888e-ac330fc429b3)


4. Create an Assembly Definition for your code:  
   Navigate to the folder containing the code you want to test. Right-click and 
   create a new Assembly Definition file. In my case, it is `LibUR_Assembly`.asmdef.

![3](https://github.com/user-attachments/assets/6f50924a-4270-4b94-97e1-21ec69f00b56)


### Why Assembly Definitions Are Needed in Unity

By default, Unity compiles all scripts into a single "compilation unit", meaning every script is part of Unity's default assembly. However, if you want to separate your test code from your production code and improve modularity and compilation times, you need to use **Assembly Definition (.asmdef)** files.

These files tell Unity to treat all scripts in a folder as a distinct module. This separation is essential for structured unit testing.

> If you have plain C# classes shared between your test and production code and they belong to the same or a shared `.asmdef`, **you don't need to add extra references**—as long as the dependency is one-way (e.g. tests use the code, not the other way around).

In more complex setups, where your production code and test code are in separate `.asmdef` files (e.g. `MyGame.asmdef` for production and `MyGame.Tests.asmdef` for tests), your test `.asmdef` must reference the production one in its Inspector. This allows your tests to "see" the code they are testing.

5. Link the production assembly in the test assembly:  
   Select the `EditTests.asmdef` file and, in the Inspector, add a reference to the `myCode_Assembly` definition created in step 4. Don't forget to click apply after!

![3](https://github.com/user-attachments/assets/ce080ea3-2243-4451-85a6-6aa47b872b3d)

---

## Adding Moq to Unity

6. Create a folder named `libs` in `Assets`.  
   This will store the external Moq and Castle.Core libraries.

7. Download the correct versions of Moq and Castle.Core:  
   Unity uses an older .NET version, so you need:
   - **Moq 4.13.1**
   - **Castle.Core 4.4.0**

   Download the `.nupkg` packages from NuGet and extract them. If your OS doesn't recognize the `.nupkg` file format, rename it to `.zip`.

   From the extracted content:
   - From **Moq**, copy everything inside the `lib/netstandard2.0` folder.
   - From **Castle.Core**, copy everything inside the `lib/netstandard1.5` folder.

   Place all `.dll` files into the `libs` folder you created in step 6.

8. For each DLL file:
   - Select it in the Unity Project window.
   - In the Inspector, uncheck **Any Platform**.
   - Check **Editor** only.
   - Click **Apply**.

9. Create an Assembly Definition for the libraries:  
   In the `libs` folder, create an `.asmdef` file (e.g. `libsAssembly`) with the following content:

   ```json
   {
     "name": "libsAssembly",
     "precompiledReferences": [
       "Moq.dll",
       "Castle.Core.dll"
     ],
     "includePlatforms": [ "Editor" ]
   }
   ```

10. Add a reference to the Moq libraries in your test assembly:  
    Open the `EditTests.asmdef` file and add a reference to `libsAssembly`.

11. Update the `EditTests.asmdef` file:  
    Make sure the `overrideReferences` field is set to `false` (this is important to allow transitive references from other assemblies like `libsAssembly` to be included).

   Example content of `EditTests.asmdef`:

   ```json
   {
     "name": "EditTests",
     "rootNamespace": "",
     "references": [
       "UnityEngine.TestRunner",
       "UnityEditor.TestRunner",
       "myCode_Assembly",
       "libsAssembly"
     ],
     "includePlatforms": [
       "Editor"
     ],
     "excludePlatforms": [],
     "allowUnsafeCode": false,
     "overrideReferences": false,
     "precompiledReferences": [
       "nunit.framework.dll"
     ],
     "autoReferenced": false,
     "defineConstraints": [
       "UNITY_INCLUDE_TESTS"
     ],
     "versionDefines": [],
     "noEngineReferences": false
   }
   ```

> ⚠️ If `overrideReferences` is set to `true`, Unity will only include the DLLs listed under `precompiledReferences` and will **ignore transitive references** from other `.asmdef` files (like `libsAssembly`). That’s why it must be set to `false`.

---

You are now ready to write unit tests in Unity using **NUnit** and **Moq**!

```csharp
[TestFixture]
public class ExampleTest
{
    [Test]
    public void SampleTest()
    {
        var mockedService = new Mock<IMyService>();
        mockedService.Setup(x => x.GetValue()).Returns(42);

        var result = mockedService.Object.GetValue();
        Assert.AreEqual(42, result);
    }
}
```
