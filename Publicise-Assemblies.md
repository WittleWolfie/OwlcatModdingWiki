Publicizing assemblies can allow you to access private types and members by building your mod against libraries that have been modified to make their access modifiers public. This technique allows mods direct access private types and members without the use of reflection, providing increased performance and type safety. Mods need to be marked unsafe to prevent runtime access violations as the game is expected to be running the original assembly with non-public access modifiers.

* Install the [NuGet Package](https://www.nuget.org/packages/Aze.Publicise.MSBuild.Task/1.0.0).
* Open the project's .csproj file and add
```xml
  <PropertyGroup>
    <InstallDir Condition=" '$(InstallDir)' == '' ">C:\Program Files (x86)\Steam\steamapps\common\Pathfinder Kingmaker</InstallDir>
  </PropertyGroup>
  <Target Name="Publicise" AfterTargets="Clean">
    <ItemGroup>
      <PubliciseInputAssemblies Include="$(InstallDir)\Kingmaker_Data\Managed\Assembly-CSharp.dll" />
    </ItemGroup>
    <Publicise InputAssemblies="@(PubliciseInputAssemblies)" OutputPath="$(SolutionDir)lib/" PubliciseCompilerGenerated="true" />
  </Target>
```
* Ensure Unsafe Code is enabled. If it is not enabled, runtime access violations will be encountered in certain situations.
* Reference the publicized Assembly-CSharp
```xml
  <Reference Include="Assembly-CSharp">
      <HintPath>$(SolutionDir)lib\Assembly-CSharp_public.dll</HintPath>
      <Private>False</Private>
  </Reference>
```

* Important Note: It is intended that you reference only the publicized assembly, or the protected one. Referencing both at the same time will cause interesting complaints in your IDE.