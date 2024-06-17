Installing OpenDDS and running the examples.

This is detailed [here](https://opendds.readthedocs.io/en/latest-release/devguide/quickstart/windows.html), but this Obsidian note goes into some more detail into the steps since there were issues.

1. Install [Perl](https://strawberryperl.com/), which is required for Open DDS to run the examples (the examples are just unit tests that run using Perl scripts).
2. Ensure that Visual Studio is installed.
3. Ensure that JAVA jdk and JNI are installed and the environment variables are set up.
4. ~~Download the latest~~ [~~release~~](https://github.com/OpenDDS/OpenDDS/releases) . This actually fails to install the ACE and TAO necessary components. Its better to actually use [git to get the code (git clone)](https://github.com/OpenDDS/OpenDDS).
5. Open a Visual Studio Command prompt (use x64):![[VSCommandPrompt.png]]
6. Navigate to the Open DDS directory:![[Command1.png]]
7. run the batch file 'configure':![[Configure.png]]
8. Enable JAVA support:![[configurejava.png]]
9. The terminal should report a visual studio solution path, which should be called from the terminal to open in visual studio:![[VSOpen.png]]
10. Set the build to x64 in VS:![[VSBuildTarget.png]]
11. Build solution:![[buildSuccess.png]]
12. The environment variables must be set every time a new terminal is opened in order to run the examples:![[setEnvVar.png]]
13. Navigate to the cpp example and run. This is essentially a unit test run in perl, which creates a publisher and a subscriber, and checks the communication between them:![[runCPP.png]]
14. Test should pass:![[cppPass.png]]
16. To run the java example, we must [first fix the unit test](https://stackoverflow.com/questions/65306292/dds-java-example-throws-tons-of-warnings-jni-call-made-without-checking-excepti) by [removing a line](https://github.com/OpenDDS/OpenDDS/blob/9ac3dc24b403830e68f554a51ab0682857fb9b51/bin/PerlDDS/Process_Java.pm#L21):![[fixUnitTest.png]] 
17. Navigate to the Java example and run:![[runJava.png]]
18. Test should pass:![[javaPass.png]]

The Java bindings rely on [JNI](obsidian://open?vault=Obsidian&file=DDS%2FJNI%20JAVA%20Native%20Interface) calls to underlying cpp code.