
Java Native Interface (JNI) is a framework that allows Java code running in the Java Virtual Machine (JVM) to call and be called by native applications (programs specific to a hardware and operating system platform) and libraries written in other languages such as C, C++, and assembly. JNI is used for several purposes:

1. **Access to Platform-Specific Features**:

- Sometimes, Java programs need to access platform-specific features or low-level system resources that are not available through the standard Java API. JNI allows Java programs to leverage these platform-specific features by interfacing with native code.

2. **Integration with Existing Libraries**:

- Many existing libraries, especially those written in C or C++, offer functionality that is not available in Java. By using JNI, Java programs can utilize these libraries without the need to rewrite them in Java.


### Key Components of JNI

1. **Java Code**:
- Java classes declare native methods, which are implemented in native code.

2. **Native Code**:
- Functions in C/C++ (or other native languages) that implement the native methods declared in Java.

3. **JNI Header Files**:
- These header files define the JNI interface and are included in the native code to enable interaction with the JVM.

### Example Use Case

DDS is implemented in c++. JNI allows java to be written which enables communication across DSS.