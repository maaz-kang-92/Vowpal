include(CSharpUtilities)
include(NetFxStrongName)
include(NetFxResource)

# TODO: Make AssemblyInfo a configure()ed file
add_library(VowpalWabbit.Common SHARED
  Properties/AssemblyInfo.cs
  Reflection/InspectionHelper.cs
  Reflection/ReflectionHelper.cs
  Reflection/TypeDistance.cs
  Reflection/TypeMatch.cs
  Serializer/FeatureExpression.cs
  Serializer/LabelExpression.cs
  Serializer/PropertyConfiguration.cs
  Serializer/Schema.cs
  Serializer/StringProcessing.cs
  Bag.cs
  VowpalWabbitDecisionServiceInteractionHeader.cs
)

strongname_sign_target(VowpalWabbit.Common ${vw_DOTNET_SIGNING_KEY})

# The VowpalWabbit .NET bindings generate a dynamic assembly for the serialization logic to enable
# efficient type-safe bindings. Unfortunately, in order to reference internal members, this dynamic
# assembly needs to be StrongName signed, since the assemblies in the NuGet are StrongName signed.
# To that end, we need to embed the keyfile into the NuGet, so that we can sign the generated assembly
# on the fly.
add_target_resources(VowpalWabbit.Common ${vw_DOTNET_SIGNING_KEY})

CSHARP_SET_DESIGNER_CS_PROPERTIES(
    "Properties/AssemblyInfo.cs"
)

set (vw_DOTNET_FRAMEWORK_REFERENCES
    System
    System.Core
    System.Data
    System.Data.DataSetExtensions
    System.Xml
    System.Xml.Linq
    Microsoft.CSharp
)

target_compile_options(VowpalWabbit.Common PRIVATE
  $<$<CONFIG:Release>:/debug:pdbonly>
  $<$<CONFIG:Debug>:/debug:full>
)

set_target_properties(VowpalWabbit.Common PROPERTIES
    VS_DOTNET_REFERENCES "${vw_DOTNET_FRAMEWORK_REFERENCES}"
    VS_GLOBAL_ROOTNAMESPACE VW
    VS_GLOBAL_DocumentationFile "VowpalWabbit.Common.xml"
)

install(TARGETS VowpalWabbit.Common
  RUNTIME DESTINATION "${CMAKE_INSTALL_LIBDIR}/${VW_NET_FRAMEWORK_TFM}"
)
