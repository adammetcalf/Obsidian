
Working again from [First Steps in Java](obsidian://open?vault=Obsidian&file=DDS%2FSupporting%20notes%2FFirst%20steps%20in%20JAVA).

1. Define the idl file Hello.idl
```
module Hello {
  @topic
  struct Hi {
    string hiWorld;
  };
};
```

2. Generate the supporting header file Hello_Export.h.
```
%ACE_ROOT%\bin\generate_export_file.pl Hello> Hello_Export.h
```


3. Create the mpc file Hello.mpc:
```
project: dcps_java {
  idlflags += -Wb,stub_export_include=Hello_Export.h \
    -Wb,stub_export_macro=Hello_Export
  dcps_ts_flags += -Wb,export_macro=Hello_Export
  idl2jniflags += -Wb,stub_export_include=Hello_Export.h \
    -Wb,stub_export_macro=Hello_Export
  dynamicflags += HELLO_BUILD_DLL

  specific {
    jarname = DDS_Hello_types
  }

 TypeSupport_Files {
    Hello.idl
  }
}
```


4. Run MPC to generate platform-specific build files:
```
%ACE_ROOT%\bin\mwc.pl -type vs2019
```

5. Open Visual Studio scoped to the generated sln:
```
devenv HelloWorld_idl.sln
```

Build succeeded? Good.

# Simple Publisher

We have created all the supporting code for our idl file definition.

Create [TestPublisher.java](obsidian://open?vault=Obsidian&file=DDS%2FSupporting%20notes%2FJava%20publisher%20file) and publisher_idl_test.mpc.

publisher_idl_test.mpc:
```
project: dcps_tcp, dcps_udp, dcps_multicast, dcps_test_java {

  after      += Hello
  javacflags += -classpath ../HelloWorld_idl/DDS_Hello_types.jar
  
}
```

Note that this is defined to compile after Hello, and we have defined the path to the jar file output from the the idl file compilation.

Same as before, we must now generate the build files:
```
%ACE_ROOT%\bin\mwc.pl -type vs2019
```

And build the files:
```
devenv publisher.sln
```

NOTE: The sln is named 'publisher', since that is the name of the directory in which it has been placed.

# Simple Subscriber