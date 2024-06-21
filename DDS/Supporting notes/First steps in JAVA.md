
The steps are documented [here](https://opendds.readthedocs.io/en/latest-release/devguide/java_bindings.html#java). The openDDS installation should be followed as documented [here](obsidian://open?vault=Obsidian&file=DDS%2FAttempting%20Open%20DDS%20on%20Windows). This is also documented poorly [here](https://github.com/OpenDDS/OpenDDS/blob/DDS-3.28.1/java/INSTALL), but also recommends to run a different example/unit test.

There is another example/unit test to run:
![[setEnvVar.png]]
![[JavaUnitTestTwo.png]]
![[TestTwoPass.png]]

Note that it is necessary to set the environment variables every time a new VSx64 command prompt is opened.

**First Project setup**

1. Create an empty directory to store the idl file and all the code that is generated from it:![[EmptyDirectory.png]]



2. Define an [IDL file](obsidian://open?vault=Obsidian&file=DDS%2FSupporting%20notes%2FIDL%20Files). Here is my file (named Kuka.idl):
```
module Kuka {

  @topic
  struct Joints {
    string from;
    double JointPos[7];
    @key long subject_id;
  };
};
```



3. The C++ generated classes will be packaged in a shared library, which is loaded at runtime by the JVM. The packaged classes must be exported for external visibility - ACE provides a utility script for generating the correct export macros. This script is called (*NOTE: This script will place the export file into the directory in which the command prompt is scoped to, so ensure to cd to the correct directory before calling script*) :

```
%ACE_ROOT%\bin\generate_export_file.pl Kuka > Kuka_Export.h
```

The export file has been created: ![[ExportFileGeneration.png]]


4. Create an [MPC file](obsidian://open?vault=Obsidian&file=DDS%2FSupporting%20notes%2FMPC%20files%20in%20DDS) Kuka.mpc , following the following template:
```
project: dcps_java {
  idlflags += -Wb,stub_export_include=Kuka_Export.h \
    -Wb,stub_export_macro=Kuka_Export
  dcps_ts_flags += -Wb,export_macro=Kuka_Export
  idl2jniflags += -Wb,stub_export_include=Kuka_Export.h \
    -Wb,stub_export_macro=Kuka_Export
  dynamicflags += KUKA_BUILD_DLL

  specific {
    jarname = DDS_Kuka_types
  }

 TypeSupport_Files {
    Kuka.idl
  }
}
```

5. Run MPC to generate platform-specific build files:
```
%ACE_ROOT%\bin\mwc.pl -type [CompilerType]
```

In my case, I am using Visual Studio 2019 to build my files:

```
%ACE_ROOT%\bin\mwc.pl -type vs2019
```

![[MPCBuild.png]]

![[MPCBuildSuccess.png]]

6. Compile the generated C++ and Java code:

This may be done directly from Visual Studio (which must be launched from a command prompt using devenv so that it inherits the environment variables)
![[VSLaunch.png]]

The commands:

```
"C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\Common7\Tools\VsDevCmd.bat"

devenv C:\Users\Adam\Desktop\OpenDDS\java\AdamTest\AdamPub_idl\AdamPub_idl.sln
```